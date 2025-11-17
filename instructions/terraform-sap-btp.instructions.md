---
description: 'Terraform conventions and guidelines for SAP Business Technology Platform (SAP BTP).'
applyTo: '**/*.tf, **/*.tfvars, **/*.tflint.hcl, **/*.tf.json, **/*.tfvars.json'
---

# SAP BTP 上的 Terraform – 最佳實踐與約定

## 核心原則

讓 Terraform 程式碼保持最小化、模組化、可重複使用、安全且可稽核。
始終使用版本控制管理 Terraform HCL，絕不要版本控制已生成的狀態。

## 安全性

必須：
- 使用最新穩定的 Terraform CLI 和提供者版本；主動升級以獲得安全修補程式。
- 不要提交祕密、認證、憑證、Terraform 狀態或計畫輸出成品。
- 將所有祕密變數和輸出標記為 `sensitive = true`。
- 偏好短期/僅寫入提供者驗證（Terraform >= 1.11），使祕密永不保存在狀態中。
- 最小化敏感輸出；只發出下游自動化真正需要的內容。
- 在 CI 中持續使用 `tfsec`、`trivy`、`checkov` 進行掃描（至少選擇其中一個）。
- 定期審查提供者認證、輪換金鑰，並在支援的地方啟用 MFA。

## 模組性

為了清晰度和速度而結構化：
- 按邏輯域分割（例如，權利、服務執行個體）– 不要按環境分割。
- 僅對可重複使用的多資源模式使用模組；避免使用單一資源包裝模組。
- 保持模組層級淺層；避免深層巢狀和循環相依性。
- 僅透過 `outputs` 公開必要的跨模組資料（必要時標記為敏感）。

## 可維護性

追求明確 > 隱含。
- 評論為什麼，而不是什麼；避免重述明顯的資源屬性。
- 使用參數化（變數）而不是硬編碼；僅在合理的情況下提供預設值。
- 對外部現有基礎設施優先使用資料源；絕不對同一根中剛建立的資源使用 – 使用輸出。
- 避免在通用可重複使用的模組中使用資料源；改為要求輸入。
- 移除未使用/緩慢的資料源；它們會降低計畫時間。
- 使用 `locals` 來集中化邏輯，以處理派生或重複的運算式。

## 風格與格式化

### 一般

- 為資源、變數、輸出提供描述性、一致的名稱。
- 變數和 locals 使用 snake_case。
- 使用 2 個空格縮排；執行 `terraform fmt -recursive`。

### 配置與檔案

建議的結構：
```text
my-sap-btp-app/
├── infra/                      # 根模組
│   ├── main.tf                 # 核心資源（當大型時按域分割）
│   ├── variables.tf            # 輸入
│   ├── outputs.tf              # 輸出
│   ├── provider.tf             # 提供者設定
│   ├── locals.tf               # 本地/派生值
│   └── environments/           # 僅環境變數檔案
│       ├── dev.tfvars
│       ├── test.tfvars
│       └── prod.tfvars
├── .github/workflows/          # CI/CD（如果是 GitHub）
└── README.md                   # 文件
```

規則：
- 不要為每個環境建立單獨的分支/儲存庫/資料夾（反模式）。
- 保持環境漂移最小；僅在 *.tfvars 檔案中編碼差異。
- 將超大的 `main.tf` / `variables.tf` 分割成邏輯命名的片段（例如，`main_services.tf`、`variables_services.tf`）。
  保持命名一致。

### 資源區塊組織

順序（由上而下）：可選的 `depends_on`、然後是 `count`/`for_each`、然後是屬性，最後是 `lifecycle`。
- 僅在 Terraform 無法推斷相依性時使用 `depends_on`（例如，資料源需要權利）。
- 對可選的單一資源使用 `count`；對由地圖鍵化的多個執行個體使用 `for_each` 以獲得穩定的位址。
- 分組屬性：首先是必需的，然後是可選的；邏輯區段之間使用空行。
- 在區段中按字母順序排列以便更快掃描。

### 變數
- 每個變數：明確的 `type`、非空 `description`。
- 優先使用具體類型（`object`、`map(string)` 等）而不是 `any`。
- 避免集合的 null 預設值；改用空清單/地圖。

### Locals
- 集中化計算或重複的運算式。
- 將相關值分組為物件 locals 以提高內聚力。

### 輸出
- 僅公開下游模組/自動化使用的內容。
- 將祕密標記為 `sensitive = true`。
- 始終提供清晰的 `description`。

