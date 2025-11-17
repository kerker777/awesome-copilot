---
description: 'Playwright Python AI test generation instructions based on official documentation.'
applyTo: '**'
---

# Playwright Python 測試生成指南

## 測試撰寫指南

### 程式碼品質標準
- **定位器（Locators）**：優先使用面向使用者的、基於角色的定位器（get_by_role、get_by_label、get_by_text）以提高彈性和無障礙性。
- **斷言（Assertions）**：透過 expect API 使用自動重試的 web-first 斷言（例如 expect(page).to_have_title(...)）。除非特別測試元素可見性的變化，否則避免使用 expect(locator).to_be_visible()，因為更具體的斷言通常更可靠。
- **超時（Timeouts）**：依賴 Playwright 的內建自動等待機制。避免硬編碼等待或增加預設超時時間。
- **清晰度**：使用描述性的測試標題（例如 def test_navigation_link_works():），清楚地說明其意圖。僅在解釋複雜邏輯時添加註解，而不是描述簡單動作（如「點擊按鈕」）。

### 測試結構
- **匯入（Imports）**：每個測試檔案應以 from playwright.sync_api import Page, expect 開頭。
- **固件（Fixtures）**：使用 page: Page 固件作為測試函式的參數來與瀏覽器頁面互動。
- **設置（Setup）**：在每個測試函式的開頭放置導航步驟（如 page.goto()）。對於多個測試共享的設置動作，使用標準的 Pytest fixtures。

### 檔案組織
- **位置**：將測試檔案儲存在專用的 tests/ 目錄中，或遵循現有的專案結構。
- **命名**：測試檔案必須遵循 test_<feature-or-page>.py 命名慣例，才能被 Pytest 發現。
- **範圍**：每個主要應用程式功能或頁面設置一個測試檔案。

## 斷言最佳實踐
- **元素計數**：使用 expect(locator).to_have_count() 來斷言定位器找到的元素數量。
- **文字內容**：對於精確的文字匹配使用 expect(locator).to_have_text()，對於部分匹配使用 expect(locator).to_contain_text()。
- **導航**：使用 expect(page).to_have_url() 來驗證頁面 URL。
- **斷言風格**：優先使用 `expect` 而非 `assert`，以獲得更可靠的 UI 測試。


## 範例

```python
import re
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="function", autouse=True)
def before_each_after_each(page: Page):
    # 在每個測試之前前往起始 URL。
    page.goto("https://playwright.dev/")

def test_main_navigation(page: Page):
    expect(page).to_have_url("https://playwright.dev/")

def test_has_title(page: Page):
    # 期望標題「包含」一個子字串。
    expect(page).to_have_title(re.compile("Playwright"))

def test_get_started_link(page: Page):
    page.get_by_role("link", name="Get started").click()

    # 期望頁面有一個名為 Installation 的標題。
    expect(page.get_by_role("heading", name="Installation")).to_be_visible()
```

## 測試執行策略

1. **執行**：從終端使用 pytest 命令執行測試。
2. **偵錯失敗**：分析測試失敗並識別根本原因
