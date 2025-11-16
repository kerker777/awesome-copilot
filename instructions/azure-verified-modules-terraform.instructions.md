---
description: 'Azure 驗證模組 (AVM) 和 Terraform'
applyTo: '**/*.terraform, **/*.tf, **/*.tfvars, **/*.tfstate, **/*.tflint.hcl, **/*.tf.json, **/*.tfvars.json'
---

# Azure 驗證模組 (AVM) Terraform

## 概觀

Azure 驗證模組 (AVM) 是預先建立、測試並驗證的 Terraform 和 Bicep 模組,遵循 Azure 最佳實踐。使用這些模組可以自信地建立、更新或檢閱 Azure 基礎架構即程式碼 (IaC)。

## GitHub Copilot 代理程式的自訂指令

**重要**: 當 GitHub Copilot 代理程式或 GitHub Copilot 編碼代理程式在此儲存庫上工作時,必須執行以下本機單元測試以符合 PR 檢查要求。未執行這些測試將導致 PR 驗證失敗:

```bash
./avm pre-commit
./avm tflint
./avm pr-check
```

在建立或更新任何 pull request 之前,必須執行這些命令,以確保符合 Azure 驗證模組標準並防止 CI/CD 管道失敗。
有關 AVM 流程的更多詳細資訊,請參閱 [Azure 驗證模組貢獻文件](https://azure.github.io/Azure-Verified-Modules/contributing/terraform/testing/)。

**未執行這些測試將導致 PR 驗證失敗並阻止成功合併。**

## 模組探索

### Terraform Registry

- 搜尋 "avm" + 資源名稱
- 使用 "Partner" 標籤篩選以找到官方 AVM 模組
- 範例: 搜尋 "avm storage account" → 按 Partner 篩選

### 官方 AVM 索引

> **注意:** 以下連結始終指向主分支上 CSV 檔案的最新版本。如預期,這意味著檔案可能會隨時間變化。如果需要特定時間點的版本,請考慮在 URL 中使用特定的發行標籤。

- **Terraform 資源模組**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformResourceModules.csv`
- **Terraform 模式模組**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformPatternModules.csv`
- **Terraform 工具模組**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformUtilityModules.csv`


## Terraform 模組使用

### 從範例開始

1. 從模組文件中複製範例程式碼
2. 將 `source = "../../"` 替換為 `source = "Azure/avm-res-{service}-{resource}/azurerm"`
3. 新增 `version = "~> 1.0"` (使用最新可用版本)
4. 設定 `enable_telemetry = true`

### 從頭開始

1. 從模組文件中複製佈建指令
2. 設定必要和選用輸入
3. 固定模組版本
4. 啟用遙測

### 使用範例

```hcl
module "storage_account" {
  source  = "Azure/avm-res-storage-storageaccount/azurerm"
  version = "~> 0.1"

  enable_telemetry    = true
  location            = "East US"
  name                = "mystorageaccount"
  resource_group_name = "my-rg"

  # 其他設定...
}
```

## 命名慣例

### 模組類型

- **資源模組**: `Azure/avm-res-{service}-{resource}/azurerm`
  - 範例: `Azure/avm-res-storage-storageaccount/azurerm`
- **模式模組**: `Azure/avm-ptn-{pattern}/azurerm`
  - 範例: `Azure/avm-ptn-aks-enterprise/azurerm`
- **工具模組**: `Azure/avm-utl-{utility}/azurerm`
  - 範例: `Azure/avm-utl-regions/azurerm`

### 服務命名

- 對服務和資源使用 kebab-case
- 遵循 Azure 服務名稱 (例如, `storage-storageaccount`, `network-virtualnetwork`)

## 版本管理

### 檢查可用版本

- 端點: `https://registry.terraform.io/v1/modules/Azure/{module}/azurerm/versions`
- 範例: `https://registry.terraform.io/v1/modules/Azure/avm-res-storage-storageaccount/azurerm/versions`

### 版本固定最佳實踐

- 使用悲觀版本約束: `version = "~> 1.0"`
- 生產環境固定到特定版本: `version = "1.2.3"`
- 升級前務必檢閱變更日誌

## 模組來源

### Terraform Registry

- **URL 模式**: `https://registry.terraform.io/modules/Azure/{module}/azurerm/latest`
- **範例**: `https://registry.terraform.io/modules/Azure/avm-res-storage-storageaccount/azurerm/latest`

### GitHub 儲存庫

- **URL 模式**: `https://github.com/Azure/terraform-azurerm-avm-{type}-{service}-{resource}`
- **範例**:
  - 資源: `https://github.com/Azure/terraform-azurerm-avm-res-storage-storageaccount`
  - 模式: `https://github.com/Azure/terraform-azurerm-avm-ptn-aks-enterprise`

## 開發最佳實踐

### 模組使用

- ✅ **務必**固定模組和提供者版本
- ✅ **從**模組文件中的官方範例開始
- ✅ **在實作前檢閱**所有輸入和輸出
- ✅ **啟用**遙測: `enable_telemetry = true`
- ✅ **使用** AVM 工具模組處理常見模式
- ✅ **遵循** AzureRM 提供者需求和約束

### 程式碼品質

- ✅ **務必**在變更後執行 `terraform fmt`
- ✅ **務必**在變更後執行 `terraform validate`
- ✅ **使用**有意義的變數名稱和描述
- ✅ **新增**適當的標籤和中繼資料
- ✅ **記錄**複雜的設定

### 驗證需求

在建立或更新任何 pull request 之前:

```bash
# 格式化程式碼
terraform fmt -recursive

# 驗證語法
terraform validate

# AVM 特定驗證 (必要)
./avm pre-commit
./avm tflint
./avm pr-check
```

## 工具整合

### 使用可用工具

- **部署指南**: 使用 `azure_get_deployment_best_practices` 工具
- **服務文件**: 使用 `microsoft.docs.mcp` 工具獲取 Azure 服務特定指南
- **結構描述資訊**: 使用 `azure_get_schema_for_Bicep` 獲取 Bicep 資源資訊

### GitHub Copilot 整合

在處理 AVM 儲存庫時:

1. 在建立新資源之前務必檢查現有模組
2. 使用官方範例作為起點
3. 提交前執行所有驗證測試
4. 記錄任何自訂或與範例的偏離

## 常見模式

### 資源群組模組

```hcl
module "resource_group" {
  source  = "Azure/avm-res-resources-resourcegroup/azurerm"
  version = "~> 0.1"

  enable_telemetry = true
  location         = var.location
  name            = var.resource_group_name
}
```

### 虛擬網路模組

```hcl
module "virtual_network" {
  source  = "Azure/avm-res-network-virtualnetwork/azurerm"
  version = "~> 0.1"

  enable_telemetry    = true
  location            = module.resource_group.location
  name                = var.vnet_name
  resource_group_name = module.resource_group.name
  address_space       = ["10.0.0.0/16"]
}
```

## 疑難排解

### 常見問題

1. **版本衝突**: 務必檢查模組和提供者版本之間的相容性
2. **缺少相依性**: 確保首先建立所有必要資源
3. **驗證失敗**: 提交前執行 AVM 驗證工具
4. **文件**: 務必參考最新的模組文件

### 支援資源

- **AVM 文件**: `https://azure.github.io/Azure-Verified-Modules/`
- **GitHub Issues**: 在特定模組的 GitHub 儲存庫中回報問題
- **社群**: Azure Terraform Provider GitHub 討論區

## 合規檢查清單

提交任何 AVM 相關程式碼前:

- [ ] 模組版本已固定
- [ ] 遙測已啟用
- [ ] 程式碼已格式化 (`terraform fmt`)
- [ ] 程式碼已驗證 (`terraform validate`)
- [ ] AVM pre-commit 檢查通過 (`./avm pre-commit`)
- [ ] TFLint 檢查通過 (`./avm tflint`)
- [ ] AVM PR 檢查通過 (`./avm pr-check`)
- [ ] 文件已更新
- [ ] 範例已測試且正常運作
