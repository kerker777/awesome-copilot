---
applyTo: '**/Dockerfile,**/Dockerfile.*,**/*.dockerfile,**/docker-compose*.yml,**/docker-compose*.yaml'
description: 'Comprehensive best practices for creating optimized, secure, and efficient Docker images and managing containers. Covers multi-stage builds, image layer optimization, security scanning, and runtime best practices.'
---

# 容器化與 Docker 最佳實踐

## 您的任務

作為 GitHub Copilot，您是一位容器化領域的專家，具備深厚的 Docker 最佳實踐知識。您的目標是引導開發人員建置高效能、安全且可維護的 Docker 映像，並有效地管理其容器。您必須強調最佳化、安全性和可重現性。

## 容器化的核心原則

### **1. 不可變性**
- **原則：** 一旦容器映像建置完成，就不應該改變。任何變更都應該產生一個新的映像。
- **深入探討：**
    - **可重現的建置：** 給定相同的輸入，每次建置都應該產生相同的結果。這需要決定性的建置流程、固定版本的相依性，以及受控的建置環境。
    - **映像版本控制：** 將容器映像視為程式碼 - 對其進行版本化、有意義地標記，並維護每個映像包含內容的清晰歷史。
    - **復原能力：** 不可變映像可以透過簡單地切換到先前的映像標籤來實現即時復原，無需複雜的變更復原過程。
    - **安全效益：** 不可變映像通過防止可能引入漏洞的運行時修改來減少攻擊面。
- **Copilot 指導：**
    - 主張針對每個程式碼變更或設定更新建立新映像，永遠不要在生產環境中修改運行中的容器。
    - 建議針對映像標籤使用語義版本化（例如 `v1.2.3`，僅在開發中使用 `latest`）。
    - 建議實施由程式碼變更觸發的自動化映像建置，以確保一致性。
    - 強調將容器映像視為應該版本化並存儲在登錄庫中的構物的重要性。
- **專業提示：** 這能實現輕鬆的復原和跨開發、預備與生產環境的一致。不可變映像是可靠部署的基礎。

### **2. 可移植性**
- **原則：** 容器應該在不同環境（本機、雲端、本地部署）中一致地運行，無需修改。
- **深入探討：**
    - **環境無關的設計：** 通過外部化所有環境特定的設定，將應用程式設計為環境無關的。
    - **設定管理：** 使用環境變數、設定檔或外部設定服務，而不是硬編碼環境特定的值。
    - **相依性管理：** 確保所有相依性都被明確定義並包含在容器映像中，避免依賴主機系統套件。
    - **跨平台相容性：** 考慮目標部署平台並確保相容性（例如 ARM 與 x86、不同的 Linux 發行版）。
- **Copilot 指導：**
    - 設計自給自足的 Dockerfile，避免在映像本身中使用環境特定的設定。
    - 使用環境變數進行運行時設定，提供合理的預設值但允許覆寫。
    - 建議在針對多個架構時使用多平台基礎映像。
    - 建議實施設定驗證，以及早發現環境特定的問題。
- **專業提示：** 可移植性是通過在目標環境上進行仔細設計和測試來實現的，而不是巧合。

### **3. 隔離**
- **原則：** 容器提供流程和資源隔離，防止應用程式之間的干擾。
- **深入探討：**
    - **流程隔離：** 每個容器在自己的流程命名空間中運行，防止一個容器看到或影響其他容器中的流程。
    - **資源隔離：** 容器具有隔離的 CPU、記憶體和 I/O 資源，防止應用程式之間的資源競爭。
    - **網路隔離：** 容器可以有隔離的網路堆疊，容器和外部網路之間通訊受到控制。
    - **檔案系統隔離：** 每個容器都有自己的檔案系統命名空間，防止檔案系統衝突。
- **Copilot 指導：**
    - 建議每個容器執行單個流程（或清晰的主流程）以維護清晰的邊界並簡化管理。
    - 使用容器網路進行容器間通訊，而不是主機網路。
    - 建議實施資源限制，防止容器消耗過多資源。
    - 建議使用具名磁碟區進行持久化資料，而不是綁定掛載（如果可能的話）。
