---
mode: 'agent'
tools: ['search/codebase', 'edit/editFiles', 'terminalCommand']
description: 'Containerize an ASP.NET Core project by creating Dockerfile and .dockerfile files customized for the project.'
---

# ASP.NET Core Docker 容器化提示

## 容器化需求

將下方設定中指定的 ASP.NET Core (.NET) 專案進行容器化，專注於**僅進行**應用程式在 Linux Docker 容器中執行所需的變更。容器化應考慮此處指定的所有設定。

遵循 .NET Core 應用程式容器化的最佳實踐，確保容器在效能、安全性和可維護性方面都經過最佳化。

## 容器化設定

本提示的此部分包含對 ASP.NET Core 應用程式進行容器化所需的特定設定和配置。在執行本提示之前，請確保使用必要資訊填寫設定。請注意，在許多情況下，只需要前幾個設定。如果不適用於您要容器化的專案，後續設定可以保持預設值。

任何未指定的設定將被設定為預設值。預設值以 `[方括號]` 提供。

### 基本專案資訊
1. 要容器化的專案：
   - `[ProjectName (提供 .csproj 檔案的路徑)]`

2. 要使用的 .NET 版本：
   - `[8.0 或 9.0 (預設為 8.0)]`

3. 要使用的 Linux 發行版：
   - `[debian、alpine、ubuntu、chiseled 或 Azure Linux (mariner)（預設為 debian）]`

4. Docker 映像建置階段的自訂基礎映像（選擇「None」以使用標準 Microsoft 基礎映像）：
   - `[指定建置階段要使用的基礎映像（預設為 None）]`

5. Docker 映像執行階段的自訂基礎映像（選擇「None」以使用標準 Microsoft 基礎映像）：
   - `[指定執行階段要使用的基礎映像（預設為 None）]`   

### 容器配置
1. 必須在容器映像中公開的連接埠：
   - 主要 HTTP 連接埠：`[例如 8080]`
   - 其他連接埠：`[列出任何其他連接埠，或「None」]`

2. 容器應執行的使用者帳戶：
   - `[使用者帳戶，或預設為「$APP_UID」]`

3. 應用程式 URL 配置：
   - `[指定 ASPNETCORE_URLS，或預設為「http://+:8080」]`

### 建置配置
1. 必須在建置容器映像之前執行的自訂建置步驟：
   - `[列出任何特定的建置步驟，或「None」]`

2. 必須在建置容器映像之後執行的自訂建置步驟：
   - `[列出任何特定的建置步驟，或「None」]`

3. 必須配置的 NuGet 套件來源：
   - `[列出任何具有驗證詳細資料的私有 NuGet 源，或「None」]`

### 依賴項
1. 必須在容器映像中安裝的系統套件：
   - `[所選 Linux 發行版的套件名稱，或「None」]`

2. 必須複製到容器映像的原生程式庫：
   - `[程式庫名稱和路徑，或「None」]`

3. 必須安裝的其他 .NET 工具：
   - `[工具名稱和版本，或「None」]`

### 系統配置
1. 必須在容器映像中設定的環境變數：
   - `[變數名稱和值，或「Use defaults」]`

### 檔案系統
1. 需要複製到容器映像的檔案/目錄：
   - `[相對於專案根目錄的路徑，或「None」]`
   - 容器中的目標位置：`[容器路徑，或「Not applicable」]`

2. 要從容器化中排除的檔案/目錄：
   - `[要排除的路徑，或「None」]`

3. 應該配置的磁碟區掛載點：
   - `[永續資料的磁碟區路徑，或「None」]`

### .dockerignore 配置
1. 要包含在 `.dockerignore` 檔案中的模式（.dockerignore 已有常見預設值；這些是額外模式）：
   - 額外模式：`[列出任何額外模式，或「None」]`

### 健康檢查配置
1. 健康檢查端點：
   - `[健康檢查 URL 路徑，或「None」]`

