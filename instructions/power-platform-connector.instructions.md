---
title: Power Platform Connectors Schema Development Instructions
description: 'Comprehensive development guidelines for Power Platform Custom Connectors using JSON Schema definitions. Covers API definitions (Swagger 2.0), API properties, and settings configuration with Microsoft extensions.'
applyTo: '**/*.{json,md}'
---

# Power Platform 連接器結構描述開發指南

## 專案概述
此工作區包含 Power Platform 自訂連接器的 JSON 結構描述定義，特別針對 `paconn`（Power Apps 連接器）工具。結構描述可驗證並為以下項目提供 IntelliSense：

- **API 定義**（Swagger 2.0 格式）
- **API 屬性**（連接器中繼資料和設定）
- **設定**（環境和部署設定）

## 檔案結構說明

### 1. apiDefinition.swagger.json
- **用途**：此檔案包含具有 Power Platform 擴充功能的 Swagger 2.0 API 定義。
- **主要功能**：
  - 標準 Swagger 2.0 屬性，包括 info、paths、definitions 等。
  - 以 `x-ms-*` 前綴開頭的 Microsoft 特定擴充功能。
  - 專為 Power Platform 設計的自訂格式類型，例如 `date-no-tz` 和 `html`。
  - 動態結構描述支援，提供執行時間彈性。
  - 支援 OAuth2、API Key 和基本驗證方法的安全性定義。

### 2. apiProperties.json
- **用途**：此檔案定義連接器中繼資料、驗證設定和政策設定。
- **主要元件**：
  - **連線參數**：這些支援各種驗證類型，包括 OAuth、API Key 和閘道設定。
  - **政策範本執行個體**：這些處理連接器的資料轉換和路由政策。
  - **連接器中繼資料**：包括發行者資訊、功能和品牌元素。

### 3. settings.json
- **用途**：此檔案提供 paconn 工具的環境和部署設定。
- **設定選項**：
  - 針對特定 Power Platform 環境的環境 GUID 目標。
  - 連接器資產和設定檔案的檔案路徑對應。
  - 用於生產環境和測試環境 (PROD/TIP1) 的 API 端點 URL。
  - API 版本規格以確保與 Power Platform 服務的相容性。

## 開發指南

### 使用 API 定義 (Swagger) 時
1. **總是針對 Swagger 2.0 規範進行驗證** - 結構描述強制執行嚴格的 Swagger 2.0 合規性

2. **Microsoft 操作擴充功能**：
   - `x-ms-summary`：使用此選項提供使用者易讀的顯示名稱，並確保使用標題大小寫格式。
   - `x-ms-visibility`：使用此選項控制參數可見性，值為 `important`、`advanced` 或 `internal`。
   - `x-ms-trigger`：使用此選項將操作標記為觸發程序，值為 `batch` 或 `single`。
   - `x-ms-trigger-hint`：使用此選項提供有用的提示文字，在使用者使用觸發程序時引導使用者。
   - `x-ms-trigger-metadata`：使用此選項定義觸發程序設定，包括 kind 和 mode 屬性。
   - `x-ms-notification`：使用此選項設定 webhook 操作以進行即時通知。
   - `x-ms-pageable`：使用此選項透過指定 `nextLinkName` 屬性來啟用分頁功能。
   - `x-ms-safe-operation`：使用此選項在 POST 操作沒有副作用時將其標記為安全。
   - `x-ms-no-generic-test`：使用此選項停用特定操作的自動測試。
   - `x-ms-operation-context`：使用此選項設定操作模擬設定以進行測試。

