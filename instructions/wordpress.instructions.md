---
applyTo: 'wp-content/plugins/**,wp-content/themes/**,**/*.php,**/*.inc,**/*.js,**/*.jsx,**/*.ts,**/*.tsx,**/*.css,**/*.scss,**/*.json'
description: 'Coding, security, and testing rules for WordPress plugins and themes'
---

# WordPress 開發——Copilot 指引

**目標：** 產生安全、高效能、可測試且符合官方 WordPress 實踐的程式碼。偏好使用掛勾、小型函式、依賴注入（適當時）以及清晰的關注點分離。

## 1) 核心原則
- 絕不修改 WordPress 核心。透過 **actions** 和 **filters** 進行擴展。
- 對於外掛，務必在入口 PHP 檔案中包含標頭和防守直接執行。
- 使用唯一的前綴或 PHP namespace 以避免全域衝突。
- 入隊資源；決不在 PHP 模板中內聯原始 `<script>`/`<style>`。
- 使用者面對的字串應可翻譯，並載入正確的文字域。

### 最小外掛標頭與防守
```php
<?php
defined('ABSPATH') || exit;
/**
 * Plugin Name: Awesome Feature
 * Description: Example plugin scaffold.
 * Version: 0.1.0
 * Author: Example
 * License: GPL-2.0-or-later
 * Text Domain: awesome-feature
 * Domain Path: /languages
 */
```

## 2) 編碼標準（PHP、JS、CSS、HTML）
- 遵循 **WordPress 編碼標準（WPCS）** 並為公開 API 編寫 DocBlock。
- PHP：在適當的地方偏好嚴格比較（`===`、`!==`）。保持與 WPCS 一致的陣列語法和間距。
- JS：符合 WordPress JS 風格；對於區塊/編輯器程式碼優先使用 `@wordpress/*` 套件。
- CSS：在有幫助時使用類 BEM 的類別命名；避免過於具體的選擇器。
- PHP 7.4+ 相容的模式，除非專案指定更高版本。避免使用目標 WP/PHP 版本不支援的功能。

### 檢查工具設定建議
```xml
<!-- phpcs.xml -->
<?xml version="1.0"?>
<ruleset name="Project WPCS">
  <description>WordPress Coding Standards for this project.</description>
  <file>./</file>
  <exclude-pattern>vendor/*</exclude-pattern>
  <exclude-pattern>node_modules/*</exclude-pattern>
  <rule ref="WordPress"/>
  <rule ref="WordPress-Docs"/>
  <rule ref="WordPress-Extra"/>
  <rule ref="PHPCompatibility"/>
  <config name="testVersion" value="7.4-"/>
</ruleset>
```

```json
// composer.json (片段)
{
  "require-dev": {
    "dealerdirect/phpcodesniffer-composer-installer": "^1.0",
    "wp-coding-standards/wpcs": "^3.0",
    "phpcompatibility/php-compatibility": "^9.0"
  },
  "scripts": {
    "lint:php": "phpcs -p",
    "fix:php": "phpcbf -p"
  }
}
```

```json
// package.json (片段)
{
  "devDependencies": {
    "@wordpress/eslint-plugin": "^x.y.z"
  },
  "scripts": {
    "lint:js": "eslint ."
  }
}
```

## 3) 安全性與資料處理
- **輸出時逸出，輸入時淨化。**
  - 逸出：`esc_html()`、`esc_attr()`、`esc_url()`、`wp_kses_post()`。
  - 淨化：`sanitize_text_field()`、`sanitize_email()`、`sanitize_key()`、`absint()`、`intval()`。
- **能力與 nonces** 用於表單、AJAX、REST：
  - 使用 `wp_nonce_field()` 新增 nonces，並透過 `check_admin_referer()` / `wp_verify_nonce()` 驗證。
  - 使用 `current_user_can( 'manage_options' /* 或特定能力 */ )` 限制變異。
- **資料庫：** 始終使用 `$wpdb->prepare()` 搭配佔位符；決不串聯不受信任的輸入。
- **上傳：** 驗證 MIME/類型並使用 `wp_handle_upload()`/`media_handle_upload()`。

## 4) 國際化（i18n）
- 使用文字域將使用者可見的字串包裝在翻譯函式中：
  - `__( 'Text', 'awesome-feature' )`、`_x()`、`esc_html__()`。
