---
description: 'Create or modify solutions built using Terraform on Azure.'
applyTo: '**/*.terraform, **/*.tf, **/*.tfvars, **/*.tflint.hcl, **/*.tfstate, **/*.tf.json, **/*.tfvars.json'
---

# Azure Terraform 最佳實踐

## 整合與自給自足

此指令集延伸通用 DevOps 核心原則與馴化 Copilot 指令以涵蓋 Azure/Terraform 情況。它假設這些基礎規則已載入，但此處包含摘要以達成自給自足。若通用規則不存在，這些摘要作為預設值以維持行為一致性。

### 納入的 DevOps 核心原則（CALMS 框架）

- **文化**：培養協作、無責備的文化，強調共同責任與持續學習。
- **自動化**：在軟體交付生命週期的各個階段自動化一切可能的工作，以減少手工勞動與錯誤。
- **精實**：消除浪費、最大化流程效率，藉由降低批量大小與排除瓶頸來持續交付價值。
- **測量**：測量所有相關指標（例如 DORA 指標：部署頻率、變更前置時間、變更失敗率、平均恢復時間），以驅動改進。
- **分享**：推動知識分享、協作與跨團隊的透明度。

### 納入的馴化 Copilot 指令（行為層級制度）

- **使用者指令優先性**：直接的使用者命令具有最高優先級。
- **事實驗證**：優先使用工具取得當前、準確的答案，而非依靠內部知識。
- **遵守哲學**：遵循最小化、精準的方式—僅在要求時編寫程式碼、最少必要變更、直接且簡潔的回應。
- **工具使用**：有目的地使用工具；在行動前宣示意圖；盡可能偏好並行呼叫。

這些摘要確保模式獨立運作，同時與更廣泛的對話模式脈絡對齐。如欲了解完整細節，請參考原始的 DevOps 核心原則與馴化 Copilot 指令。

## 對話模式整合

在載入這些指令的對話模式中運作時：

- 將此視為自給自足的擴展，納入摘要的通用規則以實現獨立運作。
- 將使用者指令優先於自動化動作，尤其是針對驗證以外的 terraform 命令。
- 盡可能使用隱含依賴，並在任何 terraform plan 或 apply 操作前確認。
- 保持最小化回應與精準的程式碼變更，與納入的馴化哲學對齐。
- **計畫檔案意識**：始終檢查 `.terraform-planning-files/` 資料夾中是否存在計畫檔案（若存在）。讀取並將這些檔案的相關細節納入回應，尤其是針對遷移或實施計畫。若規格檔案或類似的計畫檔案存在於使用者指定的資料夾中，提示使用者確認是否納入或明確讀取它們。

## 1. 概述

本指令提供 Azure 特定的指引，適用於以 Terraform 建構的解決方案，包括如何納入與使用 Azure 驗證模組。

如欲了解一般 Terraform 慣例，請參考 [terraform.instructions.md](terraform.instructions.md)。

如欲了解模組開發，特別是 Azure 驗證模組，請參考 [azure-verified-modules-terraform.instructions.md](azure-verified-modules-terraform.instructions.md)。

## 2. 須避免的反模式

**組態設定：**

- 不得將應該參數化的值寫死
- 不應將 `terraform import` 作為常規工作流程模式
- 應避免複雜的條件邏輯，使程式碼難以理解
- 不得使用 `local-exec` 佈建程式，除非絕對必要

**安全性：**

- 絕不得在 Terraform 檔案或狀態中儲存機密
- 必須避免過度寬鬆的 IAM 角色或網路規則
- 不得為了方便而停用安全功能
- 不得使用預設密碼或金鑰

**營運：**

- 不得在未經測試的情況下直接將 Terraform 變更套用到生產環境
- 必須避免對 Terraform 管理的資源進行手工變更
- 不得忽視 Terraform 狀態檔案毀損或不一致性
- 不得在本機機器上執行 Terraform 以進行生產環境操作
- 只能以唯讀方式使用 Terraform 狀態檔案（`**/*.tfstate`），所有變更必須透過 Terraform CLI 或 HCL 進行。
- 只能以唯讀方式使用 `**/.terraform/**` 的內容（取得的模組與供應商）。