3. **Microsoft 參數擴充功能**：
   - `x-ms-dynamic-list`：使用此選項啟用從 API 呼叫填入的動態下拉清單。
   - `x-ms-dynamic-values`：使用此選項設定動態值來源以填入參數選項。
   - `x-ms-dynamic-tree`：使用此選項為巢狀資料結構建立階層式選擇器。
   - `x-ms-dynamic-schema`：使用此選項允許根據使用者選擇進行執行時間結構描述變更。
   - `x-ms-dynamic-properties`：使用此選項進行動態屬性設定，該設定會根據內容調整。
   - `x-ms-enum-values`：使用此選項提供增強的列舉定義，並包含顯示名稱以提供更好的使用者體驗。
   - `x-ms-test-value`：使用此選項提供測試的範例值，但永遠不要包含機密或敏感資料。
   - `x-ms-trigger-value`：使用此選項為觸發程序參數指定值，並具有 `value-collection` 和 `value-path` 屬性。
   - `x-ms-url-encoding`：使用此選項指定 URL 編碼風格為 `single` 或 `double`（預設為 `single`）。
   - `x-ms-parameter-location`：使用此選項為 API 提供參數位置提示（AutoRest 擴充功能 - Power Platform 忽略）。
   - `x-ms-localizeDefaultValue`：使用此選項啟用預設參數值的當地語系化。
   - `x-ms-skip-url-encoding`：使用此選項略過路徑參數的 URL 編碼（AutoRest 擴充功能 - Power Platform 忽略）。

4. **Microsoft 結構描述擴充功能**：
   - `x-ms-notification-url`：使用此選項將結構描述屬性標記為 webhook 設定的通知 URL。
   - `x-ms-media-kind`：使用此選項指定內容的媒體類型，支援的值為 `image` 或 `audio`。
   - `x-ms-enum`：使用此選項提供增強的列舉中繼資料（AutoRest 擴充功能 - Power Platform 忽略）。
   - 請注意，上面列出的所有參數擴充功能也適用於結構描述屬性，並可在結構描述定義中使用。

5. **根層級擴充功能**：
   - `x-ms-capabilities`：使用此選項定義連接器功能，例如檔案選擇器和 testConnection 功能。
   - `x-ms-connector-metadata`：使用此選項提供標準屬性以外的其他連接器中繼資料。
   - `x-ms-docs`：使用此選項設定連接器的文件設定和參考。
   - `x-ms-deployment-version`：使用此選項追蹤部署管理的版本資訊。
   - `x-ms-api-annotation`：使用此選項新增 API 層級批註以增強功能。

6. **路徑層級擴充功能**：
   - `x-ms-notification-content`：使用此選項定義 webhook 路徑項目的通知內容結構描述。

7. **操作層級功能**：
   - `x-ms-capabilities`（在操作層級）：使用此選項啟用操作特定的功能，例如用於大型檔案傳輸的 `chunkTransfer`。

8. **安全性考慮**：
   - 您應該為您的 API 定義適當的 `securityDefinitions`，以確保適當的驗證。
   - **允許多個安全性定義** - 您可以定義最多兩個驗證方法（例如 oauth2 + apiKey、basic + apiKey）。
   - **例外**：如果使用「無」驗證，連接器中不能存在其他安全性定義。
   - 您應該為現代 API 使用 `oauth2`，為簡單的權杖驗證使用 `apiKey`，並且僅為內部/舊版系統考慮 `basic` 驗證。
   - 每個安全性定義必須是正確的類型（此條件約束由 oneOf 驗證強制執行）。

9. **參數最佳做法**：
   - 您應該使用描述性的 `description` 欄位，以協助使用者瞭解每個參數的用途。
   - 您應該實作 `x-ms-summary` 以提供更好的使用者體驗（需要標題大小寫）。
   - 您必須正確標記必要參數以確保適當的驗證。
   - 您應該使用適當的 `format` 值（包括 Power Platform 擴充功能）以啟用適當的資料處理。
   - 您應該利用動態擴充功能以提供更好的使用者體驗和資料驗證。

10. **Power Platform 格式擴充功能**：
   - `date-no-tz`：這表示沒有時間偏移資訊的日期時間。
   - `html`：此格式告訴用戶端在編輯時發出 HTML 編輯器，在檢視內容時發出 HTML 檢視器。
   - 標準格式包括：`int32`、`int64`、`float`、`double`、`byte`、`binary`、`date`、`date-time`、`password`、`email`、`uri`、`uuid`。

