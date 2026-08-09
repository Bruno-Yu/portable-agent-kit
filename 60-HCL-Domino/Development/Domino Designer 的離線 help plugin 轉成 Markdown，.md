
- Export-DominoHelp.
``` powershell

<#
.SYNOPSIS
    將 HCL Notes / Domino Designer 的離線 help plugin 轉成 Markdown，
    並產生一份 deterministic symbol index (JSONL) 供本地 RAG 使用。

.DESCRIPTION
    Designer 的說明文件以 Eclipse plugin jar 形式安裝在本機，內容是 DITA 產生的
    HTML。本腳本直接從 jar 內串流讀取（不解壓到磁碟），轉為帶 YAML front-matter
    的 Markdown，同時解析 LotusScript / @Formula / Java 的符號名稱與 Syntax 區段，
    輸出成 symbols.jsonl。

    輸出結構：
        <OutputPath>\
            manifest.json          # 來源 jar 清單與統計
            symbols.jsonl          # 符號索引（每行一筆 JSON）
            md\<plugin>\<path>.md   # 轉換後文件

.PARAMETER NotesPath
    HCL Notes / Designer 安裝目錄。未指定時自動從登錄檔與常見路徑偵測。

.PARAMETER OutputPath
    輸出目錄。預設為腳本所在目錄下的 domino-help-kb。

.PARAMETER DocVersion
    用於重建線上文件 URL 的版本字串，例如 11.0.1。影響 front-matter 的 source_url。

.PARAMETER JarFilter
    要納入的 jar 檔名樣式。預設抓 *doc*.jar 與 *help*.jar。

.PARAMETER ListOnly
    只列出偵測到的 jar 與其中的 HTML 數量，不做任何轉換。第一次執行建議先用這個。

.PARAMETER KeepHtml
    另外保留原始 HTML（放在 html\ 子目錄），方便比對轉換品質。

.PARAMETER Force
    輸出目錄已存在時直接覆寫。

.EXAMPLE
    .\Export-DominoHelp.ps1 -ListOnly

.EXAMPLE
    .\Export-DominoHelp.ps1 -OutputPath D:\kb\domino -DocVersion 11.0.1

.NOTES
    需求：Windows PowerShell 5.1 以上，無外部相依套件。
    輸出一律 UTF-8 無 BOM。
#>

[CmdletBinding()]
param(
    [string]   $NotesPath,
    [string]   $OutputPath = (Join-Path $PSScriptRoot 'domino-help-kb'),
    [string]   $DocVersion = '11.0.1',
    [string[]] $JarFilter  = @('*doc*.jar', '*help*.jar'),
    [switch]   $ListOnly,
    [switch]   $KeepHtml,
    [switch]   $Force
)

$ErrorActionPreference = 'Stop'
Add-Type -AssemblyName System.IO.Compression.FileSystem

$Utf8NoBom = New-Object System.Text.UTF8Encoding($false)

#region ---------- Notes 安裝路徑偵測 ----------

function Resolve-NotesPath {
    param([string]$Explicit)

    if ($Explicit) {
        if (-not (Test-Path $Explicit)) { throw "指定的 NotesPath 不存在：$Explicit" }
        return (Resolve-Path $Explicit).Path
    }

    $regKeys = @(
        'HKLM:\SOFTWARE\WOW6432Node\Lotus\Notes',
        'HKLM:\SOFTWARE\Lotus\Notes',
        'HKLM:\SOFTWARE\WOW6432Node\HCL\Notes',
        'HKLM:\SOFTWARE\HCL\Notes'
    )
    foreach ($k in $regKeys) {
        if (Test-Path $k) {
            $p = (Get-ItemProperty -Path $k -ErrorAction SilentlyContinue).Path
            if ($p -and (Test-Path $p)) { return $p.TrimEnd('\') }
        }
    }

    $candidates = @(
        'C:\Program Files (x86)\HCL\Notes',
        'C:\Program Files\HCL\Notes',
        'C:\Program Files (x86)\IBM\Notes',
        'C:\Program Files\IBM\Notes',
        'C:\Program Files (x86)\IBM\Lotus\Notes',
        'C:\Notes'
    )
    foreach ($c in $candidates) {
        if (Test-Path $c) { return $c }
    }

    throw '找不到 Notes 安裝目錄，請用 -NotesPath 明確指定。'
}

function Get-HelpJar {
    param([string]$Root, [string[]]$Filter)

    $pluginDirs = @(
        (Join-Path $Root 'framework\eclipse\plugins'),
        (Join-Path $Root 'framework\shared\eclipse\plugins'),
        (Join-Path $Root 'framework\rcp\eclipse\plugins'),
        (Join-Path $Root 'osgi\shared\eclipse\plugins'),
        (Join-Path $Root 'osgi\rcp\eclipse\plugins')
    ) | Where-Object { Test-Path $_ }

    if (-not $pluginDirs) {
        throw "在 $Root 底下找不到任何 eclipse\plugins 目錄。"
    }

    $jars = foreach ($d in $pluginDirs) {
        foreach ($f in $Filter) {
            Get-ChildItem -Path $d -Filter $f -File -Recurse -ErrorAction SilentlyContinue
        }
    }
    $jars | Sort-Object FullName -Unique
}

#endregion

#region ---------- HTML -> Markdown ----------

function ConvertFrom-HtmlToMarkdown {
    param([string]$Html)

    if ([string]::IsNullOrWhiteSpace($Html)) { return '' }

    $s = $Html

    # 1. 移除不需要的整段區塊
    $s = [regex]::Replace($s, '(?is)<!DOCTYPE[^>]*>',        '')
    $s = [regex]::Replace($s, '(?is)<head\b.*?</head>',      '')
    $s = [regex]::Replace($s, '(?is)<script\b.*?</script>',  '')
    $s = [regex]::Replace($s, '(?is)<style\b.*?</style>',    '')
    $s = [regex]::Replace($s, '(?is)<!--.*?-->',             '')

    # 2. 保護 pre / code 區塊，避免後續規則破壞其內容
    $vault = @{}
    $idx   = 0
    $preEval = [System.Text.RegularExpressions.MatchEvaluator] {
        param($m)
        $inner = $m.Groups[1].Value
        $inner = [regex]::Replace($inner, '(?is)<br\s*/?>', "`n")
        $inner = [regex]::Replace($inner, '(?is)<[^>]+>', '')
        $inner = [System.Net.WebUtility]::HtmlDecode($inner)
        $inner = $inner -replace "`r`n", "`n"
        $inner = $inner.Trim("`n")
        $key   = "@@CODEBLOCK$($script:idx)@@"
        $script:vault[$key] = "``````lotusscript`n$inner`n``````"
        $script:idx++
        return "`n$key`n"
    }
    $script:vault = $vault
    $script:idx   = $idx
    $s = [regex]::Replace($s, '(?is)<pre[^>]*>(.*?)</pre>', $preEval)
    $vault = $script:vault

    # 3. 表格（在移除標籤前處理）
    $tableEval = [System.Text.RegularExpressions.MatchEvaluator] {
        param($m)
        $body = $m.Groups[1].Value
        $rows = [regex]::Matches($body, '(?is)<tr[^>]*>(.*?)</tr>')
        if ($rows.Count -eq 0) { return '' }

        $lines   = New-Object System.Collections.Generic.List[string]
        $colCount = 0
        $rowIndex = 0
        foreach ($r in $rows) {
            $cells = [regex]::Matches($r.Groups[1].Value, '(?is)<t[hd][^>]*>(.*?)</t[hd]>')
            if ($cells.Count -eq 0) { continue }
            $vals = foreach ($c in $cells) {
                $t = [regex]::Replace($c.Groups[1].Value, '(?is)<br\s*/?>', ' ')
                $t = [regex]::Replace($t, '(?is)<[^>]+>', '')
                $t = [System.Net.WebUtility]::HtmlDecode($t)
                ($t -replace '\s+', ' ').Trim() -replace '\|', '\|'
            }
            if ($colCount -eq 0) { $colCount = @($vals).Count }
            $lines.Add('| ' + (@($vals) -join ' | ') + ' |')
            if ($rowIndex -eq 0) {
                $lines.Add('|' + (' --- |' * $colCount))
            }
            $rowIndex++
        }
        return "`n" + ($lines -join "`n") + "`n"
    }
    $s = [regex]::Replace($s, '(?is)<table[^>]*>(.*?)</table>', $tableEval)

    # 4. 標題
    for ($h = 6; $h -ge 1; $h--) {
        $hashes = '#' * $h
        $s = [regex]::Replace($s, "(?is)<h$h[^>]*>(.*?)</h$h>", "`n`n$hashes `$1`n`n")
    }

    # 5. 清單
    $s = [regex]::Replace($s, '(?is)<li[^>]*>',   "`n- ")
    $s = [regex]::Replace($s, '(?is)</li>',       '')
    $s = [regex]::Replace($s, '(?is)</?[uo]l[^>]*>', "`n")
    $s = [regex]::Replace($s, '(?is)<dt[^>]*>',   "`n- **")
    $s = [regex]::Replace($s, '(?is)</dt>',       '**')
    $s = [regex]::Replace($s, '(?is)<dd[^>]*>',   "`n  ")
    $s = [regex]::Replace($s, '(?is)</?d[dl][^>]*>', '')

    # 6. 連結（保留 href，RAG 可用來做交叉引用）
    $s = [regex]::Replace($s, '(?is)<a[^>]*href\s*=\s*["'']([^"'']+)["''][^>]*>(.*?)</a>', '[$2]($1)')
    $s = [regex]::Replace($s, '(?is)<a[^>]*>(.*?)</a>', '$1')

    # 7. 行內格式
    $s = [regex]::Replace($s, '(?is)<(code|tt|samp|kbd|varname|codeph)[^>]*>(.*?)</\1>', '`$2`')
    $s = [regex]::Replace($s, '(?is)<(b|strong)[^>]*>(.*?)</\1>', '**$2**')
    $s = [regex]::Replace($s, '(?is)<(i|em)[^>]*>(.*?)</\1>',     '*$2*')

    # 8. 區塊與換行
    $s = [regex]::Replace($s, '(?is)<br\s*/?>',       "`n")
    $s = [regex]::Replace($s, '(?is)</(p|div|section|tr|table)>', "`n`n")
    $s = [regex]::Replace($s, '(?is)<hr\s*/?>',       "`n---`n")

    # 9. 清掉殘餘標籤與實體
    $s = [regex]::Replace($s, '(?s)<[^>]+>', '')
    $s = [System.Net.WebUtility]::HtmlDecode($s)

    # 10. 還原 code block
    foreach ($k in $vault.Keys) { $s = $s.Replace($k, $vault[$k]) }

    # 11. 空白正規化
    $s = $s -replace "`r`n", "`n"
    $s = [regex]::Replace($s, '[ \t]+\n', "`n")
    $s = [regex]::Replace($s, '\n{3,}', "`n`n")
    $s = [regex]::Replace($s, '(?m)^[ \t]+$', '')

    return $s.Trim()
}