2. 健康檢查間隔和超時：
   - `[間隔和超時值，或「Use defaults」]`

### 額外指示
1. 必須遵循以容器化專案的其他指示：
   - `[特定需求，或「None」]`

2. 要解決的已知問題：
   - `[描述任何已知問題，或「None」]`

## 範圍

- ✅ 應用程式配置修改，以確保可以從環境變數讀取應用程式設定和連線字串
- ✅ 為 ASP.NET Core 應用程式建立和配置 Dockerfile
- ✅ 在 Dockerfile 中指定多個階段以建置/發佈應用程式，並將輸出複製到最終映像
- ✅ Linux 容器平台相容性配置（Alpine、Ubuntu、Chiseled 或 Azure Linux (Mariner)）
- ✅ 依賴項的適當處理（系統套件、原生程式庫、其他工具）
- ❌ 無基礎架構設置（假定由別處處理）
- ❌ 除了容器化所需的變更外，無其他程式碼變更

## 執行流程

1. 檢閱上方的容器化設定，以了解容器化需求
2. 建立 `progress.md` 檔案以使用核取標記追蹤變更
3. 通過檢查 `TargetFramework` 元素，從專案的 .csproj 檔案確定 .NET 版本
4. 根據以下因素選擇適當的 Linux 容器映像：
   - 從專案偵測到的 .NET 版本
   - 在容器化設定中指定的 Linux 發行版（Alpine、Ubuntu、Chiseled 或 Azure Linux (Mariner)）
   - 如果使用者未在容器化設定中要求特定的基礎映像，則基礎映像必須是有效的 mcr.microsoft.com/dotnet 映像，其標籤如下方示例 Dockerfile 或文件中所示
   - 用於建置和執行時階段的官方 Microsoft .NET 映像：
      - SDK 映像標籤（用於建置階段）：https://github.com/dotnet/dotnet-docker/blob/main/README.sdk.md
      - ASP.NET Core 執行時間映像標籤：https://github.com/dotnet/dotnet-docker/blob/main/README.aspnet.md
      - .NET 執行時間映像標籤：https://github.com/dotnet/dotnet-docker/blob/main/README.runtime.md
5. 在專案目錄的根目錄中建立 Dockerfile 以容器化應用程式
   - Dockerfile 應使用多個階段：
     - 建置階段：使用 .NET SDK 映像建置應用程式
       - 先複製 csproj 檔案
       - 如果存在 NuGet.config 則複製，並配置任何私有源
       - 還原 NuGet 套件
       - 然後複製其餘原始程式碼，並建置和發佈應用程式至 /app/publish
     - 最終階段：使用選定的 .NET 執行時間映像執行應用程式
       - 將工作目錄設定為 /app
       - 按指示設定使用者（預設為非 root 使用者（例如 `$APP_UID`））
         - 除非在容器化設定中另有指示，否則不需要建立新使用者。使用 `$APP_UID` 變數指定使用者帳戶。
       - 從建置階段將已發佈的輸出複製到最終映像
   - 務必考慮容器化設定中的所有需求：
     - .NET 版本和 Linux 發行版
     - 公開的連接埠
     - 容器的使用者帳戶
     - ASPNETCORE_URLS 配置
     - 系統套件安裝
     - 原生程式庫依賴項
     - 其他 .NET 工具
     - 環境變數
     - 檔案/目錄複製
     - 磁碟區掛載點
     - 健康檢查配置
6. 在專案目錄的根目錄中建立 `.dockerignore` 檔案以排除不必要的檔案。`.dockerignore` 檔案**必須**至少包含以下元素以及在容器化設定中指定的其他模式：
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
   - 在容器化設定中指定的任何額外模式
7. 如果在容器化設定中指定，請配置健康檢查：
   - 如果提供了健康檢查端點，請將 HEALTHCHECK 指示新增到 Dockerfile
   - 使用 curl 或 wget 檢查健康端點
