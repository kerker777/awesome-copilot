---
mode: 'agent'
tools: ['search/codebase', 'edit/editFiles', 'terminalCommand']
description: 'Containerize an ASP.NET .NET Framework project by creating Dockerfile and .dockerfile files customized for the project.'
---

# ASP.NET .NET Framework 容器化提示

依照下方的容器化設定，對指定的 ASP.NET (.NET Framework) 專案進行容器化，**專注於**確保應用程式能在 Windows Docker 容器中執行所需的變更。容器化應納入此處指定的所有設定。

**記住：** 這是 .NET Framework 應用程式，不是 .NET Core。容器化流程與 .NET Core 應用程式不同。

## 容器化設定

本提示的此部分包含容器化 ASP.NET (.NET Framework) 應用程式所需的特定設定和組態。執行本提示前，請確保設定已填入必要資訊。請注意，許多情況下只需要前幾個設定。如果後續設定不適用於要容器化的專案，可保留為預設值。

未指定的任何設定將設為預設值。預設值以 `[方括弧]` 表示。

### 基本專案資訊
1. 要容器化的專案：
   - `[專案名稱（提供 .csproj 檔案路徑）]`

2. 要使用的 Windows Server SKU：
   - `[Windows Server Core（預設）或 Windows Server Full]`

3. 要使用的 Windows Server 版本：
   - `[2022、2019 或 2016（預設 2022）]`

4. Docker 映像建構階段的自訂基礎映像（輸入「None」可使用標準 Microsoft 基礎映像）：
   - `[指定用於建構階段的基礎映像（預設 None）]`

5. Docker 映像執行階段的自訂基礎映像（輸入「None」可使用標準 Microsoft 基礎映像）：
   - `[指定用於執行階段的基礎映像（預設 None）]`   

### 容器組態
1. 必須在容器映像中公開的連接埠：
   - 主 HTTP 連接埠：`[例如，80]`
   - 額外連接埠：`[列出任何額外連接埠，或「None」]`

2. 容器應執行的使用者帳戶：
   - `[使用者帳戶，或預設為「ContainerUser」]`

3. 必須在容器映像中設定的 IIS 設定：
   - `[列出任何特定 IIS 設定，或「None」]`

### 組建組態
1. 建構容器映像前必須執行的自訂組建步驟：
   - `[列出任何特定組建步驟，或「None」]`

2. 建構容器映像後必須執行的自訂組建步驟：
   - `[列出任何特定組建步驟，或「None」]`

### 相依性
1. 應在容器映像中向 GAC 註冊的 .NET 組件：
   - `[組件名稱和版本，或「None」]`

2. 必須複製到容器映像並安裝的 MSI：
   - `[MSI 名稱和版本，或「None」]`

3. 必須在容器映像中註冊的 COM 元件：
   - `[COM 元件名稱，或「None」]`

### 系統組態
1. 必須新增至容器映像的登錄機碼和值：
   - `[登錄路徑和值，或「None」]`

2. 必須在容器映像中設定的環境變數：
   - `[變數名稱和值，或「使用預設值」]`

3. 必須在容器映像中安裝的 Windows Server 角色和功能：
   - `[角色/功能名稱，或「None」]`

### 檔案系統
1. 需要複製到容器映像的檔案/目錄：
   - `[相對於專案根目錄的路徑，或「None」]`
   - 容器中的目標位置：`[容器路徑，或「不適用」]`

2. 要從容器化中排除的檔案/目錄：
   - `[要排除的路徑，或「None」]`

### .dockerignore 組態
1. 要納入 `.dockerignore` 檔案的模式（.dockerignore 已包含常見預設值；這些是額外模式）：
   - 額外模式：`[列出任何額外模式，或「None」]`

### 健康檢查組態
1. 健康檢查端點：
   - `[健康檢查 URL 路徑，或「None」]`

2. 健康檢查間隔和逾時：
   - `[間隔和逾時值，或「使用預設值」]`

