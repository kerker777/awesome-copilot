---
description: 'Ruby on Rails coding conventions and guidelines'
applyTo: '**/*.rb'
---

# Ruby on Rails

## 通用指南

- 遵循 RuboCop 風格指南，使用 `rubocop`、`standardrb` 或 `rufo` 等工具維持一致的程式碼格式。
- 變數和方法使用 snake_case，類別和模組使用 CamelCase。
- 保持方法簡潔且專注，使用早期返回、衛語句和私有方法來降低複雜度。
- 優先使用有意義的名稱，而不是簡短或泛用的名稱。
- 只在必要時才加上註解，避免解釋顯而易見的事情。
- 對類別、方法和模組應用單一責任原則。
- 優先選擇組合而不是繼承，將可重用的邏輯提取到模組或服務中。
- 保持控制器精簡，將商業邏輯移入模型、服務或命令/查詢物件。
- 明智地應用「肥厚模型，精簡控制器」的模式，並使用清晰的抽象。
- 將商業邏輯提取到服務物件中，以提高可重用性和可測試性。
- 使用部分視圖或視圖元件來減少重複並簡化視圖。
- 對於否定條件使用 `unless`，但為了清晰起見，避免搭配 `else` 使用。
- 避免深層嵌套的條件判斷，優先使用衛語句和方法提取。
- 使用安全導航 (`&.`) 來替代多個 `nil` 檢查。
- 優先使用 `.present?`、`.blank?` 和 `.any?`，而不是手動的 nil/空值檢查。
- 在路由和控制器動作中遵循 RESTful 慣例。
- 使用 Rails 產生器來一致地建立資源架構。
- 使用強參數安全地白名單化屬性。
- 優先使用列舉和類型化屬性，以獲得更清晰的模型和驗證。
- 保持遷移資料庫不可知，盡可能避免原始 SQL。
- 始終為外鍵和經常查詢的欄位添加索引。
- 在資料庫層級定義 `null: false` 和 `unique: true`，而不只是在模型中。
- 使用 `find_each` 來迭代大型資料集，以減少記憶體使用。
- 在模型中設定查詢範圍或使用查詢物件以提高清晰度和可重用性。
- 節制使用 `before_action` 回調，避免在其中包含商業邏輯。
- 使用 `Rails.cache` 來儲存耗費計算資源的運算結果或經常存取的資料。
- 使用 `Rails.root.join(...)` 來構建檔案路徑，而不是硬編碼。
- 在關聯中使用 `class_name` 和 `foreign_key` 來明確表達關係。
- 使用 `Rails.application.credentials` 或環境變數將祕密和設定保存在程式碼庫外。
- 為模型、服務和輔助工具撰寫獨立的單元測試。
- 用請求/系統測試覆蓋端到端邏輯。
- 使用背景工作 (ActiveJob) 來執行非阻塞操作，例如發送電子郵件或呼叫 API。
- 使用 `FactoryBot` (RSpec) 或 fixtures (Minitest) 來乾淨地設定測試資料。
- 避免使用 `puts`，改用 `byebug`、`pry` 或日誌工具進行除錯。
- 使用 YARD 或 RDoc 來記錄複雜的程式碼路徑和方法。

## 應用程式目錄結構

- 在 `app/services` 目錄中定義服務物件來封裝商業邏輯。
- 使用位於 `app/forms` 的表單物件來管理驗證和提交邏輯。
- 在 `app/serializers` 目錄中實作 JSON 序列化器來格式化 API 響應。
- 在 `app/policies` 中定義授權政策來控制使用者對資源的存取。
- 通過在 `app/graphql` 中組織 schemas、查詢和變更來構建 GraphQL API。
- 在 `app/validators` 中建立自訂驗證器來強制執行專門的驗證邏輯。
- 在 `app/queries` 中隔離並封裝複雜的 ActiveRecord 查詢，以提高可重用性和可測試性。
- 在 `app/types` 目錄中定義自訂資料類型和強制轉型邏輯，以擴展或覆蓋 ActiveModel 的類型行為。

## 指令

- 使用 `rails generate` 來建立新的模型、控制器和遷移。
- 使用 `rails db:migrate` 來套用資料庫遷移。
- 使用 `rails db:seed` 來用初始資料填充資料庫。
- 使用 `rails db:rollback` 來復原最後一個遷移。
- 使用 `rails console` 來在 REPL 環境中與 Rails 應用程式互動。
- 使用 `rails server` 來啟動開發伺服器。
- 使用 `rails test` 來執行測試套件。
- 使用 `rails routes` 來列出應用程式中所有定義的路由。
- 使用 `rails assets:precompile` 來編譯生產環境的資產。


## API 開發最佳實踐

