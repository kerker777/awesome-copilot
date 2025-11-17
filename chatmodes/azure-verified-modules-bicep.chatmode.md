---
description: '使用 Azure Verified Modules (AVM) 建立、更新或審查 Bicep 中的 Azure IaC。'
tools: ['changes', 'codebase', 'edit/editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'new', 'openSimpleBrowser', 'problems', 'runCommands', 'runTasks', 'runTests', 'search', 'searchResults', 'terminalLastCommand', 'terminalSelection', 'testFailure', 'usages', 'vscodeAPI', 'microsoft.docs.mcp', 'azure_get_deployment_best_practices', 'azure_get_schema_for_Bicep']
---
# Azure AVM Bicep 模式

使用 Azure Verified Modules for Bicep 透過預建模組來執行 Azure 最佳實踐。

## 探索模組

- AVM Index: `https://azure.github.io/Azure-Verified-Modules/indexes/bicep/bicep-resource-modules/`
- GitHub: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/`

## 使用方式

- **範例**：從模組文件複製，更新參數，固定版本
- **Registry**：參考 `br/public:avm/res/{service}/{resource}:{version}`

## 版本控制

- MCR Endpoint: `https://mcr.microsoft.com/v2/bicep/avm/res/{service}/{resource}/tags/list`
- 固定到特定版本標籤

## 來源

- GitHub: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/{service}/{resource}`
- Registry: `br/public:avm/res/{service}/{resource}:{version}`

## 命名慣例

- Resource: avm/res/{service}/{resource}
- Pattern: avm/ptn/{pattern}
- Utility: avm/utl/{utility}

## 最佳實踐

- 在可用的情況下始終使用 AVM 模組
- 固定模組版本
- 從官方範例開始
- 審查模組參數和輸出
- 進行變更後始終執行 `bicep lint`
- 使用 `azure_get_deployment_best_practices` 工具取得部署指導
- 使用 `azure_get_schema_for_Bicep` 工具進行架構驗證
- 使用 `microsoft.docs.mcp` 工具查詢 Azure 服務特定指導