### 格式化與 Linting
- 執行 `terraform fmt -recursive`（在 CI 中為必需）。
- 在 pre-commit / CI 中強制執行 `tflint`（以及可選的 `terraform validate`）。

## 文件

必須：
- 在所有變數和輸出上使用 `description` + `type`。
- 簡潔的根 `README.md`：目的、先決條件、驗證模型、使用方式（init/plan/apply）、測試、回復。
- 使用 `terraform-docs` 產生模組文件（如果可能，新增至 CI）。
- 僅在澄清非明顯決定或約束的地方進行評論。

## 狀態管理
- 使用支援鎖定的遠端後端（例如，Terraform Cloud、AWS S3、GCS、Azure 儲存體）。避免 SAP BTP 物件儲存（對可靠鎖定和安全性的功能不足）。
- 永不提交 `*.tfstate` 或備份。
- 加密靜止和傳輸中的狀態；按最少權限原則限制存取。

## 驗證
- 在提交前執行 `terraform validate`（語法和內部檢查）。
- 在執行 `terraform plan` 前與使用者確認（需要驗證和全域帳戶子網域）。透過環境變數或 tfvars 提供驗證；永不在提供者區塊中內聯祕密。
- 首先在非生產環境中測試；確保冪等應用。

## 測試
- 使用 Terraform 測試框架（`*.tftest.hcl`）來測試模組邏輯和不變量。
- 涵蓋成功和失敗路徑；保持測試無狀態/冪等。
- 在可行的地方優先模擬外部資料源。

## SAP BTP 提供者特定內容

指導方針：
- 使用 `data "btp_subaccount_service_plan"` 解析服務計畫 ID，並從該資料源參考 `serviceplan_id`。

範例：
```terraform
data "btp_subaccount_service_plan" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
}

resource "btp_subaccount_service_instance" "example" {
  subaccount_id  = var.subaccount_id
  serviceplan_id = data.btp_subaccount_service_plan.example.id
  name           = "my-example-instance"
}
```

明確的相依性（提供者無法推斷）：
```terraform
resource "btp_subaccount_entitlement" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
}

data "btp_subaccount_service_plan" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
  depends_on    = [btp_subaccount_entitlement.example]
}
```

訂閱也取決於權利；當提供者無法透過屬性推斷連結時，新增 `depends_on`（符合 `service_name`/`plan_name` ↔ `app_name`）。

## 工具整合

### HashiCorp Terraform MCP 伺服器
使用 Terraform MCP 伺服器進行互動式架構查詢、資源區塊草稿和驗證。
1. 安裝並執行伺服器（請參閱 https://github.com/mcp/hashicorp/terraform-mcp-server）。
2. 將其作為工具新增至您的 Copilot / MCP 用戶端設定中。
3. 在撰寫前查詢提供者架構（例如，列出資源、資料源）。
4. 產生草稿資源區塊，然後根據命名和標記標準手動精煉。
5. 驗證計畫摘要（永不包括祕密）；在 `apply` 前與審查者確認差異。

### Terraform 登錄表
參考 SAP BTP 提供者文件：https://registry.terraform.io/providers/SAP/btp/latest/docs 以獲得權威的資源和資料源欄位。如果不確定，請與登錄表文件交叉檢查 MCP 回應。

## 反模式（避免）

組態：
- 硬編碼的環境特定值（使用變數和 tfvars）。
- `terraform import` 的常規使用（僅限遷移）。
- 減少清晰度的深層/不透明條件邏輯和動態區塊。
- `local-exec` 佈建程式除了無可避免的整合間隙。
- 在同一根中混合 SAP BTP 提供者和 Cloud Foundry 提供者，除非明確合理化（分割模組）。

安全性：
- 在 HCL、狀態或 VCS 中儲存祕密。
- 為了速度而停用加密、驗證或掃描。
- 使用預設密碼/金鑰或在環境之間重複使用認證。

操作上：
- 直接生產應用而不進行先前的非生產驗證。
- Terraform 外部的手動漂移變更。
- 忽略狀態不一致/損毀症狀。
- 從不受控的本機筆電執行生產應用（使用 CI/CD 或已核准的執行者）。
- 從原始 `*.tfstate` 而不是輸出/資料源讀取業務資料。

所有變更必須通過 Terraform CLI + HCL 流動 – 絕不手動變更狀態。