function Get-HtmlTitle {
    param([string]$Html)
    $m = [regex]::Match($Html, '(?is)<title[^>]*>(.*?)</title>')
    if (-not $m.Success) {
        $m = [regex]::Match($Html, '(?is)<h1[^>]*>(.*?)</h1>')
    }
    if ($m.Success) {
        $t = [regex]::Replace($m.Groups[1].Value, '(?s)<[^>]+>', '')
        return ([System.Net.WebUtility]::HtmlDecode($t) -replace '\s+', ' ').Trim()
    }
    return ''
}

#endregion

#region ---------- 符號索引 ----------

function Get-DocLanguage {
    param([string]$EntryPath)
    switch -Regex ($EntryPath) {
        '(?i)lsaz|lotusscript|/ls[a-z]*/' { 'lotusscript'; break }
        '(?i)fozn|formula|/at[a-z]*/'     { 'formula';     break }
        '(?i)javadoc|/java/'              { 'java';        break }
        '(?i)xsp|xpages'                  { 'xpages';      break }
        '(?i)basic|designer'              { 'guide';       break }
        default                           { 'other' }
    }
}

function Get-SymbolRecord {
    param(
        [string]$Title,
        [string]$Markdown,
        [string]$EntryPath,
        [string]$RelMdPath,
        [string]$SourceUrl,
        [string]$Language
    )

    if ([string]::IsNullOrWhiteSpace($Title)) { return $null }

    $symbol = $null
    $kind   = $null

    # @Formula：@DbLookup、@If ...
    $m = [regex]::Match($Title, '^\s*(@[A-Za-z][A-Za-z0-9_]*)')
    if ($m.Success) {
        $symbol = $m.Groups[1].Value
        $kind   = 'function'
    }

    # LotusScript：NotesDatabase class / GetDocumentByUNID method / IsOpen property
    if (-not $symbol) {
        $m = [regex]::Match($Title,
            '^\s*([A-Za-z_][A-Za-z0-9_.]*)\s+(class|method|property|event|constant|statement|function|sub|type)\b',
            'IgnoreCase')
        if ($m.Success) {
            $symbol = $m.Groups[1].Value
            $kind   = $m.Groups[2].Value.ToLower()
        }
    }

    # Java：完整類別名
    if (-not $symbol -and $Language -eq 'java') {
        $m = [regex]::Match($Title, '^\s*(?:Class|Interface)\s+([A-Za-z_][A-Za-z0-9_.]*)', 'IgnoreCase')
        if ($m.Success) {
            $symbol = $m.Groups[1].Value
            $kind   = 'class'
        }
    }

    if (-not $symbol) { return $null }

    # 擷取 Syntax 區段（到下一個同級或更高級標題為止）
    $syntax = ''
    $sm = [regex]::Match($Markdown, '(?ims)^#{1,6}\s*Syntax\s*$(.*?)(?=^#{1,6}\s|\z)')
    if ($sm.Success) {
        $syntax = $sm.Groups[1].Value.Trim()
        if ($syntax.Length -gt 1200) { $syntax = $syntax.Substring(0, 1200) + ' ...' }
    }

    [pscustomobject][ordered]@{
        symbol      = $symbol
        kind        = $kind
        language    = $Language
        title       = $Title
        syntax      = $syntax
        md_path     = $RelMdPath
        entry_path  = $EntryPath
        source_url  = $SourceUrl
    }
}

