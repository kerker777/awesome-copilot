---
applyTo: '**/*.Tests.ps1'
description: 'PowerShell Pester testing best practices based on Pester v5 conventions'
---

# PowerShell Pester v5 測試指南

本指南提供使用 PowerShell Pester v5 模組建立自動化測試的 PowerShell 特定指示。請遵循 [powershell.instructions.md](./powershell.instructions.md) 中的 PowerShell cmdlet 開發指南，以獲得一般 PowerShell 腳本最佳實踐。

## 檔案命名和結構

- **檔案慣例：** 使用 `*.Tests.ps1` 命名模式
- **放置位置：** 將測試檔案放在被測試程式碼旁邊或專用測試目錄中
- **匯入模式：** 使用 `BeforeAll { . $PSScriptRoot/FunctionName.ps1 }` 來匯入被測試的函式
- **無直接程式碼：** 將所有程式碼放在 Pester 區塊內（`BeforeAll`、`Describe`、`Context`、`It` 等）

## 測試結構階層

```powershell
BeforeAll { # 匯入被測試的函式 }
Describe 'FunctionName' {
    Context 'When condition' {
        BeforeAll { # 設置 context }
        It 'Should behavior' { # 個別測試 }
        AfterAll { # 清理 context }
    }
}
```

## 核心關鍵字

- **`Describe`**: 頂層分組，通常以被測試的函式命名
- **`Context`**: Describe 內的子分組，用於特定情境
- **`It`**: 個別測試案例，使用描述性名稱
- **`Should`**: 用於測試驗證的斷言關鍵字
- **`BeforeAll/AfterAll`**: 每個區塊執行一次設置/拆卸
- **`BeforeEach/AfterEach`**: 每個測試前/後執行設置/拆卸

## 設置和拆卸

- **`BeforeAll`**: 在包含區塊開始時執行一次，用於昂貴的操作
- **`BeforeEach`**: 在區塊中的每個 `It` 之前執行，用於特定測試的設置
- **`AfterEach`**: 在每個 `It` 之後執行，即使測試失敗也保證執行
- **`AfterAll`**: 在區塊結束時執行一次，用於清理
- **變數作用域**: `BeforeAll` 變數可供子區塊使用（唯讀），`BeforeEach/It/AfterEach` 共享相同作用域

## 斷言（Should）

- **基本比較**: `-Be`、`-BeExactly`、`-Not -Be`
- **集合**: `-Contain`、`-BeIn`、`-HaveCount`
- **數值**: `-BeGreaterThan`、`-BeLessThan`、`-BeGreaterOrEqual`
- **字串**: `-Match`、`-Like`、`-BeNullOrEmpty`
- **型別**: `-BeOfType`、`-BeTrue`、`-BeFalse`
- **檔案**: `-Exist`、`-FileContentMatch`
- **例外**: `-Throw`、`-Not -Throw`

## 模擬（Mocking）

- **`Mock CommandName { ScriptBlock }`**: 替換命令行為
- **`-ParameterFilter`**: 僅在參數符合條件時模擬
- **`-Verifiable`**: 將模擬標記為需要驗證
- **`Should -Invoke`**: 驗證模擬被呼叫的特定次數
- **`Should -InvokeVerifiable`**: 驗證所有可驗證的模擬都已被呼叫
- **作用域**: 模擬預設為包含區塊的作用域

```powershell
Mock Get-Service { @{ Status = 'Running' } } -ParameterFilter { $Name -eq 'TestService' }
Should -Invoke Get-Service -Exactly 1 -ParameterFilter { $Name -eq 'TestService' }
```

## 測試案例（資料驅動測試）

使用 `-TestCases` 或 `-ForEach` 進行參數化測試：

```powershell
It 'Should return <Expected> for <Input>' -TestCases @(
    @{ Input = 'value1'; Expected = 'result1' }
    @{ Input = 'value2'; Expected = 'result2' }
) {
    Get-Function $Input | Should -Be $Expected
}
```

## 資料驅動測試

- **`-ForEach`**: 可在 `Describe`、`Context` 和 `It` 上使用，用於從資料生成多個測試
- **`-TestCases`**: `It` 區塊上的 `-ForEach` 別名（向後相容）
- **雜湊表資料**: 每個項目定義測試中可用的變數（例如 `@{ Name = 'value'; Expected = 'result' }`）
- **陣列資料**: 使用 `$_` 變數表示當前項目
- **範本**: 在測試名稱中使用 `<variablename>` 進行動態擴展

