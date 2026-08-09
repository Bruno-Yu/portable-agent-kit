

你說的是「跳轉到定義後，回到原本位置」的快捷鍵，答案如下：

---

### 🔙 返回上一個位置

|作業系統|快捷鍵|
|---|---|
|**Windows / Linux**|`Alt + ←`|
|**Mac**|`Ctrl + -`|

> 對應功能：**Go Back**（返回上一個游標位置）

反向操作（往前跳）：

|作業系統|快捷鍵|
|---|---|
|**Windows / Linux**|`Alt + →`|
|**Mac**|`Ctrl + Shift + -`|

---

### 📌 其他常用導航快捷鍵

|功能|Windows / Linux|Mac|
|---|---|---|
|**跳到定義**|`F12`|`F12`|
|**查看定義（Peek）**|`Alt + F12`|`Option + F12`|
|**跳到實作**|`Ctrl + F12`|`Cmd + F12`|
|**查找所有參考**|`Shift + F12`|`Shift + F12`|
|**回到上一個位置**|`Alt + ←`|`Ctrl + -`|
|**前進到下一個位置**|`Alt + →`|`Ctrl + Shift + -`|
|**快速開啟檔案**|`Ctrl + P`|`Cmd + P`|
|**命令面板**|`Ctrl + Shift + P`|`Cmd + Shift + P`|
|**跳到某一行**|`Ctrl + G`|`Ctrl + G`|
|**符號搜尋**|`Ctrl + Shift + O`|`Cmd + Shift + O`|



``` powershell
# 排除 Backup 資料夾，並正確處理唯讀檔案
Get-ChildItem -Path "D:\CIM12A_PI\Ken\Code\QTAP_TH_WIN_Dev_AUTOPOC2" -Recurse -Include "*.cs","*.vb","*.h","*.cpp" |
Where-Object { $_.FullName -notlike "*\Backup\*" } |   # ← 排除 Backup 資料夾
ForEach-Object {
    $bytes = [System.IO.File]::ReadAllBytes($_.FullName)

    # 先判斷是否已經是 UTF-8 with BOM
    if ($bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF) {
        Write-Host "⏭️  略過（已是 UTF-8 BOM）：$($_.FullName)"
    }
    else {
        # 檢查是否為唯讀
        $isReadOnly = $_.IsReadOnly

        try {
            # 如果是唯讀，先移除唯讀屬性
            if ($isReadOnly) {
                Set-ItemProperty $_.FullName -Name IsReadOnly -Value $false
                Write-Host "🔓 移除唯讀：$($_.Name)"
            }

            # 轉換編碼
            $content = Get-Content $_.FullName -Encoding Default
            $content | Set-Content $_.FullName -Encoding UTF8

            # 轉換完成後還原唯讀屬性
            if ($isReadOnly) {
                Set-ItemProperty $_.FullName -Name IsReadOnly -Value $true
            }

            Write-Host "✅ 已轉換：$($_.FullName)"
        }
        catch {
            Write-Host "❌ 轉換失敗：$($_.FullName) → $($_.Exception.Message)"
        }
    }
}

```