#endregion

#region ---------- 主流程 ----------

$notes = Resolve-NotesPath -Explicit $NotesPath
Write-Host "Notes 安裝路徑：$notes" -ForegroundColor Cyan

$jars = @(Get-HelpJar -Root $notes -Filter $JarFilter)
if ($jars.Count -eq 0) {
    throw "找不到符合 $($JarFilter -join ', ') 的 jar。試著放寬 -JarFilter，例如 -JarFilter '*.jar'。"
}

Write-Host "找到 $($jars.Count) 個候選 jar。" -ForegroundColor Cyan

# --- ListOnly：只做偵察 ---
if ($ListOnly) {
    $report = foreach ($j in $jars) {
        $zip = $null
        try {
            $zip = [System.IO.Compression.ZipFile]::OpenRead($j.FullName)
            $htmlCount = @($zip.Entries | Where-Object { $_.FullName -match '(?i)\.html?$' }).Count
            [pscustomobject]@{
                Jar       = $j.Name
                SizeMB    = [math]::Round($j.Length / 1MB, 2)
                HtmlFiles = $htmlCount
            }
        }
        catch {
            [pscustomobject]@{ Jar = $j.Name; SizeMB = 0; HtmlFiles = -1 }
        }
        finally { if ($zip) { $zip.Dispose() } }
    }
    $report | Where-Object { $_.HtmlFiles -gt 0 } |
        Sort-Object HtmlFiles -Descending | Format-Table -AutoSize
    Write-Host "`n以上為含 HTML 的 plugin。確認無誤後移除 -ListOnly 正式執行。" -ForegroundColor Yellow
    return
}