### 額外指令
1. 必須遵循以容器化專案的其他指令：
   - `[具體需求，或「None」]`

2. 需要解決的已知問題：
   - `[描述任何已知問題，或「None」]`

## 範圍

- ✅ 應用程式組態修改，確保使用組態建構程式從環境變數讀取應用程式設定和連接字串
- ✅ ASP.NET 應用程式的 Dockerfile 建立和組態
- ✅ 在 Dockerfile 中指定多個階段以建構/發佈應用程式，並將輸出複製到最終映像
- ✅ Windows 容器平台相容性組態（Windows Server Core 或 Full）
- ✅ 適當處理相依性（GAC 組件、MSI、COM 元件）
- ❌ 不涉及基礎結構設定（假設另行處理）
- ❌ 除容器化所需外，無其他程式碼變更

## 執行程序

1. 檢閱上述容器化設定以了解容器化需求
2. 建立 `progress.md` 檔案以追蹤包含勾選標記的變更
3. 通過檢查 `.csproj` 檔案中的 `TargetFrameworkVersion` 元素來決定 .NET Framework 版本
4. 根據以下項目選擇適當的 Windows Server 容器映像：
   - 從專案偵測到的 .NET Framework 版本
   - 容器化設定中指定的 Windows Server SKU（Core 或 Full）
   - 容器化設定中指定的 Windows Server 版本（2016、2019 或 2022）
   - Windows Server Core 標籤可在此找到：https://github.com/microsoft/dotnet-framework-docker/blob/main/README.aspnet.md#full-tag-listing
5. 確保已安裝必要的 NuGet 套件。**不要**在遺失時安裝。如果未安裝，使用者必須手動安裝。如果未安裝，請暫停執行此提示並要求使用者使用 Visual Studio NuGet 套件管理員或 Visual Studio 套件管理員主控台安裝。以下套件為必要：
   - `Microsoft.Configuration.ConfigurationBuilders.Environment`
6. 修改 `web.config` 檔案以新增組態建構程式區段和設定，從環境變數讀取應用程式設定和連接字串：
   - 在 configSections 中新增 ConfigBuilders 區段
   - 在根目錄中新增 configBuilders 區段
   - 為 appSettings 和 connectionStrings 設定 EnvironmentConfigBuilder
   - 範例模式：
     ```xml
     <configSections>
       <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
     </configSections>
     <configBuilders>
       <builders>
         <add name="Environment" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
       </builders>
     </configBuilders>
     <appSettings configBuilders="Environment">
       <!-- existing app settings -->
     </appSettings>
     <connectionStrings configBuilders="Environment">
       <!-- existing connection strings -->
     </connectionStrings>
     ```
7. 通過複製本提示結尾的參考 `LogMonitorConfig.json` 檔案，在 Dockerfile 所在的資料夾中建立 `LogMonitorConfig.json` 檔案。檔案內容**不得**修改，應完全符合參考內容，除非容器化設定中的指令另有說明。
   - 特別是，確保不要更改要記錄的問題等級，因為對 EventLog 來源使用 `Information` 等級會造成不必要的噪音。