### 使用 API 屬性時
1. **連線參數**：
   - 您應該選擇適當的參數類型，例如 `string`、`securestring` 或 `oauthSetting`。
   - 您應該使用正確的身分識別提供者設定 OAuth 設定。
   - 您應該在適當時使用 `allowedValues` 進行下拉清單選項。
   - 您應該在需要時實作參數依賴性以進行條件式參數。

2. **政策範本**：
   - 您應該使用 `routerequesttoendpoint` 將後端路由至不同的 API 端點。
   - 您應該實作 `setqueryparameter` 以設定查詢參數的預設值。
   - 您應該使用 `updatenextlink` 進行分頁案例，以正確處理分頁。
   - 您應該為需要輪詢行為的觸發程序操作應用 `pollingtrigger`。

3. **品牌和中繼資料**：
   - 您必須始終指定 `iconBrandColor`，因為所有連接器都需要此屬性。
   - 您應該定義適當的 `capabilities`，以指定您的連接器是否支援動作或觸發程序。
   - 您應該設定有意義的 `publisher` 和 `stackOwner` 值以識別連接器的擁有權。

### 使用設定時
1. **環境設定**：
   - 您應該為 `environment` 使用與驗證模式相符的適當 GUID 格式。
   - 您應該為目標環境設定正確的 `powerAppsUrl` 和 `flowUrl`。
   - 您應該將 API 版本與您的特定需求相符。

2. **檔案參考**：
   - 您應該使用 `apiProperties.json` 和 `apiDefinition.swagger.json` 的預設值保持一致的檔案命名。
   - 您應該為本機開發環境使用相對路徑。
   - 您應該確保圖示檔案存在並在您的設定中正確參考。

## 結構描述驗證規則

### 必要屬性
- **API 定義**：`swagger: "2.0"`、`info`（具有 `title` 和 `version`）、`paths`
- **API 屬性**：具有 `iconBrandColor` 的 `properties`
- **設定**：無必要屬性（全部可選，具有預設值）

### 模式驗證
- **廠商擴充功能**：非 Microsoft 擴充功能必須符合 `^x-(?!ms-)` 模式
- **路徑項目**：API 路徑必須以 `/` 開頭
- **環境 GUID**：必須符合 UUID 格式模式 `^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$`
- **URL**：端點設定必須是有效的 URI
- **主機模式**：必須符合 `^[^{}/ :\\\\]+(?::\\d+)?$`（不含空格、通訊協定或路徑）

### 類型條件約束
- **安全性定義**：
  - `securityDefinitions` 物件中最多允許兩個安全性定義
  - 每個個別安全性定義必須是正確的類型（oneOf 驗證：`basic`、`apiKey`、`oauth2`）
  - **例外**：「無」驗證不能與其他安全性定義共存
- **參數類型**：限於特定列舉值（`string`、`number`、`integer`、`boolean`、`array`、`file`）
- **政策範本**：類型特定的參數需求
- **格式值**：擴展集合，包括 Power Platform 格式
- **可見性值**：必須為 `important`、`advanced` 或 `internal` 之一
- **觸發程序類型**：必須為 `batch` 或 `single`

### 其他驗證規則
- **$ref 參考**：應僅指向 `#/definitions/`、`#/parameters/` 或 `#/responses/`
- **路徑參數**：必須標記為 `required: true`
- **Info 物件**：描述應與標題不同
- **Contact 物件**：電子郵件必須是有效的電子郵件格式，URL 必須是有效的 URI
- **License 物件**：名稱是必要的，如果提供了 URL 必須是有效的 URI
- **外部文件**：需要 URL，並必須是有效的 URI
- **標籤**：陣列內必須有唯一的名稱
- **配置**：必須是有效的 HTTP 配置（`http`、`https`、`ws`、`wss`）
- **MIME 類型**：在 `consumes` 和 `produces` 中必須遵循有效的 MIME 類型格式