```powershell
# 雜湊表方法
It 'Returns <Expected> for <Name>' -ForEach @(
    @{ Name = 'test1'; Expected = 'result1' }
    @{ Name = 'test2'; Expected = 'result2' }
) { Get-Function $Name | Should -Be $Expected }

# 陣列方法
It 'Contains <_>' -ForEach 'item1', 'item2' { Get-Collection | Should -Contain $_ }
```

## 標籤（Tags）

- **可用於**: `Describe`、`Context` 和 `It` 區塊
- **篩選**: 使用 `Invoke-Pester` 的 `-TagFilter` 和 `-ExcludeTagFilter`
- **萬用字元**: 標籤支援 `-like` 萬用字元以實現靈活篩選

```powershell
Describe 'Function' -Tag 'Unit' {
    It 'Should work' -Tag 'Fast', 'Stable' { }
    It 'Should be slow' -Tag 'Slow', 'Integration' { }
}

# 僅執行快速單元測試
Invoke-Pester -TagFilter 'Unit' -ExcludeTagFilter 'Slow'
```

## 跳過（Skip）

- **`-Skip`**: 可在 `Describe`、`Context` 和 `It` 上使用以跳過測試
- **條件式**: 使用 `-Skip:$condition` 進行動態跳過
- **執行時跳過**: 在測試執行期間使用 `Set-ItResult -Skipped`（設置/拆卸仍會執行）

```powershell
It 'Should work on Windows' -Skip:(-not $IsWindows) { }
Context 'Integration tests' -Skip { }
```

## 錯誤處理

- **失敗時繼續**: 使用 `Should.ErrorAction = 'Continue'` 收集多個失敗
- **關鍵時停止**: 對於前置條件使用 `-ErrorAction Stop`
- **測試例外**: 使用 `{ Code } | Should -Throw` 進行例外測試

## 最佳實踐

- **描述性名稱**: 使用清楚解釋行為的測試描述
- **AAA 模式**: 安排（設置）、執行（操作）、斷言（驗證）
- **隔離測試**: 每個測試應該是獨立的
- **避免別名**: 使用完整 cmdlet 名稱（`Where-Object` 而非 `?`）
- **單一職責**: 盡可能每個測試一個斷言
- **測試檔案組織**: 在 Context 區塊中分組相關測試。Context 區塊可以巢狀。

## 範例測試模式

```powershell
BeforeAll {
    . $PSScriptRoot/Get-UserInfo.ps1
}

Describe 'Get-UserInfo' {
    Context 'When user exists' {
        BeforeAll {
            Mock Get-ADUser { @{ Name = 'TestUser'; Enabled = $true } }
        }

        It 'Should return user object' {
            $result = Get-UserInfo -Username 'TestUser'
            $result | Should -Not -BeNullOrEmpty
            $result.Name | Should -Be 'TestUser'
        }

        It 'Should call Get-ADUser once' {
            Get-UserInfo -Username 'TestUser'
            Should -Invoke Get-ADUser -Exactly 1
        }
    }

    Context 'When user does not exist' {
        BeforeAll {
            Mock Get-ADUser { throw "User not found" }
        }

        It 'Should throw exception' {
            { Get-UserInfo -Username 'NonExistent' } | Should -Throw "*not found*"
        }
    }
}
```

## 設定

設定在測試檔案**外部**定義，在呼叫 `Invoke-Pester` 時控制執行行為。

```powershell
# 建立設定（Pester 5.2+）
$config = New-PesterConfiguration
$config.Run.Path = './Tests'
$config.Output.Verbosity = 'Detailed'
$config.TestResult.Enabled = $true
$config.TestResult.OutputFormat = 'NUnitXml'
$config.Should.ErrorAction = 'Continue'
Invoke-Pester -Configuration $config
```

**關鍵區段**: Run（Path、Exit）、Filter（Tag、ExcludeTag）、Output（Verbosity）、TestResult（Enabled、OutputFormat）、CodeCoverage（Enabled、Path）、Should（ErrorAction）、Debug