8. 在專案目錄的根目錄中建立 Dockerfile 以容器化應用程式
   - Dockerfile 應使用多個階段：
     - 組建階段：使用 Windows Server Core 映像來建構應用程式
       - 組建階段必須使用 `mcr.microsoft.com/dotnet/framework/sdk` 基礎映像，除非設定檔中指定了自訂基礎映像
       - 先複製 sln、csproj 和 packages.config 檔案
       - 複製 NuGet.config（如果存在），並組態任何私密摘要
       - 還原 NuGet 套件
       - 然後，複製其餘原始程式碼，並使用 MSBuild 建構並發佈應用程式到 C:\publish
     - 最終階段：使用選定的 Windows Server 映像執行應用程式
       - 最終階段必須使用 `mcr.microsoft.com/dotnet/framework/aspnet` 基礎映像，除非設定檔中指定了自訂基礎映像
       - 將 `LogMonitorConfig.json` 檔案複製到容器中的目錄（例如 C:\LogMonitor）
       - 從 Microsoft 儲存庫下載 LogMonitor.exe 到相同目錄
           - 正確的 LogMonitor.exe URL 為：https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe
       - 將工作目錄設為 C:\inetpub\wwwroot
       - 將來自組建階段的發佈輸出（位於 C:\publish）複製到最終映像
       - 設定容器的進入點以執行 LogMonitor.exe 搭配 ServiceMonitor.exe 來監控 IIS 服務
           - `ENTRYPOINT [ "C:\\LogMonitor\\LogMonitor.exe", "C:\\ServiceMonitor.exe", "w3svc" ]`
   - 務必考慮容器化設定中的所有需求：
     - Windows Server SKU 和版本
     - 公開的連接埠
     - 容器的使用者帳戶
     - IIS 設定
     - GAC 組件註冊
     - MSI 安裝
     - COM 元件註冊
     - 登錄機碼
     - 環境變數
     - Windows 角色和功能
     - 檔案/目錄複製
   - 以本提示結尾提供的範例為 Dockerfile 的模型，但確保根據特定專案需求和設定進行自訂。
   - **重要：** 除非使用者在設定檔中**明確要求**完整 Windows Server 映像，否則使用 Windows Server Core 基礎映像
9. 在專案目錄的根目錄中建立 `.dockerignore` 檔案，以從 Docker 映像中排除不必要的檔案。`.dockerignore` 檔案**必須**至少包含以下元素，以及容器化設定中指定的額外模式：
   - packages/
   - bin/
   - obj/
   - .dockerignore
   - Dockerfile
   - .git/
   - .github/
   - .vs/
   - .vscode/
   - **/node_modules/
   - *.user
   - *.suo
   - **/.DS_Store
   - **/Thumbs.db
   - 容器化設定中指定的任何額外模式
10. 如果在設定中指定了健康檢查，請進行設定：
   - 如果提供了健康檢查端點，在 Dockerfile 中新增 HEALTHCHECK 指令
11. 通過在專案檔案中新增以下項目，將 Dockerfile 新增至專案：`<None Include="Dockerfile" />`
12. 將工作標記為已完成：[ ] → [✓]
13. 繼續進行，直到所有工作完成且 Docker 建置成功

## 組建與執行時期驗證

確認 Dockerfile 完成後 Docker 建置成功。使用以下命令建置 Docker 映像：

```bash
docker build -t aspnet-app:latest .
```

如果建置失敗，請檢查錯誤訊息，並對 Dockerfile 或專案組態做出必要的調整。回報成功/失敗。

## 進度追蹤

維持 `progress.md` 檔案，採用下列結構：
```markdown
# 容器化進度

## 環境偵測
- [ ] .NET Framework 版本偵測（版本：___）
- [ ] Windows Server SKU 選擇（SKU：___）
- [ ] Windows Server 版本選擇（版本：___）

## 組態變更
- [ ] Web.config 對組態建構程式的修改
- [ ] NuGet 套件來源組態（若適用）
- [ ] 複製 LogMonitorConfig.json 及視設定要求進行調整

## 容器化
- [ ] Dockerfile 建立
- [ ] .dockerignore 檔案建立
- [ ] 使用 SDK 映像建立的組建階段
- [ ] 已複製 sln、csproj、packages.config 及（如適用）NuGet.config 以進行套件還原
- [ ] 使用執行時期映像建立的執行階段
- [ ] 非 root 使用者組態
- [ ] 相依性處理（GAC、MSI、COM、登錄、額外檔案等）
- [ ] 健康檢查組態（若適用）
- [ ] 特殊需求實作

## 驗證
- [ ] 檢閱容器化設定並確保符合所有需求
- [ ] Docker 建置成功
```

請勿在各步驟間暫停確認。持續有條不紊地進行，直到應用程式已容器化且 Docker 建置成功。

