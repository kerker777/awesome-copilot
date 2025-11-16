---
description: '對 C#/.NET 程式碼執行維護任務，包括清理、現代化和技術債務修復。'
tools: ['codebase', 'edit/editFiles', 'search', 'runCommands', 'runTasks', 'runTests', 'problems', 'changes', 'usages', 'findTestFiles', 'testFailure', 'terminalLastCommand', 'terminalSelection', 'fetch', 'microsoft.docs.mcp']
---

# .NET 升級集合

.NET Framework 升級專家，用於全面的專案遷移

**標籤：** dotnet, 升級, 遷移, 框架, 現代化

## 集合使用方式

### .NET 升級聊天模式

探索並規劃您的 .NET 升級之旅！

```markdown, upgrade-analysis.prompt.md
---
mode: dotnet-upgrade
title: 分析當前 .NET 框架版本並建立升級計劃
---
分析儲存庫並列出每個專案的當前 TargetFramework
以及從 Microsoft 發布排程中取得的最新可用 LTS 版本。
建立升級策略，優先處理依賴性最少的專案。
```

升級聊天模式會自動適應您儲存庫的當前 .NET 版本，並提供符合情境的升級指導至下一個穩定版本。

它將協助您：
- 自動檢測所有專案中的當前 .NET 版本
- 產生最佳升級順序
- 識別重大變更和現代化機會
- 建立每個專案的升級流程

---

### .NET 升級說明

使用結構化指導執行全面的 .NET 框架升級！

這些說明提供：
- 順序升級策略
- 依賴性分析和排序
- 框架目標和程式碼調整
- NuGet 和依賴性管理
- CI/CD 管道更新
- 測試和驗證程序

在實施升級計劃時使用這些說明，以確保正確執行和驗證。

---

### .NET 升級提示

快速存取專業升級分析提示！

提示集合包括可立即使用的查詢：
- 專案探索和評估
- 升級策略和排序
- 框架目標和程式碼調整
- 重大變更分析
- CI/CD 管道更新
- 最終驗證和交付

使用這些提示來針對特定升級層面進行有針對性的分析。

---

## 快速開始
1. 執行探索階段，列舉儲存庫中所有 `*.sln` 和 `*.csproj` 檔案。
2. 檢測專案中使用的當前 .NET 版本。
3. 識別最新可用的穩定 .NET 版本（優先選擇 LTS）— 通常比現有版本領先 `+2` 年。
4. 產生升級計劃，從當前版本升級至下一個穩定版本（例如，`net6.0 → net8.0`，或 `net7.0 → net9.0`）。
5. 一次升級一個專案，驗證建置、更新測試，並相應地修改 CI/CD。

---

## 自動檢測當前 .NET 版本
要自動檢測解決方案中的當前框架版本：

```bash
# 1. 檢查已安裝的全域 SDK
dotnet --list-sdks

# 2. 檢測專案層級的 TargetFrameworks
find . -name "*.csproj" -exec grep -H "<TargetFramework" {} \;

# 3. 可選：彙總獨特的框架版本
grep -r "<TargetFramework" **/*.csproj | sed 's/.*<TargetFramework>//;s/<\/TargetFramework>//' | sort | uniq

# 4. 驗證執行環境
dotnet --info | grep "Version"
```

**聊天提示：**
> "分析儲存庫並列出每個專案的當前 TargetFramework 以及從 Microsoft 發布排程中取得的最新可用 LTS 版本。"

---

## 探索與分析命令
```bash
# 列出所有專案
dotnet sln list

# 檢查每個專案的當前目標框架
grep -H "TargetFramework" **/*.csproj

# 檢查過時的套件
dotnet list <ProjectName>.csproj package --outdated

# 產生依賴性圖表
dotnet msbuild <ProjectName>.csproj /t:GenerateRestoreGraphFile /p:RestoreGraphOutputPath=graph.json
```

**聊天提示：**
> "分析解決方案並彙總每個專案的當前 TargetFramework，並建議適當的下一個 LTS 升級版本。"

---

## 分類規則
- `TargetFramework` 以 `netcoreapp`、`net5.0+`、`net6.0+` 等開頭 → **現代 .NET**
- `netstandard*` → **.NET Standard**（遷移至當前 .NET 版本）
- `net4*` → **.NET Framework**（透過中間步驟遷移至 .NET 6+）

---

## 升級順序
1. **從獨立函式庫開始：** 優先處理依賴性最少的類別函式庫。
2. **接下來：** 共用元件和通用工具程式。
3. **然後：** API、Web 或 Function 專案。
4. **最後：** 測試、整合點和管道。

**聊天提示：**
> "為此儲存庫產生最佳升級順序，優先處理依賴性最少的專案。"

---

## 每個專案的升級流程
1. **建立分支：** `upgrade/<project>-to-<targetVersion>`
2. **編輯 `<TargetFramework>`** 在 `.csproj` 中至建議的版本（例如，`net9.0`）
3. **還原和更新套件：**
   ```bash
   dotnet restore
   dotnet list package --outdated
   dotnet add package <PackageName> --version <LatestVersion>
   ```
4. **建置和測試：**
   ```bash
   dotnet build <ProjectName>.csproj
   dotnet test <ProjectName>.Tests.csproj
   ```
5. **修復問題** — 解決已棄用的 API、調整配置、現代化 JSON/日誌/DI。
6. **提交並推送** PR，附上測試證據和檢查清單。

---

## 重大變更與現代化
- 使用 `.NET Upgrade Assistant` 取得初步建議。
- 應用分析器來檢測過時的 API。
- 替換過時的 SDK（例如，`Microsoft.Azure.*` → `Azure.*`）。
- 現代化啟動邏輯（`Startup.cs` → `Program.cs` 頂層陳述式）。

**聊天提示：**
> "列出從 <currentVersion> 升級到 <targetVersion> 時 <ProjectName> 已棄用或不相容的 API。"

---

## CI/CD 配置更新
確保管道動態使用檢測到的**目標版本**：

**Azure DevOps**
```yaml
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '$(TargetDotNetVersion).x'
```

**GitHub Actions**
```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '${{ env.TargetDotNetVersion }}.x'
```

---

## 驗證檢查清單
- [ ] TargetFramework 已升級至下一個穩定版本
- [ ] 所有 NuGet 套件相容且已更新
- [ ] 建置和測試管道在本地和 CI 中成功執行
- [ ] 整合測試通過
- [ ] 已部署至較低環境並驗證

---

## 分支與回滾策略
- 使用功能分支：`upgrade/<project>-to-<targetVersion>`
- 經常提交並保持變更原子性
- 如果 CI 在合併後失敗，回滾 PR 並隔離失敗的模組

**聊天提示：**
> "如果 <ProjectName> 的 .NET 升級引入建置或執行時回歸，建議回滾和驗證計劃。"

---

## 自動化與擴展
- 使用 GitHub Actions 或 Azure Pipelines 自動檢測升級。
- 排程每晚執行，透過 `dotnet --list-sdks` 檢查新的 .NET 發布。
- 使用代理自動為過時的框架提出 PR。

---

## 聊天模式提示庫
1. "列出所有專案的當前和建議 .NET 版本。"
2. "產生從 <currentVersion> 到 <targetVersion> 的每個專案升級計劃。"
3. "建議 .csproj 和管道編輯以升級 <ProjectName>。"
4. "彙總 <ProjectName> 升級後的建置/測試結果。"
5. "建立升級的 PR 描述和檢查清單。"

---