8. 標記任務為已完成：[ ] → [✓]
9. 繼續直到所有任務完成且 Docker 建置成功

## 建置和執行時間驗證

一旦完成 Dockerfile，請確認 Docker 建置成功。使用以下命令建置 Docker 映像：

```bash
docker build -t aspnetcore-app:latest .
```

如果建置失敗，請檢閱錯誤訊息並對 Dockerfile 或專案配置進行必要的調整。報告成功/失敗。

## 進度追蹤

維持 `progress.md` 檔案，結構如下：
```markdown
# 容器化進度

## 環境偵測
- [ ] .NET 版本偵測（版本：___）
- [ ] Linux 發行版選擇（發行版：___）

## 配置變更
- [ ] 應用程式配置驗證以取得環境變數支援
- [ ] NuGet 套件來源配置（如適用）

## 容器化
- [ ] Dockerfile 建立
- [ ] .dockerignore 檔案建立
- [ ] 使用 SDK 映像建立的建置階段
- [ ] 為套件還原複製的 csproj 檔案
- [ ] 如適用複製的 NuGet.config
- [ ] 使用執行時間映像建立的執行時間階段
- [ ] 非 root 使用者配置
- [ ] 依賴項處理（系統套件、原生程式庫、工具等）
- [ ] 健康檢查配置（如適用）
- [ ] 特殊需求實施

## 驗證
- [ ] 檢閱容器化設定並確保滿足所有需求
- [ ] Docker 建置成功
```

不要在步驟之間暫停等待確認。系統地繼續進行，直到應用程式已容器化且 Docker 建置成功。

**在所有核取方塊都被標記之前，您還沒有完成！** 這包括成功建置 Docker 映像以及解決建置流程中出現的任何問題。

## 範例 Dockerfile

使用 Linux 基礎映像的 ASP.NET Core (.NET) 應用程式的範例 Dockerfile。