## 常見模式和範例

### API 定義範例

#### 具有 Microsoft 擴充功能的基本操作
```json
{
  "get": {
    "operationId": "GetItems",
    "summary": "Get items",
    "x-ms-summary": "Get Items",
    "x-ms-visibility": "important",
    "description": "Retrieves a list of items from the API",
    "parameters": [
      {
        "name": "category",
        "in": "query",
        "type": "string",
        "x-ms-summary": "Category",
        "x-ms-visibility": "important",
        "x-ms-dynamic-values": {
          "operationId": "GetCategories",
          "value-path": "id",
          "value-title": "name"
        }
      }
    ],
    "responses": {
      "200": {
        "description": "Success",
        "x-ms-summary": "Success",
        "schema": {
          "type": "object",
          "properties": {
            "items": {
              "type": "array",
              "x-ms-summary": "Items",
              "items": {
                "$ref": "#/definitions/Item"
              }
            }
          }
        }
      }
    }
  }
}
```

#### 觸發程序操作設定
```json
{
  "get": {
    "operationId": "WhenItemCreated",
    "x-ms-summary": "When an Item is Created",
    "x-ms-trigger": "batch",
    "x-ms-trigger-hint": "To see it work now, create an item",
    "x-ms-trigger-metadata": {
      "kind": "query",
      "mode": "polling"
    },
    "x-ms-pageable": {
      "nextLinkName": "@odata.nextLink"
    }
  }
}
```

#### 動態結構描述範例
```json
{
  "name": "dynamicSchema",
  "in": "body",
  "schema": {
    "x-ms-dynamic-schema": {
      "operationId": "GetSchema",
      "parameters": {
        "table": {
          "parameter": "table"
        }
      },
      "value-path": "schema"
    }
  }
}
```

#### 檔案選擇器功能
```json
{
  "x-ms-capabilities": {
    "file-picker": {
      "open": {
        "operationId": "OneDriveFilePickerOpen",
        "parameters": {
          "dataset": {
            "value-property": "dataset"
          }
        }
      },
      "browse": {
        "operationId": "OneDriveFilePickerBrowse",
        "parameters": {
          "dataset": {
            "value-property": "dataset"
          }
        }
      },
      "value-title": "DisplayName",
      "value-collection": "value",
      "value-folder-property": "IsFolder",
      "value-media-property": "MediaType"
    }
  }
}
```

#### 測試連線功能（注意：自訂連接器不支援）
```json
{
  "x-ms-capabilities": {
    "testConnection": {
      "operationId": "TestConnection",
      "parameters": {
        "param1": "literal-value"
      }
    }
  }
}
```

#### 模擬的操作內容
```json
{
  "x-ms-operation-context": {
    "simulate": {
      "operationId": "SimulateOperation",
      "parameters": {
        "param1": {
          "parameter": "inputParam"
        }
      }
    }
  }
}
```

### 基本 OAuth 設定
```json
{
  "type": "oauthSetting",
  "oAuthSettings": {
    "identityProvider": "oauth2",
    "clientId": "your-client-id",
    "scopes": ["scope1", "scope2"],
    "redirectMode": "Global"
  }
}
```

#### 多個安全性定義範例
```json
{
  "securityDefinitions": {
    "oauth2": {
      "type": "oauth2",
      "flow": "accessCode",
      "authorizationUrl": "https://api.example.com/oauth/authorize",
      "tokenUrl": "https://api.example.com/oauth/token",
      "scopes": {
        "read": "Read access",
        "write": "Write access"
      }
    },
    "apiKey": {
      "type": "apiKey",
      "name": "X-API-Key",
      "in": "header"
    }
  }
}
```

**注意**：最多兩個安全性定義可以共存，但「無」驗證不能與其他方法組合。