- 使用 `load_plugin_textdomain()` 或 `load_theme_textdomain()` 載入翻譯。
- 在 `/languages` 中保留 `.pot` 檔案，並確保一致的域使用。

## 5) 效能
- 將繁重邏輯延遲到特定掛勾；除非必要，否則避免在 `init`/`wp_loaded` 上進行昂貴工作。
- 為昂貴查詢使用 transient 或物件快取；規劃無效化。
- 僅入隊所需的資源，並有條件地入隊（前端與管理；特定螢幕/路由）。
- 優先使用分頁/參數化查詢，而不是無限迴圈。

## 6) 管理界面與設定
- 對選項頁面使用 **Settings API**；為每個設定提供 `sanitize_callback`。
- 對於表格，遵循 `WP_List_Table` 模式。對於通知，使用管理通知 API。
- 避免為複雜 UI 直接輸出 HTML；偏好範本或有逸出的小檢視助手。

## 7) REST API
- 使用 `register_rest_route()` 註冊；始終設定 `permission_callback`。
- 透過 `args` 結構描述驗證/淨化請求引數。
- 傳回 `WP_REST_Response` 或乾淨映射到 JSON 的陣列/物件。

## 8) 區塊與編輯器（Gutenberg）
- 使用 `block.json` + `register_block_type()`；依靠 `@wordpress/*` 套件。
- 在需要時提供伺服器轉譯回呼（動態區塊）。
- E2E 測試應涵蓋：插入區塊 → 編輯 → 儲存 → 前端轉譯。

## 9) 資源載入
```php
add_action('wp_enqueue_scripts', function () {
  wp_enqueue_style(
    'af-frontend',
    plugins_url('assets/frontend.css', __FILE__),
    [],
    '0.1.0'
  );

  wp_enqueue_script(
    'af-frontend',
    plugins_url('assets/frontend.js', __FILE__),
    [ 'wp-i18n', 'wp-element' ],
    '0.1.0',
    true
  );
});
```
- 如果多個元件依賴相同資源，使用 `wp_register_style/script` 先註冊。
- 對管理螢幕，掛在 `admin_enqueue_scripts` 上，並檢查螢幕 ID。

## 10) 測試
### PHP 單元/整合測試
- 使用 **WordPress 測試套件**搭配 `PHPUnit` 和 `WP_UnitTestCase`。
- 測試：淨化、能力檢查、REST 權限、資料庫查詢、掛勾。
- 優先使用工廠（`self::factory()->post->create()` 等）來設定測試夾具。

```xml
<!-- phpunit.xml.dist (最小) -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap.php" colors="true">
  <testsuites>
    <testsuite name="Plugin Test Suite">
      <directory suffix="Test.php">tests/</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

```php
// tests/bootstrap.php (最小草圖)
<?php
$_tests_dir = getenv('WP_TESTS_DIR') ?: '/tmp/wordpress-tests-lib';
require_once $_tests_dir . '/includes/functions.php';
tests_add_filter( 'muplugins_loaded', function () {
  require dirname(__DIR__) . '/awesome-feature.php';
} );
require $_tests_dir . '/includes/bootstrap.php';
```

### E2E 測試
- 使用 Playwright（或 Puppeteer）進行編輯器/前端流程。
- 涵蓋基本使用者旅程和迴歸（區塊插入、設定儲存、前端轉譯）。

## 11) 文件與提交
- 保持 `README.md` 最新：安裝、使用方式、能力、掛勾/篩選器和測試指引。
- 使用清晰的祈使式提交訊息；參考議題/票據並總結影響。

## 12) Copilot 必須確保（檢查清單）
- ✅ 唯一的前綴/namespace；無意外的全域。
- ✅ 任何寫入動作的 nonce + 能力檢查（AJAX/REST/表單）。
- ✅ 輸入已淨化；輸出已逸出。
- ✅ 使用者可見的字串以 i18n 包裝，具有正確的文字域。
- ✅ 資源透過 API 入隊（無內聯指令碼/樣式）。
- ✅ 為新行為新增/更新測試。
- ✅ 程式碼通過 PHPCS（WPCS）和 ESLint（適用時）。
- ✅ 避免直接資料庫串聯；始終準備查詢。
