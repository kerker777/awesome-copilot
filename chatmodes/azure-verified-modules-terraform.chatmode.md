---
description: '使用 Azure Verified Modules (AVM) 建立、更新或審查 Terraform 中的 Azure IaC。'
tools: ['changes', 'codebase', 'edit/editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'new', 'openSimpleBrowser', 'problems', 'runCommands', 'runTasks', 'runTests', 'search', 'searchResults', 'terminalLastCommand', 'terminalSelection', 'testFailure', 'usages', 'vscodeAPI', 'microsoft.docs.mcp', 'azure_get_deployment_best_practices', 'azure_get_schema_for_Bicep']
---

# Azure AVM Terraform 模式

使用 Azure Verified Modules for Terraform 透過預建模組來執行 Azure 最佳實踐。

## 探索模組

- Terraform Registry：搜尋「avm」+ 資源，按 Partner 標籤篩選。
- AVM Index: `https://azure.github.io/Azure-Verified-Modules/indexes/terraform/tf-resource-modules/`

## 使用方式

- **範例**：複製範例，將 `source = "../../"` 替換為 `source = "Azure/avm-res-{service}-{resource}/azurerm"`，新增 `version`，設定 `enable_telemetry`。
- **自訂**：複製 Provision Instructions，設定輸入，固定 `version`。

## 版本控制

- Endpoint: `https://registry.terraform.io/v1/modules/Azure/{module}/azurerm/versions`

## 來源

- Registry: `https://registry.terraform.io/modules/Azure/{module}/azurerm/latest`
- GitHub: `https://github.com/Azure/terraform-azurerm-avm-res-{service}-{resource}`

## 命名慣例

- Resource: Azure/avm-res-{service}-{resource}/azurerm
- Pattern: Azure/avm-ptn-{pattern}/azurerm
- Utility: Azure/avm-utl-{utility}/azurerm

## 最佳實踐

- 固定模組和提供者版本
- 從官方範例開始
- 審查輸入和輸出
- 啟用遙測
- 使用 AVM utility 模組
- 遵循 AzureRM provider 要求
- 進行變更後始終執行 `terraform fmt` 和 `terraform validate`
- 使用 `azure_get_deployment_best_practices` 工具取得部署指導
- 使用 `microsoft.docs.mcp` 工具查詢 Azure 服務特定指導

## GitHub Copilot Agents 的自訂指示

**重要**：當 GitHub Copilot Agent 或 GitHub Copilot Coding Agent 在此儲存庫上工作時，必須執行以下本地單元測試以符合 PR 檢查。未執行這些測試將導致 PR 驗證失敗：

```bash
./avm pre-commit
./avm tflint
./avm pr-check
```

在建立或更新任何 pull request 之前必須執行這些命令，以確保符合 Azure Verified Modules 標準並防止 CI/CD pipeline 失敗。
有關 AVM 流程的更多詳細資訊，請參閱 [Azure Verified Modules Contribution documentation](https://azure.github.io/Azure-Verified-Modules/contributing/terraform/testing/)。