**在所有核取方塊都標記完成之前，你尚未完成！** 這包括成功建置 Docker 映像，以及解決建置程序中出現的任何問題。

## 參考資料

### 範例 Dockerfile

使用 Windows Server Core 基礎映像的 ASP.NET (.NET Framework) 應用程式範例 Dockerfile。

```dockerfile
# escape=`
# The escape directive changes the escape character from \ to `
# This is especially useful in Windows Dockerfiles where \ is the path separator

# ============================================================
# Stage 1: Build and publish the application
# ============================================================

# Base Image - Select the appropriate .NET Framework version and Windows Server Core version
# Possible tags include:
# - 4.8.1-windowsservercore-ltsc2025 (Windows Server 2025)
# - 4.8-windowsservercore-ltsc2022 (Windows Server 2022)
# - 4.8-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.8-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.2-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.7.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.1-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.6.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 3.5-windowsservercore-ltsc2025 (Windows Server 2025)
# - 3.5-windowsservercore-ltsc2022 (Windows Server 2022)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2019)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2016)
# Uses the .NET Framework SDK image for building the application
FROM mcr.microsoft.com/dotnet/framework/sdk:4.8-windowsservercore-ltsc2022 AS build
ARG BUILD_CONFIGURATION=Release

# Set the default shell to PowerShell
SHELL ["powershell", "-command"]

WORKDIR /app

