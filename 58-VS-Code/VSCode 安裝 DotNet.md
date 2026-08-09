

- **C#**	用於在 VS Code 中開發 C# 程式，具有完整的語言支持，包括 IntelliSense 和偵錯功能。
- **C# Dev Kit**	額外增強 C# 開發體驗，提供模板、程式碼範例和性能分析工具。
- **NuGet Package Manager**	管理 NuGet 套件，方便搜尋、安裝、更新或移除專案中的依賴項。
- **Error Lens**	在程式碼行內顯示即時的錯誤與警告，使除錯更快速和直觀。

API 的 Properties > launchSettings.json 增加設定 (否則無法 F5)

    "Project": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "http://app-dev.example.test:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
在 appsettings.Development.json Serilog 加入 Console 模式 (否則無法自動開啟 Swagger UI Url)

                        {
                            "Name": "Console",
                            "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message}{NewLine}{Exception}"
                        }