# --- 準備輸出目錄 ---
if (Test-Path $OutputPath) {
    if (-not $Force) {
        throw "輸出目錄已存在：$OutputPath（加上 -Force 覆寫）"
    }
    Remove-Item $OutputPath -Recurse -Force
}
$mdRoot = Join-Path $OutputPath 'md'
New-Item -ItemType Directory -Path $mdRoot -Force | Out-Null

$symbolsPath = Join-Path $OutputPath 'symbols.jsonl'
$symbolWriter = New-Object System.IO.StreamWriter($symbolsPath, $false, $Utf8NoBom)

$manifest    = New-Object System.Collections.Generic.List[object]
$totalDocs   = 0
$totalSyms   = 0

foreach ($jar in $jars) {
    $zip = $null
    try {
        $zip = [System.IO.Compression.ZipFile]::OpenRead($jar.FullName)
    }
    catch {
        Write-Warning "無法開啟 $($jar.Name)：$($_.Exception.Message)"
        continue
    }

    $entries = @($zip.Entries | Where-Object { $_.FullName -match '(?i)\.html?$' -and $_.Length -gt 0 })
    if ($entries.Count -eq 0) { $zip.Dispose(); continue }

    # plugin 名稱：去掉版本尾碼
    $pluginName = [regex]::Replace($jar.BaseName, '_\d+(\.\d+)*(\.[A-Za-z0-9\-]+)?$', '')
    $pluginDir  = Join-Path $mdRoot $pluginName
    New-Item -ItemType Directory -Path $pluginDir -Force | Out-Null

    Write-Host "  處理 $pluginName（$($entries.Count) 個 HTML）..." -ForegroundColor DarkGray

    $docCount = 0
    $symCount = 0

    foreach ($e in $entries) {
        try {
            $reader = New-Object System.IO.StreamReader($e.Open(), [System.Text.Encoding]::UTF8)
            $html   = $reader.ReadToEnd()
            $reader.Dispose()
        }
        catch {
            Write-Warning "    讀取失敗 $($e.FullName)"
            continue
        }

        $title = Get-HtmlTitle -Html $html
        $md    = ConvertFrom-HtmlToMarkdown -Html $html
        if ([string]::IsNullOrWhiteSpace($md)) { continue }

        # 相對路徑：拿掉 jar 內常見的 html/ doc/ 前綴
        $rel = $e.FullName -replace '^(?i)(html|doc|docs)/', ''
        $rel = $rel -replace '/', '\'
        $relMd = [System.IO.Path]::ChangeExtension($rel, '.md')

        $outFile = Join-Path $pluginDir $relMd
        $outDir  = Split-Path $outFile -Parent
        if (-not (Test-Path $outDir)) { New-Item -ItemType Directory -Path $outDir -Force | Out-Null }

        $lang      = Get-DocLanguage -EntryPath $e.FullName
        $urlPath   = ($e.FullName -replace '^(?i)(html|doc|docs)/', '')
        $sourceUrl = "https://help.hcl-software.com/dom_designer/$DocVersion/$urlPath"

        $fm = @(
            '---'
            "title: `"$($title -replace '"', '\"')`""
            "plugin: $pluginName"
            "language: $lang"
            "doc_version: `"$DocVersion`""
            "entry_path: `"$($e.FullName)`""
            "source_url: `"$sourceUrl`""
            'tags:'
            '  - domino'
            '  - reference'
            "  - $lang"
            '---'
            ''
        ) -join "`n"

        [System.IO.File]::WriteAllText($outFile, $fm + $md + "`n", $Utf8NoBom)
        $docCount++

        $symRec = Get-SymbolRecord -Title $title -Markdown $md `
                    -EntryPath $e.FullName `
                    -RelMdPath (Join-Path $pluginName $relMd) `
                    -SourceUrl $sourceUrl -Language $lang
        if ($symRec) {
            $symbolWriter.WriteLine(($symRec | ConvertTo-Json -Compress -Depth 4))
            $symCount++
        }

        if ($KeepHtml) {
            $htmlOut = Join-Path (Join-Path $OutputPath 'html') (Join-Path $pluginName $rel)
            $htmlDir = Split-Path $htmlOut -Parent
            if (-not (Test-Path $htmlDir)) { New-Item -ItemType Directory -Path $htmlDir -Force | Out-Null }
            [System.IO.File]::WriteAllText($htmlOut, $html, $Utf8NoBom)
        }
    }

    $zip.Dispose()

    $manifest.Add([pscustomobject][ordered]@{
        plugin    = $pluginName
        jar       = $jar.Name
        jar_path  = $jar.FullName
        documents = $docCount
        symbols   = $symCount
    })

    $totalDocs += $docCount
    $totalSyms += $symCount
}

$symbolWriter.Flush()
$symbolWriter.Dispose()

$manifestObj = [pscustomobject][ordered]@{
    generated_at  = (Get-Date).ToString('s')
    notes_path    = $notes
    doc_version   = $DocVersion
    total_plugins = $manifest.Count
    total_docs    = $totalDocs
    total_symbols = $totalSyms
    plugins       = $manifest
}
[System.IO.File]::WriteAllText(
    (Join-Path $OutputPath 'manifest.json'),
    ($manifestObj | ConvertTo-Json -Depth 5),
    $Utf8NoBom
)

Write-Host ''
Write-Host "完成。" -ForegroundColor Green
Write-Host "  文件：$totalDocs 篇"
Write-Host "  符號：$totalSyms 筆  ->  $symbolsPath"
Write-Host "  輸出：$OutputPath"

#endregion
```


**先偵察再轉換**（第一次一定要跑這步，各版本 plugin 命名不一致）：

```powershell
.\Export-DominoHelp.ps1 -ListOnly
```

會列出每個 plugin 的 HTML 篇數。確認清單合理後：


```powershell
.\Export-DominoHelp.ps1 -OutputPath D:\kb\domino -DocVersion 11.0.1
```

如果 `-ListOnly` 抓不到東西，放寬過濾：`-JarFilter '*.jar'`（會慢很多，但保證掃到）。

**設計上的幾個重點**

- **不解壓到磁碟** — 直接從 zip entry 串流讀，help plugin 動輒數百 MB，避免佔空間也快得多
- **`<pre>` 先鎖起來** — 轉換前把程式碼區塊抽出存進 vault，最後才還原。不然 LotusScript 裡的 `<` `>` 會被當標籤吃掉，這是 HTML→MD 最常爆的地方
- **front-matter 帶 `source_url`** — 重建回 `help.hcl-software.com` 的線上網址，RAG 引用時可以直接給連結，人也點得進去驗證
- **`symbols.jsonl` 是重點** — 這就是你要的 deterministic index。它解析標題模式（`NotesDatabase class`、`GetDocumentByUNID method`、`@DbLookup`）抽出符號名 + kind + Syntax 區段