- 使用 Rails 的 `resources` 來構建路由，遵循 RESTful 慣例。
- 使用命名空間路由（例如 `/api/v1/`）來進行版本控制和向前相容。
- 使用 `ActiveModel::Serializer` 或 `fast_jsonapi` 來序列化響應，確保輸出一致。
- 為每個響應返回適當的 HTTP 狀態碼（例如 200 OK、201 Created、422 Unprocessable Entity）。
- 使用 `before_action` 篩選器來載入和授權資源，而不是商業邏輯。
- 利用分頁（例如 `kaminari` 或 `pagy`）來處理返回大型資料集的端點。
- 使用中介軟體或 `rack-attack` 等 gem 來限制速率並節流敏感端點。
- 以結構化的 JSON 格式返回錯誤，包括錯誤碼、訊息和詳細資訊。
- 使用強參數來清理和白名單化輸入參數。
- 使用自訂序列化器或展示器來解耦內部邏輯和響應格式。
- 通過在預加載相關資料時使用 `includes` 來避免 N+1 查詢。
- 實作背景工作來執行非阻塞任務，例如發送電子郵件或與外部 API 同步。
- 記錄請求/響應中繼資料以供除錯、可觀測性和稽核之用。
- 使用 OpenAPI (Swagger)、`rswag` 或 `apipie-rails` 來記錄端點。
- 在需要時使用 CORS 標頭 (`rack-cors`) 來允許 API 的跨來源存取。
- 確保敏感資料永遠不會在 API 響應或錯誤訊息中暴露。

## 前端開發最佳實踐

- 在 Rails 6+ 中使用 `app/javascript` 作為管理 JavaScript 套件、模組和前端邏輯的主要目錄，搭配 Webpacker 或 esbuild。
- 根據元件或領域而不是檔案類型來組織 JavaScript，以保持模組化。
- 利用 Hotwire (Turbo + Stimulus) 在 Rails 原生應用程式中實現實時更新和最少的 JavaScript。
- 使用 Stimulus 控制器來將行為綁定到 HTML 並以宣告式方式管理 UI 邏輯。
- 在 `app/assets/stylesheets` 下使用 SCSS 模組、Tailwind 或 BEM 慣例來組織樣式。
- 通過將重複的標記提取到部分視圖或元件中來保持視圖邏輯的清晰。
- 在所有視圖中使用語義 HTML 標籤並遵循無障礙 (a11y) 最佳實踐。
- 避免內聯 JavaScript 和樣式，改為將邏輯移到單獨的 `.js` 或 `.scss` 檔案中，以提高清晰度和可重用性。
- 使用資產管道或打包工具來最佳化資產（圖像、字體、圖示），以進行快取和壓縮。
- 使用 `data-*` 屬性來橋接前端互動與 Rails 產生的 HTML 和 Stimulus。
- 使用系統測試 (Capybara) 或整合測試（例如 Cypress 或 Playwright）來測試前端功能。
- 使用環境特定的資產載入來防止在生產環境中載入不必要的指令碼或樣式。
- 遵循設計系統或元件庫來保持 UI 的一致性和可擴展性。
- 使用延遲載入、Turbo Frames 和延遲 JS 來最佳化首次繪製時間 (TTFP) 和資產載入。

## 測試指南

- 使用 `test/models` (Minitest) 或 `spec/models` (RSpec) 為模型撰寫單元測試來驗證商業邏輯。
- 使用 fixtures (Minitest) 或 `FactoryBot` (RSpec) 的工廠來乾淨且一致地管理測試資料。
- 在 `test/controllers` 或 `spec/requests` 下組織控制器規格來測試 RESTful API 行為。
- 優先使用 RSpec 的 `before` 區塊或 Minitest 的 `setup` 來初始化常見測試資料。
- 避免在測試中呼叫外部 API，使用 `WebMock`、`VCR` 或 `stub_request` 來隔離測試環境。
- 在 Minitest 中使用 `system tests` 或在 RSpec 中使用 Capybara 的 `feature specs` 來模擬完整的使用者流程。
- 將緩慢且昂貴的測試（例如外部服務、檔案上傳）隔離到單獨的測試類型或標籤中。
- 執行 `SimpleCov` 等測試覆蓋工具來確保充分的程式碼覆蓋。
- 避免在測試中使用 `sleep`，改用 `perform_enqueued_jobs` (Minitest) 或 RSpec 的 `ActiveJob::TestHelper`。
- 使用資料庫清理工具（`rails test:prepare`、`DatabaseCleaner` 或 `transactional_fixtures`）來維持測試之間的乾淨狀態。
- 通過使用 `ActiveJob::TestHelper` 或 `have_enqueued_job` 匹配器來排隊和執行工作來測試背景工作。
- 使用 CI 工具（例如 GitHub Actions、CircleCI）確保測試在各環境中一致執行。
- 使用自訂匹配器 (RSpec) 或自訂判斷式 (Minitest) 來實現可重用且具表現力的測試邏輯。
- 按類型標記測試（例如 `:model`、`:request`、`:feature`）以進行更快的並具針對性的測試執行。
- 避免脆弱的測試，除非明確必要，否則不要依賴特定的時間戳、隨機資料或順序。
- 為跨越多層 (模型、視圖、控制器) 的端到端流程撰寫整合測試。
- 保持測試快速、可靠，並像生產程式碼一樣簡潔。