# Copy the solution and project files
COPY YourSolution.sln .
COPY YourProject/*.csproj ./YourProject/
COPY YourOtherProject/*.csproj ./YourOtherProject/

# Copy packages.config files
COPY YourProject/packages.config ./YourProject/
COPY YourOtherProject/packages.config ./YourOtherProject/

# Restore NuGet packages
RUN nuget restore YourSolution.sln

# Copy source code
COPY . .

# Perform custom pre-build steps here, if needed

# Build and publish the application to C:\publish
RUN msbuild /p:Configuration=$BUILD_CONFIGURATION `
            /p:WebPublishMethod=FileSystem `
            /p:PublishUrl=C:\publish `
            /p:DeployDefaultTarget=WebPublish

# Perform custom post-build steps here, if needed

# ============================================================
# Stage 2: Final runtime image
# ============================================================

# Base Image - Select the appropriate .NET Framework version and Windows Server Core version
# Possible tags include:
# - 4.8.1-windowsservercore-ltsc2025 (Windows Server 2025)
# - 4.8-windowsservercore-ltsc2022 (Windows Server 2022)
# - 4.8-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.8-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.2-windowsservercore-ltsc2019 (Windows Server 2019)
# - 4.7.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7.1-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.7-windowsservercore-ltsc2016 (Windows Server 2016)
# - 4.6.2-windowsservercore-ltsc2016 (Windows Server 2016)
# - 3.5-windowsservercore-ltsc2025 (Windows Server 2025)
# - 3.5-windowsservercore-ltsc2022 (Windows Server 2022)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2019)
# - 3.5-windowsservercore-ltsc2019 (Windows Server 2016)
# Uses the .NET Framework ASP.NET image for running the application
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2022

# Set the default shell to PowerShell
SHELL ["powershell", "-command"]

WORKDIR /inetpub/wwwroot

# Copy from build stage
COPY --from=build /publish .

# Add any additional environment variables needed for your application (uncomment and modify as needed)
# ENV KEY=VALUE

# Install MSI packages (uncomment and modify as needed)
# COPY ./msi-installers C:/Installers
# RUN Start-Process -Wait -FilePath 'msiexec.exe' -ArgumentList '/i', 'C:\Installers\your-package.msi', '/quiet', '/norestart'

# Install custom Windows Server roles and features (uncomment and modify as needed)
# RUN dism /Online /Enable-Feature /FeatureName:YOUR-FEATURE-NAME

# Add additional Windows features (uncomment and modify as needed)
# RUN Add-WindowsFeature Some-Windows-Feature; `
#    Add-WindowsFeature Another-Windows-Feature

# Install MSI packages if needed (uncomment and modify as needed)
# COPY ./msi-installers C:/Installers
# RUN Start-Process -Wait -FilePath 'msiexec.exe' -ArgumentList '/i', 'C:\Installers\your-package.msi', '/quiet', '/norestart'

# Register assemblies in GAC if needed (uncomment and modify as needed)
# COPY ./assemblies C:/Assemblies
# RUN C:\Windows\Microsoft.NET\Framework64\v4.0.30319\gacutil -i C:/Assemblies/YourAssembly.dll

# Register COM components if needed (uncomment and modify as needed)
# COPY ./com-components C:/Components
# RUN regsvr32 /s C:/Components/YourComponent.dll

# Add registry keys if needed (uncomment and modify as needed)
# RUN New-Item -Path 'HKLM:\Software\YourApp' -Force; `
#     Set-ItemProperty -Path 'HKLM:\Software\YourApp' -Name 'Setting' -Value 'Value'

# Configure IIS settings if needed (uncomment and modify as needed)
# RUN Import-Module WebAdministration; `
#     Set-ItemProperty 'IIS:\AppPools\DefaultAppPool' -Name somePropertyName -Value 'SomePropertyValue'; `
#     Set-ItemProperty 'IIS:\Sites\Default Web Site' -Name anotherPropertyName -Value 'AnotherPropertyValue'

# Expose necessary ports - By default, IIS uses port 80
EXPOSE 80
# EXPOSE 443  # Uncomment if using HTTPS

# Copy LogMonitor from the microsoft/windows-container-tools repository
WORKDIR /LogMonitor
RUN curl -fSLo LogMonitor.exe https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe

# Copy LogMonitorConfig.json from local files
COPY LogMonitorConfig.json .

# Set non-administrator user
USER ContainerUser

# Override the container's default entry point to take advantage of the LogMonitor
ENTRYPOINT [ "C:\\LogMonitor\\LogMonitor.exe", "C:\\ServiceMonitor.exe", "w3svc" ]
```

## 調整此範例

**注意：** 根據容器化設定中的特定需求自訂此範本。

調整此範例 Dockerfile 時：

1. 使用實際檔案名稱取代 `YourSolution.sln`、`YourProject.csproj` 等
2. 視需要調整 Windows Server 和 .NET Framework 版本
3. 根據需求修改相依性安裝步驟，並移除任何不必要的步驟
4. 根據特定工作流程視需要新增或移除階段

## 階段命名附註

- `AS stage-name` 語法為每個階段指定名稱
- 使用 `--from=stage-name` 從先前的階段複製檔案
- 可有多個中間階段不用於最終映像

### LogMonitorConfig.json

LogMonitorConfig.json 檔案應建立在專案目錄的根目錄中。它用於設定 LogMonitor 工具，該工具在容器中監控日誌。此檔案的內容應完全如下所示，以確保適當的記錄功能：
```json
{
  "LogConfig": {
    "sources": [
      {
        "type": "EventLog",
        "startAtOldestRecord": true,
        "eventFormatMultiLine": false,
        "channels": [
          {
            "name": "system",
            "level": "Warning"
          },
          {
            "name": "application",
            "level": "Error"
          }
        ]
      },
      {
        "type": "File",
        "directory": "c:\\inetpub\\logs",
        "filter": "*.log",
        "includeSubdirectories": true,
        "includeFileNames": false
      },
      {
        "type": "ETW",
        "eventFormatMultiLine": false,
        "providers": [
          {
            "providerName": "IIS: WWW Server",
            "providerGuid": "3A2A4E84-4C21-4981-AE10-3FDA0D9B0F83",
            "level": "Information"
          },
          {
            "providerName": "Microsoft-Windows-IIS-Logging",
            "providerGuid": "7E8AD27F-B271-4EA2-A783-A47BDE29143B",
            "level": "Information"
          }
        ]
      }
    ]
  }
}
```