```dockerfile
# ============================================================
# Stage 1: Build and publish the application
# ============================================================

# Base Image - Select the appropriate .NET SDK version and Linux distribution
# Possible tags include:
# - 8.0-bookworm-slim (Debian 12)
# - 8.0-noble (Ubuntu 24.04)
# - 8.0-alpine (Alpine Linux)
# - 9.0-bookworm-slim (Debian 12)
# - 9.0-noble (Ubuntu 24.04)
# - 9.0-alpine (Alpine Linux)
# Uses the .NET SDK image for building the application
FROM mcr.microsoft.com/dotnet/sdk:8.0-bookworm-slim AS build
ARG BUILD_CONFIGURATION=Release

WORKDIR /src

# Copy project files first for better caching
COPY ["YourProject/YourProject.csproj", "YourProject/"]
COPY ["YourOtherProject/YourOtherProject.csproj", "YourOtherProject/"]

# Copy NuGet configuration if it exists
COPY ["NuGet.config", "."]

# Restore NuGet packages
RUN dotnet restore "YourProject/YourProject.csproj"

# Copy source code
COPY . .

# Perform custom pre-build steps here, if needed
# RUN echo "Running pre-build steps..."

# Build and publish the application
WORKDIR "/src/YourProject"
RUN dotnet build "YourProject.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Publish the application
RUN dotnet publish "YourProject.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Perform custom post-build steps here, if needed
# RUN echo "Running post-build steps..."

# ============================================================
# Stage 2: Final runtime image
# ============================================================

# Base Image - Select the appropriate .NET runtime version and Linux distribution
# Possible tags include:
# - 8.0-bookworm-slim (Debian 12)
# - 8.0-noble (Ubuntu 24.04)
# - 8.0-alpine (Alpine Linux)
# - 8.0-noble-chiseled (Ubuntu 24.04 Chiseled)
# - 8.0-azurelinux3.0 (Azure Linux)
# - 9.0-bookworm-slim (Debian 12)
# - 9.0-noble (Ubuntu 24.04)
# - 9.0-alpine (Alpine Linux)
# - 9.0-noble-chiseled (Ubuntu 24.04 Chiseled)
# - 9.0-azurelinux3.0 (Azure Linux)
# Uses the .NET runtime image for running the application
FROM mcr.microsoft.com/dotnet/aspnet:8.0-bookworm-slim AS final

# Install system packages if needed (uncomment and modify as needed)
# RUN apt-get update && apt-get install -y \
#     curl \
#     wget \
#     ca-certificates \
#     libgdiplus \
#     && rm -rf /var/lib/apt/lists/*

# Install additional .NET tools if needed (uncomment and modify as needed)
# RUN dotnet tool install --global dotnet-ef --version 8.0.0
# ENV PATH="$PATH:/root/.dotnet/tools"

WORKDIR /app

# Copy published application from build stage
COPY --from=build /app/publish .

# Copy additional files if needed (uncomment and modify as needed)
# COPY ./config/appsettings.Production.json .
# COPY ./certificates/ ./certificates/

# Set environment variables
ENV ASPNETCORE_ENVIRONMENT=Production
ENV ASPNETCORE_URLS=http://+:8080

# Add custom environment variables if needed (uncomment and modify as needed)
# ENV CONNECTIONSTRINGS__DEFAULTCONNECTION="your-connection-string"
# ENV FEATURE_FLAG_ENABLED=true

# Configure SSL/TLS certificates if needed (uncomment and modify as needed)
# ENV ASPNETCORE_Kestrel__Certificates__Default__Path=/app/certificates/app.pfx
# ENV ASPNETCORE_Kestrel__Certificates__Default__Password=your_password

# Expose the port the application listens on
EXPOSE 8080
# EXPOSE 8081  # Uncomment if using HTTPS

# Install curl for health checks if not already present
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Configure health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Create volumes for persistent data if needed (uncomment and modify as needed)
# VOLUME ["/app/data", "/app/logs"]

# Switch to non-root user for security
USER $APP_UID

# Set the entry point for the application
ENTRYPOINT ["dotnet", "YourProject.dll"]
```

## 調適此範例

**注意：** 根據容器化設定中的具體需求自訂此範本。

調適此範例 Dockerfile 時：

1. 用您的實際專案名稱取代 `YourProject.csproj`、`YourProject.dll` 等
2. 根據需要調整 .NET 版本和 Linux 發行版
3. 根據您的需求修改依賴項安裝步驟，並移除任何不必要的步驟
4. 配置應用程式特定的環境變數
5. 根據您的特定工作流程需要新增或移除階段
6. 更新健康檢查端點以符合您應用程式的健康檢查路徑

## Linux 發行版變化

### Alpine Linux
對於較小的映像大小，您可以使用 Alpine Linux：

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
# ... build steps ...

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS final
# Install packages using apk
RUN apk update && apk add --no-cache curl ca-certificates
```

### Ubuntu Chiseled
為了最小化攻擊表面，考慮使用 chiseled 映像：

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0-jammy-chiseled AS final
# Note: Chiseled images have minimal packages, so you may need to use a different base for additional dependencies
```

### Azure Linux (Mariner)
針對 Azure 最佳化的容器：

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0-azurelinux3.0 AS final
# Install packages using tdnf
RUN tdnf update -y && tdnf install -y curl ca-certificates && tdnf clean all
```

## 階段命名說明

- `AS stage-name` 語法為每個階段指定名稱
- 使用 `--from=stage-name` 從先前的階段複製檔案
- 您可以有多個未在最終映像中使用的中間階段
- `final` 階段是成為最終容器映像的階段

## 安全最佳實踐

- 在生產環境中始終以非 root 使用者執行
- 使用特定的映像標籤而不是 `latest`
- 最小化安裝的套件數量
- 保持基礎映像更新
- 使用多階段建置以排除最終映像中的建置依賴項