### 動態參數設定
```json
{
  "x-ms-dynamic-values": {
    "operationId": "GetItems",
    "value-path": "id",
    "value-title": "name"
  }
}
```

### 路由的政策範本
```json
{
  "templateId": "routerequesttoendpoint",
  "title": "Route to backend",
  "parameters": {
    "x-ms-apimTemplate-operationName": ["GetData"],
    "x-ms-apimTemplateParameter.newPath": "/api/v2/data"
  }
}
```

## 最佳做法

1. **使用 IntelliSense**：這些結構描述提供豐富的自動完成和驗證功能，在開發期間提供協助。
2. **遵循命名慣例**：為操作和參數使用描述性名稱以改善程式碼可讀性。
3. **實作錯誤處理**：定義適當的回應結構描述和錯誤碼，以正確處理失敗案例。
4. **充分測試**：部署前驗證結構描述，以在開發流程早期捕捉問題。
5. **文件擴充功能**：註解 Microsoft 特定擴充功能，以便團隊理解和未來維護。
6. **版本管理**：在 API 資訊中使用語義版本控制來追蹤變更和相容性。
7. **安全第一**：始終實作適當的驗證機制以保護 API 端點。

## 疑難排解

### 常見結構描述違規
- **遺失必要屬性**：`swagger: "2.0"`、`info.title`、`info.version`、`paths`
- **無效的模式格式**：
  - GUID 必須符合確切格式 `^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$`
  - URL 必須是具有適當配置的有效 URI
  - 路徑必須以 `/` 開頭
  - 主機不得包含通訊協定、路徑或空格
- **不正確的廠商擴充功能命名**：對 Microsoft 擴充功能使用 `x-ms-*`，對其他使用 `^x-(?!ms-)`
- **不相符的安全性定義類型**：每個安全性定義必須是正確的類型
- **無效的列舉值**：檢查 `x-ms-visibility`、`x-ms-trigger` 參數類型的允許值
- **$ref 指向無效位置**：必須指向 `#/definitions/`、`#/parameters/` 或 `#/responses/`
- **路徑參數未標記為必要**：所有路徑參數必須具有 `required: true`
- **類型「檔案」在錯誤的內容中**：僅在 `formData` 參數中允許，不在結構描述中

### API 定義特定問題
- **動態結構描述衝突**：不能將 `x-ms-dynamic-schema` 與固定結構描述屬性一起使用
- **觸發程序設定錯誤**：`x-ms-trigger-metadata` 需要 `kind` 和 `mode`
- **分頁設定**：`x-ms-pageable` 需要 `nextLinkName` 屬性
- **檔案選擇器設定錯誤**：必須包含 `open` 操作和必要屬性
- **功能衝突**：某些功能可能與某些參數類型衝突
- **測試值安全性**：永遠不要在 `x-ms-test-value` 中包含機密或個人可識別資訊
- **操作內容設定**：`x-ms-operation-context` 需要具有 `operationId` 的 `simulate` 物件
- **通知內容結構描述**：路徑層級 `x-ms-notification-content` 必須定義適當的結構描述結構
- **媒體種類限制**：`x-ms-media-kind` 僅支援 `image` 或 `audio` 值
- **觸發程序值設定**：`x-ms-trigger-value` 必須至少有一個屬性（`value-collection` 或 `value-path`）

### 驗證工具
- 使用 JSON 結構描述驗證器檢查您的結構描述定義是否合規。
- 利用 VS Code 的內建結構描述驗證在開發期間捕捉錯誤。
- 在部署前使用 paconn CLI 進行測試：`paconn validate --api-def apiDefinition.swagger.json`
- 針對 Power Platform 連接器需求進行驗證以確保相容性。
- 使用 Power Platform 連接器入口網站進行驗證和在目標環境中進行測試。
- 檢查操作回應是否符合預期的結構描述以防止執行時間錯誤。

記住：這些結構描述確保您的 Power Platform 連接器格式正確，並且將在 Power Platform 生態系統中正確運作。