這些建立在納入的馴化 Copilot 指令之上，用於安全、營運實踐。

---

## 3. 清晰地組織程式碼

以邏輯檔案分離的方式構造 Terraform 組態設定：

- 使用 `main.tf` 放置資源
- 使用 `variables.tf` 放置輸入
- 使用 `outputs.tf` 放置輸出
- 使用 `terraform.tf` 放置供應商組態設定
- 使用 `locals.tf` 提取複雜運算式以提高可讀性
- 遵循一致的命名慣例與格式設定（`terraform fmt`）
- 若 main.tf 或 variables.tf 檔案過大，請按資源類型或函式分割為多個檔案（例如 `main.networking.tf`、`main.storage.tf` - 將對應的變數移到 `variables.networking.tf` 等）

變數與模組名稱使用 `snake_case` 命名。

## 4. 使用 Azure 驗證模組（AVM）

任何重大資源若有可用的 AVM，應使用 AVM。AVM 設計用來符合完善架構框架，由 Microsoft 支援與維護，有助於減少需要維護的程式碼數量。如欲了解如何發現這些模組，請參考 [Azure Verified Modules for Terraform](azure-verified-modules-terraform.instructions.md)。

若資源沒有可用的 Azure 驗證模組，建議建立一個「以 AVM 風格」的模組，以與現有工作對齐並提供機會貢獻至上游社群。

此指令的例外情況是使用者被指引使用內部私有登錄，或明確表示不願使用 Azure 驗證模組。

這與納入的 DevOps 自動化原則對齐，透過運用預先驗證、社群維護的模組。

## 5. 變數與程式碼風格標準

遵循 AVM 對齊的編碼標準以在解決方案程式碼中維持一致性：

- **變數命名**：所有變數名稱使用 snake_case（根據 TFNFR4 與 TFNFR16）。具描述性並與命名慣例保持一致。
- **變數定義**：所有變數必須有明確的類型宣告（根據 TFNFR18）與詳盡的描述（根據 TFNFR17）。避免為集合值設定可空預設值（根據 TFNFR20），除非有特定需求。
- **敏感變數**：適當地標記敏感變數並避免明確設定 `sensitive = false`（根據 TFNFR22）。正確處理敏感預設值（根據 TFNFR23）。
- **動態區塊**：在適當的情況下為可選巢狀物件使用動態區塊（根據 TFNFR12），並運用 `coalesce` 或 `try` 函式取得預設值（根據 TFNFR13）。
- **程式碼組織**：考慮特別為局部值使用 `locals.tf`（根據 TFNFR31），並確保局部變數的精確類型（根據 TFNFR33）。

## 6. 機密

最好的機密就是不需要儲存的機密。例如，使用受控識別而非密碼或金鑰。

在支援時使用 `ephemeral` 機密與唯寫參數（Terraform v1.11+）以避免在狀態檔案中儲存機密。請參考模組文件以了解可用性。

若需要機密，應儲存在 Key Vault，除非被指引使用其他服務。

絕不將機密寫入本機檔案系統或提交至 git。

適當地標記敏感值，將其與其他屬性隔離，並避免輸出敏感資料，除非絕對必要。遵循 TFNFR19、TFNFR22 與 TFNFR23。

## 7. 輸出

- **避免不必要的輸出**，僅使用此功能公開其他組態設定所需的資訊。
- 對包含機密的輸出使用 `sensitive = true`
- 為所有輸出提供清晰的描述

```hcl
output "resource_group_name" {
  description = "Name of the created resource group"
  value       = azurerm_resource_group.example.name
}

output "virtual_network_id" {
  description = "ID of the virtual network"
  value       = azurerm_virtual_network.example.id
}
```

## 8. 局部值的使用