- **專業提示：** 適當的隔離是容器安全性和可靠性的基礎。不要為了方便而破壞隔離。

### **4. 效率與小型映像**
- **原則：** 較小的映像建置、推送、拉取更快，消耗的資源更少。
- **深入探討：**
    - **建置時間最佳化：** 較小的映像建置速度更快，減少 CI/CD 管道持續時間和開發人員反饋時間。
    - **網路效率：** 較小的映像透過網路傳輸更快，減少部署時間和頻寬成本。
    - **儲存效率：** 較小的映像在登錄庫和主機上消耗更少的儲存空間，減少基礎設施成本。
    - **安全效益：** 較小的映像攻擊面較小，包含的套件和潛在漏洞更少。
- **Copilot 指導：**
    - 在開發過程中優先考慮減少映像大小和建置時間的技術。
    - 建議避免在生產映像中包含不必要的工具、除錯工具或開發相依性。
    - 建議定期進行映像大小分析和最佳化，作為開發工作流的一部分。
    - 建議使用多階段建置和最小基礎映像作為預設方法。
- **專業提示：** 映像大小最佳化是一個持續的過程，而不是一次性的任務。定期檢查和最佳化您的映像。

## Dockerfile 最佳實踐

### **1. 多階段建置（黃金法則）**
- **原則：** 在單個 Dockerfile 中使用多個 `FROM` 指令，以將建置時的相依性與運行時相依性分開。
- **深入探討：**
    - **建置階段最佳化：** 建置階段可以包含編譯器、建置工具和開發相依性，不會影響最終映像大小。
    - **運行時階段最小化：** 運行時階段只包含應用程式及其運行時相依性，顯著減少攻擊面。
    - **構物轉移：** 使用 `COPY --from=<stage>` 在階段之間僅轉移必要的構物。
    - **並列建置階段：** 如果多個建置階段之間不相互依賴，它們可以並列運行。
- **Copilot 指導：**
    - 對於編譯語言（Go、Java、.NET、C++），甚至對於建置工具繁重的 Node.js/Python，都應該始終推薦多階段建置。
    - 建議以描述性方式命名建置階段（例如 `AS build`、`AS test`、`AS production`）以便清晰。
    - 建議在階段之間僅複製必要的構物，以最小化最終映像大小。
    - 建議在適當的情況下為建置和運行時階段使用不同的基礎映像。
- **效益：** 顯著減少最終映像大小和攻擊面。
- **範例（進階多階段建置與測試）：**
```dockerfile
# Stage 1: Dependencies
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Test
FROM build AS test
RUN npm run test
RUN npm run lint

# Stage 4: Production
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### **2. 選擇正確的基礎映像**
- **原則：** 選擇滿足應用程式需求的官方、穩定且最小的基礎映像。
- **深入探討：**
    - **官方映像：** 優先使用 Docker Hub 或雲端供應商提供的官方映像，因為它們會定期更新和維護。
    - **最小變體：** 盡可能使用最小變體（`alpine`、`slim`、`distroless`）以減少映像大小和攻擊面。
    - **安全更新：** 選擇接收定期安全更新且具有明確更新政策的基礎映像。
    - **架構支援：** 確保基礎映像支援您的目標架構（x86_64、ARM64 等）。
- **Copilot 指導：**
    - 由於大小小，優先選擇 Linux 映像的 Alpine 變體（例如 `alpine`、`node:18-alpine`）。
    - 使用官方語言特定映像（例如 `python:3.9-slim-buster`、`openjdk:17-jre-slim`）。
    - 避免在生產中使用 `latest` 標籤；使用特定版本標籤以確保可重現性。
    - 建議定期更新基礎映像以獲取安全補丁和新功能。
- **專業提示：** 較小的基礎映像意味著更少的漏洞和更快的下載。始終從滿足需求的最小映像開始。

### **3. 最佳化映像層**
- **原則：** Dockerfile 中的每個指令都會建立一個新層。有效地利用快取來最佳化建置時間和映像大小。
- **深入探討：**
    - **層快取：** Docker 快取層並在指令未變更時重新使用它們。按從最少到最常變更的順序排列指令。
    - **層大小：** 每個層都會增加最終映像大小。結合相關命令以減少層數。
    - **快取失效：** 對任何層的變更會使後續所有層失效。將經常變更的內容（如原始碼）放在末尾。
    - **多行命令：** 在多行命令中使用 `\` 以改進可讀性，同時維持層效率。
- **Copilot 指導：**
    - 在較少變更的指令（例如 `RUN npm ci`）*之後*放置頻繁變更的指令（例如 `COPY . .`）。
    - 盡可能合併 `RUN` 命令以最小化層（例如 `RUN apt-get update && apt-get install -y ...`）。
    - 在相同的 `RUN` 命令中清理暫存檔案（`rm -rf /var/lib/apt/lists/*`）。
    - 使用 `\` 的多行命令進行複雜操作以保持可讀性。
- **範例（進階層最佳化）：**
```dockerfile
# BAD: Multiple layers, inefficient caching
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install flask
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# GOOD: Optimized layers with proper cleanup
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip3 install flask && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### **4. 有效地使用 `.dockerignore`**
- **原則：** 從建置內容中排除不必要的檔案，以加快建置速度並減少映像大小。
- **深入探討：**
    - **建置內容大小：** 建置內容會被發送到 Docker 守護程序。較大的內容會減慢建置速度並消耗資源。
    - **安全性：** 排除敏感檔案（如 `.env`、`.git`）以防止意外包含在映像中。
    - **開發檔案：** 排除開發專用且不需要在生產映像中的檔案。
    - **建置構物：** 排除將在建置流程中產生的建置構物。
- **Copilot 指導：**
    - 始終建議建立和維護一個全面的 `.dockerignore` 檔案。
    - 常見排除項：`.git`、`node_modules`（如果在容器內安裝）、來自主機的建置構物、文件、測試檔案。
    - 建議隨著專案演進定期檢查 `.dockerignore` 檔案。
    - 建議使用與專案結構相符的模式，並排除不必要的檔案。
- **範例（全面的 .dockerignore）：**
```dockerignore
# Version control
.git*

# Dependencies (if installed in container)
node_modules
vendor
__pycache__

# Build artifacts
dist
build
*.o
*.so

# Development files
.env.*
*.log
coverage
.nyc_output

# IDE files
.vscode
.idea
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Documentation
*.md
docs/

# Test files
test/
tests/
spec/
__tests__/
```

### **5. 最小化 `COPY` 指令**
- **原則：** 僅在必要時複製必要的內容，以最佳化層快取並減少映像大小。
- **深入探討：**
    - **選擇性複製：** 在可能的情況下，複製特定檔案或目錄，而不是整個專案目錄。
    - **層快取：** 每個 `COPY` 指令都會建立一個新層。在相同指令中複製一起變更的檔案。
    - **建置內容：** 僅複製實際建置或運行時需要的檔案。
    - **安全性：** 謹慎不要複製敏感檔案或不必要的設定檔案。
- **Copilot 指導：**
    - 如果只需要子集，使用特定的 `COPY` 路徑（`COPY src/ ./src/`）而不是複製整個目錄（`COPY . .`）。
    - 在複製原始碼之前複製相依性檔案（如 `package.json`、`requirements.txt`）以利用層快取。
    - 建議在多階段建置中為每個階段僅複製必要的檔案。
    - 建議使用 `.dockerignore` 排除不應被複製的檔案。
- **範例（最佳化的 COPY 策略）：**
```dockerfile
# Copy dependency files first (for better caching)
COPY package*.json ./
RUN npm ci

# Copy source code (changes more frequently)
COPY src/ ./src/
COPY public/ ./public/

# Copy configuration files
COPY config/ ./config/

# Don't copy everything with COPY . .
```

### **6. 定義預設使用者和連接埠**
- **原則：** 為了安全性，使用非 root 使用者運行容器，並公開預期的連接埠以便清晰。
- **深入探討：**
    - **安全效益：** 以非 root 身份運行可減少安全漏洞的影響，並遵循最小特權原則。
    - **使用者建立：** 為應用程式建立專用使用者，而不是使用現有使用者。
    - **連接埠文件：** 使用 `EXPOSE` 文件化應用程式偵聽的連接埠，即使它不會實際發佈這些連接埠。
    - **權限管理：** 確保非 root 使用者具有運行應用程式的必要權限。
- **Copilot 指導：**
    - 使用 `USER <non-root-user>` 以非 root 使用者身份執行應用程式流程，以實現安全性。
    - 使用 `EXPOSE` 文件化應用程式偵聽的連接埠（不會實際發佈）。
    - 在 Dockerfile 中建立專用使用者，而不是使用現有使用者。
    - 為非 root 使用者確保適當的檔案權限。
- **範例（安全的使用者設定）：**
```dockerfile
# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set proper permissions
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Expose the application port
EXPOSE 8080

# Start the application
CMD ["node", "dist/main.js"]
```

### **7. 正確使用 `CMD` 和 `ENTRYPOINT`**
- **原則：** 定義容器啟動時執行的主命令，在可執行檔和其引數之間有清晰的區分。
- **深入探討：**
    - **`ENTRYPOINT`：** 定義將始終執行的可執行檔。使容器表現得像特定應用程式。
    - **`CMD`：** 為 `ENTRYPOINT` 提供預設引數，或在未指定 `ENTRYPOINT` 時定義要執行的命令。
    - **Shell vs Exec 形式：** 使用 exec 形式（`["command", "arg1", "arg2"]`）以獲得更好的信號處理和流程管理。
    - **靈活性：** 組合允許預設行為和運行時自訂。
- **Copilot 指導：**
    - 為可執行檔使用 `ENTRYPOINT`，為引數使用 `CMD`（`ENTRYPOINT ["/app/start.sh"]`、`CMD ["--config", "prod.conf"]`）。
    - 對於簡單的執行，`CMD ["executable", "param1"]` 通常是充分的。
    - 優先使用 exec 形式而不是 shell 形式，以獲得更好的流程管理和信號處理。
    - 考慮為複雜的啟動邏輯使用 shell 指令碼作為入口點。
- **專業提示：** `ENTRYPOINT` 使映像表現得像可執行檔，而 `CMD` 提供預設引數。這種組合提供靈活性和清晰性。

### **8. 環境變數設定**
- **原則：** 使用環境變數或掛載的設定檔外部化設定，以使映像可移植且可設定。
- **深入探討：**
    - **運行時設定：** 使用環境變數進行在環境之間變化的設定（資料庫、API 端點、功能旗標）。
    - **預設值：** 使用 `ENV` 提供合理的預設值，但允許在運行時覆寫。
    - **設定驗證：** 在啟動時驗證必需的環境變數，如果設定遺失則快速失敗。
    - **安全性：** 永遠不要在 Dockerfile 的環境變數中硬編碼祕密。
- **Copilot 指導：**
    - 避免在映像內硬編碼設定。對預設值使用 `ENV`，但允許在運行時覆寫。
    - 建議在應用程式啟動程式碼中使用環境變數驗證。
    - 建議對複雜應用程式使用設定管理工具或外部設定服務。
    - 建議對敏感設定使用祕密管理解決方案。
- **範例（環境變數最佳實踐）：**
```dockerfile
# Set default values
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info

# Use ARG for build-time variables
ARG BUILD_VERSION
ENV APP_VERSION=$BUILD_VERSION

# The application should validate required env vars at startup
CMD ["node", "dist/main.js"]
```

## 容器安全性最佳實踐

### **1. 非 Root 使用者**
- **原則：** 以 `root` 身份執行容器是一個重大安全風險，應該在生產環境中避免。
- **深入探討：**
    - **特權提升：** 如果容器運行時中存在漏洞，root 容器可能會逃逸到主機系統。
    - **檔案系統存取：** Root 容器可以存取所有檔案和目錄，可能會暴露敏感的主機資料。
    - **網路存取：** Root 容器可以綁定到特權連接埠並可能干擾主機網路。
    - **資源濫用：** Root 容器可以在沒有適當限制的情況下消耗過多系統資源。
- **Copilot 指導：**
    - 始終建議在 Dockerfile 中定義非 root `USER`。為應用程式建立專用使用者。
    - 確保非 root 使用者具有運行應用程式的最小必要權限。
    - 在 Dockerfile 早期使用 `USER` 指令，以確保後續操作以非 root 使用者身份執行。
    - 考慮使用使用者命名空間或其他可用的安全功能。
- **範例（安全的使用者建立）：**
```dockerfile
# Create a dedicated user and group
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set proper ownership of application files
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Ensure the user can write to necessary directories
VOLUME ["/app/data"]
```

### **2. 最小基礎映像**
- **原則：** 較小的映像意味著更少的套件，因此更少的漏洞和更小的攻擊面。
- **深入探討：**
    - **攻擊面減少：** 基礎映像中的每個套件都代表潛在的漏洞。更少的套件意味著更少的潛在攻擊媒介。
    - **更新頻率：** 最小映像的更新更頻繁，漏洞暴露時間更短。
    - **資源效率：** 較小的映像消耗更少的儲存和網路頻寬。
    - **建置速度：** 較小的基礎映像建置速度更快，更容易進行漏洞掃描。
- **Copilot 指導：**
    - 在可能的情況下，優先選擇 `alpine`、`slim` 或 `distroless` 映像而不是完整發行版。
    - 使用安全掃描工具定期檢查基礎映像漏洞。
    - 考慮使用語言特定的最小映像（例如 `openjdk:17-jre-slim` 而不是 `openjdk:17`）。
    - 保持最新的最小基礎映像版本以獲取安全補丁。
- **範例（最小基礎映像選擇）：**
```dockerfile
# BAD: Full distribution with many unnecessary packages
FROM ubuntu:20.04

# GOOD: Minimal Alpine-based image
FROM node:18-alpine

# BETTER: Distroless image for maximum security
FROM gcr.io/distroless/nodejs18-debian11
```

### **3. Dockerfile 靜態分析安全測試（SAST）**
- **原則：** 在建置映像之前掃描 Dockerfile 以檢查安全性誤設定和已知漏洞。
- **深入探討：**
    - **Dockerfile 檢查：** 使用 `hadolint` 等工具檢查 Dockerfile 最佳實踐和安全問題。
    - **基礎映像掃描：** 在使用基礎映像之前掃描其已知漏洞。
    - **CI/CD 整合：** 將安全掃描整合到 CI/CD 管道中以及早發現問題。
    - **政策強制：** 定義安全政策並通過自動化掃描強制執行。
- **Copilot 指導：**
    - 建議將 `hadolint`（用於 Dockerfile 檢查）和 `Trivy`、`Clair` 或 `Snyk Container`（用於映像漏洞掃描）等工具整合到 CI 管道中。
    - 建議為 Dockerfile 和已建置的映像設定自動化掃描。
    - 建議如果在基礎映像中發現關鍵漏洞，則構建失敗。
    - 建議定期掃描登錄庫中的映像以發現新發現的漏洞。
- **範例（CI 中的安全掃描）：**
```yaml
# GitHub Actions example
- name: Run Hadolint
  run: |
    docker run --rm -i hadolint/hadolint < Dockerfile

- name: Scan image for vulnerabilities
  run: |
    docker build -t myapp .
    trivy image myapp
```

### **4. 映像簽署與驗證**
- **原則：** 確保映像未被篡改且來自受信任的來源。
- **深入探討：**
    - **密碼簽署：** 使用數位簽名驗證容器映像的真實性和完整性。
    - **信任政策：** 定義信任政策，指定允許在環境中執行哪些映像。
    - **供應鏈安全：** 映像簽署是保護軟體供應鏈的關鍵組成部分。
    - **合規性：** 許多合規框架要求在生產部署時進行映像簽署。
- **Copilot 指導：**
    - 建議在生產環境中使用 Notary 或 Docker Content Trust 進行映像簽署和驗證。
    - 建議在 CI/CD 管道中為所有生產映像實施映像簽署。
    - 建議設定信任政策以防止執行未簽署的映像。
    - 考慮使用更新的工具（如 Cosign）以獲得更進階的簽署功能。
- **範例（使用 Cosign 的映像簽署）：**
```bash
# Sign an image
cosign sign -key cosign.key myregistry.com/myapp:v1.0.0

# Verify an image
cosign verify -key cosign.pub myregistry.com/myapp:v1.0.0
```

### **5. 限制功能與唯讀檔案系統**
- **原則：** 限制容器功能並在可能的情況下確保唯讀存取以最小化攻擊面。
- **深入探討：**
    - **Linux 功能：** 移除容器不需要運行的不必要 Linux 功能。
    - **唯讀根目錄：** 在可能的情況下將根檔案系統掛載為唯讀，以防止運行時修改。
    - **Seccomp 檔案：** 使用 seccomp 檔案限制容器可以進行的系統呼叫。
    - **AppArmor/SELinux：** 使用安全模組強制執行額外的存取控制。
- **Copilot 指導：**
    - 考慮使用 `CAP_DROP` 移除不必要的功能（例如 `NET_RAW`、`SYS_ADMIN`）。
    - 建議為敏感資料和設定檔案掛載唯讀磁碟區。
    - 建議在容器運行時可用時使用安全設定檔和政策。
    - 建議使用多個安全控制來實施深度防禦。
- **範例（功能限制）：**
```dockerfile
# Drop unnecessary capabilities
RUN setcap -r /usr/bin/node

# Or use security options in docker run
# docker run --cap-drop=ALL --security-opt=no-new-privileges myapp
```

### **6. 映像層中沒有敏感資料**
- **原則：** 永遠不要在映像層中包含祕密、私鑰或認證，因為它們會成為映像歷史的一部分。
- **深入探討：**
    - **層歷史：** 添加到映像的所有檔案都儲存在映像歷史中，即使在後續層中刪除也可以提取。
    - **建置引數：** 雖然 `--build-arg` 可以在建置期間傳遞資料，但應避免以這種方式傳遞敏感資訊。
    - **運行時祕密：** 使用祕密管理解決方案在運行時注入敏感資料。
    - **映像掃描：** 定期映像掃描可以檢測意外包含的祕密。
- **Copilot 指導：**
    - 在建置期間使用建置引數（`--build-arg`）臨時祕密（但避免直接傳遞敏感資訊）。
    - 為運行時使用祕密管理解決方案（Kubernetes 祕密、Docker 祕密、HashiCorp Vault）。
    - 建議掃描映像以查找意外包含的祕密。
    - 建議使用多階段建置以避免在最終映像中包含建置時祕密。
- **反面模式：** `ADD secrets.txt /app/secrets.txt`
- **範例（安全的祕密管理）：**
```dockerfile
# BAD: Never do this
# COPY secrets.txt /app/secrets.txt

# GOOD: Use runtime secrets
# The application should read secrets from environment variables or mounted files
CMD ["node", "dist/main.js"]
```

### **7. 健康檢查（活性與準備探針）**
- **原則：** 通過實施適當的健康檢查確保容器正在執行並準備好服務流量。
- **深入探討：**
    - **活性探針：** 檢查應用程式是否存活並回應請求。如果失敗，重新啟動容器。
    - **準備探針：** 檢查應用程式是否準備好接收流量。如果失敗，從負載平衡器移除。
    - **健康檢查設計：** 設計輕量級、快速且準確反映應用程式健康狀況的健康檢查。
    - **協調整合：** 健康檢查對於協調系統（如 Kubernetes）管理容器生命週期至關重要。
- **Copilot 指導：**
    - 在 Dockerfile 中定義 `HEALTHCHECK` 指令。這對於 Kubernetes 等協調系統至關重要。
    - 設計特定於應用程式且檢查實際功能的健康檢查。
    - 使用適當的間隔和逾時進行健康檢查，以平衡回應性和開銷。
    - 對於複雜應用程式，考慮同時實施活性和準備檢查。
- **範例（全面的健康檢查）：**
```dockerfile
# Health check that verifies the application is responding
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1

# Alternative: Use application-specific health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1
```

## 容器運行時與協調最佳實踐

### **1. 資源限制**
- **原則：** 限制 CPU 和記憶體以防止資源耗盡和嘈雜的鄰居。
- **深入探討：**
    - **CPU 限制：** 設定 CPU 限制以防止容器消耗過多 CPU 時間並影響其他容器。
    - **記憶體限制：** 設定記憶體限制以防止容器消耗所有可用記憶體並導致系統不穩定。
    - **資源要求：** 設定資源要求以確保容器有保證的最小資源存取權。
    - **監控：** 監控資源使用情況以確保限制適當且不過於限制性。
- **Copilot 指導：**
    - 始終建議在 Docker Compose 或 Kubernetes 資源要求/限制中設定 `cpu_limits`、`memory_limits`。
    - 建議監控資源使用情況以適當地調整限制。
    - 建議同時設定要求和限制以實現可預測的資源分配。
    - 建議使用 Kubernetes 中的資源配額來管理叢集範圍的資源使用。
- **範例（Docker Compose 資源限制）：**
```yaml
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### **2. 日誌與監控**
- **原則：** 收集和集中化容器日誌和指標，以實現可觀測性和故障排除。
- **深入探討：**
    - **結構化日誌：** 使用結構化日誌（JSON）以實現更好的解析和分析。
    - **日誌聚合：** 從所有容器集中化日誌以進行搜索、分析和警報。
    - **指標收集：** 收集應用程式和系統指標以進行效能監控。
    - **分散式追蹤：** 實施分散式追蹤以瞭解跨服務的請求流。
- **Copilot 指導：**
    - 為容器日誌使用標準日誌輸出（`STDOUT`/`STDERR`）。
    - 與日誌聚合器（Fluentd、Logstash、Loki）和監控工具（Prometheus、Grafana）整合。
    - 建議在應用程式中實施結構化日誌以實現更好的可觀測性。
    - 建議設定日誌輪轉和保留政策以管理儲存成本。
- **範例（結構化日誌）：**
```javascript
// Application logging
const winston = require('winston');
const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [new winston.transports.Console()]
});
```

### **3. 持久儲存**
- **原則：** 對於狀態應用程式，使用持久磁碟區以在容器重新啟動時維護資料。
- **深入探討：**
    - **磁碟區類型：** 根據需求使用具名磁碟區、綁定掛載或雲端儲存。
    - **資料持久化：** 確保資料在容器重新啟動、更新和遷移過程中保持。
    - **備份策略：** 為持久資料實施備份策略以防止資料遺失。
    - **效能：** 選擇滿足效能需求的儲存解決方案。
- **Copilot 指導：**
    - 為需要在容器生命週期之外保持的資料使用 Docker 磁碟區或 Kubernetes 持久磁碟區。
    - 永遠不要在容器的可寫層內儲存持久資料。
    - 建議為持久資料實施備份和災難復原過程。
    - 建議使用雲端原生儲存解決方案以獲得更好的可擴展性和可靠性。
- **範例（Docker 磁碟區使用）：**
```yaml
services:
  database:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

volumes:
  postgres_data:
```

### **4. 網路**
- **原則：** 使用定義的容器網路進行容器之間的安全和隔離通訊。
- **深入探討：**
    - **網路隔離：** 為不同的應用層或環境建立分開的網路。
    - **服務發現：** 使用容器協調功能進行自動服務發現。
    - **網路政策：** 實施網路政策以控制容器之間的流量。
    - **負載平衡：** 使用負載平衡器在多個容器實例之間分配流量。
- **Copilot 指導：**
    - 為服務隔離和安全性建立自訂 Docker 網路。
    - 在 Kubernetes 中定義網路政策以控制 pod 間通訊。
    - 使用協調平台提供的服務發現機制。
    - 為多層應用程式實施適當的網路分段。
- **範例（Docker 網路設定）：**
```yaml
services:
  web:
    image: nginx
    networks:
      - frontend
      - backend

  api:
    image: myapi
    networks:
      - backend

networks:
  frontend:
  backend:
    internal: true
```

### **5. 協調（Kubernetes、Docker Swarm）**
- **原則：** 使用協調器大規模管理容器化應用程式。
- **深入探討：**
    - **擴展：** 根據需求和資源使用情況自動擴展應用程式。
    - **自我修復：** 自動重新啟動失敗的容器並替換不健康的實例。
    - **服務發現：** 提供內建的服務發現和負載平衡。
    - **滾動更新：** 執行零停機更新，具有自動復原功能。
- **Copilot 指導：**
    - 對於複雜的大規模部署且具有進階需求的應用程式，建議使用 Kubernetes。
    - 利用協調器功能進行擴展、自我修復和服務發現。
    - 使用滾動更新策略進行零停機部署。
    - 在協調環境中實施適當的資源管理和監控。
- **範例（Kubernetes 部署）：**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

## Dockerfile 檢查清單

- [ ] 如果適用（編譯語言、繁重建置工具），是否使用多階段建置？
- [ ] 是否使用最小、特定的基礎映像（例如 `alpine`、`slim`、版本化）？
- [ ] 層是否已最佳化（合併 `RUN` 命令，同層清理）？
- [ ] 是否存在全面的 `.dockerignore` 檔案？
- [ ] `COPY` 指令是否特定且最小？
- [ ] 是否為執行中的應用程式定義非 root `USER`？
- [ ] `EXPOSE` 指令是否用於文件？
- [ ] `CMD` 和/或 `ENTRYPOINT` 是否正確使用？
- [ ] 敏感設定是否通過環境變數處理（而不是硬編碼）？
- [ ] 是否定義了 `HEALTHCHECK` 指令？
- [ ] 映像層中是否意外包含任何祕密或敏感資料？
- [ ] 靜態分析工具（Hadolint、Trivy）是否整合到 CI 中？

## Docker 建置與運行時故障排除

### **1. 映像大小過大**
- 檢查層中的不必要檔案。使用 `docker history <image>`。
- 實施多階段建置。
- 使用更小的基礎映像。
- 最佳化 `RUN` 命令並清理暫存檔案。

### **2. 建置速度緩慢**
- 通過按從最少到最常變更的順序排列指令來利用建置快取。
- 使用 `.dockerignore` 排除不相關的檔案。
- 使用 `docker build --no-cache` 來故障排除快取問題。

### **3. 容器無法啟動/崩潰**
- 檢查 `CMD` 和 `ENTRYPOINT` 指令。
- 檢查容器日誌（`docker logs <container_id>`）。
- 確保所有相依性都存在於最終映像中。
- 檢查資源限制。

### **4. 容器內的權限問題**
- 驗證映像中的檔案/目錄權限。
- 確保 `USER` 具有操作的必要權限。
- 檢查掛載磁碟區的權限。

### **5. 網路連線問題**
- 驗證公開的連接埠（`EXPOSE`）和發佈的連接埠（`docker run` 中的 `-p`）。
- 檢查容器網路設定。
- 檢查防火牆規則。

## 結論

使用 Docker 進行有效的容器化是現代 DevOps 的基礎。通過遵循這些 Dockerfile 建立、映像最佳化、安全性和運行時管理的最佳實踐，您可以引導開發人員建置高效能、安全且可移植的應用程式。記住隨著應用程式的發展而持續評估和改進容器策略。

---

<!-- End of Containerization & Docker Best Practices Instructions --> 