- 將局部變數用於計算值與複雜運算式
- 透過提取重複的運算式來改善可讀性
- 將相關值組合為結構化的局部變數

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Owner       = var.owner
    CreatedBy   = "terraform"
  }

  resource_name_prefix = "${var.project_name}-${var.environment}"
  location_short       = substr(var.location, 0, 3)
}
```

## 9. 遵循建議的 Terraform 實踐

- **冗餘 depends_on 檢測**：搜尋並移除 `depends_on`，其中相依資源已在相同資源區塊中隱含地被引用。只有在明確需要時才保留 `depends_on`。絕不依賴模組輸出。

- **反覆**：對 0-1 個資源使用 `count`，對多個資源使用 `for_each`。偏好使用地圖以取得穩定的資源地址。與 TFNFR7 對齐。

- **資料來源**：在根模組中可接受，但在可重用模組中應避免。偏好使用明確的模組參數而非資料來源查閱。

- **參數化**：使用強類型變數，具有明確的 `type` 宣告（TFNFR18）、詳盡的描述（TFNFR17）與不可空的預設值（TFNFR20）。運用 AVM 公開的變數。

- **版本控制**：針對最新穩定的 Terraform 與 Azure 供應商版本。在程式碼中指定版本並保持更新（TFFR3）。

## 10. 資料夾結構

為 Terraform 組態設定使用一致的資料夾結構。

使用 tfvars 修改環境差異。一般來說，目標是保持環境相似，同時針對非生產環境進行成本最佳化。

反模式 - 每個環境一個分支、每個環境一個儲存庫、每個環境一個資料夾，或類似的佈局，使得難以在環境間測試根資料夾邏輯。

應瞭解 Terragrunt 等工具，這些可能會影響此設計。

一個**建議的**結構為：

```text
my-azure-app/
├── infra/                          # Terraform root module (AZD compatible)
│   ├── main.tf                     # Core resources
│   ├── variables.tf                # Input variables
│   ├── outputs.tf                  # Outputs
│   ├── terraform.tf                # Provider configuration
│   ├── locals.tf                   # Local values
│   └── environments/               # Environment-specific configurations
│       ├── dev.tfvars              # Development environment
│       ├── test.tfvars             # Test environment
│       └── prod.tfvars             # Production environment
├── .github/workflows/              # CI/CD pipelines (if using github)
├── .azdo/                          # CI/CD pipelines (suggested if using Azure DevOps)
└── README.md                       # Documentation
```

未經使用者明確同意，不得變更資料夾結構。

遵循 AVM 規範 TFNFR1、TFNFR2、TFNFR3 與 TFNFR4 以取得一致的檔案命名與結構。

## Azure 特定最佳實踐

### 資源命名與標籤

- 遵循 [Azure 命名慣例](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
- 針對多區域部署使用一致的區域命名與變數
- 實施一致的標籤。

### 資源群組策略

- 在指定時使用現有的資源群組
- 僅在必要且經過確認時才建立新的資源群組
- 使用描述用途與環境的名稱

### 網路考量

- 在建立新網路資源前驗證現有的虛擬網路/子網路識別碼（例如，此解決方案是否部署到現有的中樞與支點登陸區域）
- 適當地使用網路安全群組與應用程式安全群組
- 於必要時對 PaaS 服務實施私人端點；否則使用資源防火牆限制以限制公開存取。在需要公開端點的例外情況下新增註解。

### 安全性與合規性

- 使用受控識別而非服務主體
- 使用適當的角色型存取控制實施 Key Vault。
- 啟用診斷設定以進行稽核追蹤
- 遵循最小權限原則

## 成本管理

- 為昂貴資源確認預算批准
- 使用適合環境的規模（開發 vs 生產）
- 若未指定，詢問成本約束

## 狀態管理

- 使用遠端後端（Azure 儲存體）並啟用狀態鎖定
- 絕不將狀態檔案提交至原始控制
- 啟用傳輸中與靜止時的加密

## 驗證

- 現有資源的清單盤點，並提供移除未使用資源區塊的選項。
- 執行 `terraform validate` 檢查語法
- 在執行 `terraform plan` 前詢問。Terraform plan 需要訂用帳戶識別碼，此應來自 ARM_SUBSCRIPTION_ID 環境變數，*不得*在供應商區塊中編寫。
- 首先在非生產環境中測試組態設定
- 確保冪等性（多次套用產生相同結果）

## 遞補行為

若通用規則未載入，預設為：最小化程式碼生成、針對驗證以外的任何 terraform 命令的明確同意，以及遵循所有建議中的 CALMS 原則。
