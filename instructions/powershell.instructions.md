---
applyTo: '**/*.ps1,**/*.psm1'
description: 'PowerShell cmdlet and scripting best practices based on Microsoft guidelines'
---

# PowerShell 指令程式開發指南

本指南提供 PowerShell 特定的說明，協助 GitHub Copilot 產生習慣用法、安全且易於維護的指令碼。內容遵循 Microsoft 的 PowerShell 指令程式開發指南。

## 命名慣例

- **動詞-名詞格式：**
  - 使用已核准的 PowerShell 動詞（Get-Verb）
  - 使用單數名詞
  - 動詞和名詞都使用 PascalCase
  - 避免特殊字元和空格

- **參數名稱：**
  - 使用 PascalCase
  - 選擇清晰、具有描述性的名稱
  - 使用單數形式，除非總是多個
  - 遵循 PowerShell 標準命名

- **變數名稱：**
  - 公開變數使用 PascalCase
  - 私有變數使用 camelCase
  - 避免縮寫
  - 使用有意義的名稱

- **避免使用別名：**
  - 使用完整的指令程式名稱
  - 避免在指令碼中使用別名（例如，使用 Get-ChildItem 而不是 gci）
  - 記錄任何自訂別名
  - 使用完整的參數名稱

### 範例

```powershell
function Get-UserProfile {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Username,

        [Parameter()]
        [ValidateSet('Basic', 'Detailed')]
        [string]$ProfileType = 'Basic'
    )

    process {
        # Logic here
    }
}
```

## 參數設計

- **標準參數：**
  - 使用常見的參數名稱（`Path`、`Name`、`Force`）
  - 遵循內建指令程式慣例
  - 為專業術語使用別名
  - 記錄參數用途

- **參數名稱：**
  - 使用單數形式，除非總是多個
  - 選擇清晰、具有描述性的名稱
  - 遵循 PowerShell 慣例
  - 使用 PascalCase 格式

- **類型選擇：**
  - 使用常見的 .NET 類型
  - 實作適當的驗證
  - 對於有限選項考慮使用 ValidateSet
  - 在可能的地方啟用定位鍵完成

- **切換參數：**
  - 使用 [switch] 用於布林值旗標
  - 避免 $true/$false 參數
  - 省略時預設為 $false
  - 使用清晰的動作名稱

### 範例

```powershell
function Set-ResourceConfiguration {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Name,

        [Parameter()]
        [ValidateSet('Dev', 'Test', 'Prod')]
        [string]$Environment = 'Dev',

        [Parameter()]
        [switch]$Force,

        [Parameter()]
        [ValidateNotNullOrEmpty()]
        [string[]]$Tags
    )

    process {
        # Logic here
    }
}
```

## 管線與輸出

- **管線輸入：**
  - 對於直接物件輸入使用 `ValueFromPipeline`
  - 對於屬性對應使用 `ValueFromPipelineByPropertyName`
  - 實作 Begin/Process/End 區塊來處理管線
  - 記錄管線輸入要求

- **輸出物件：**
  - 傳回豐富的物件，而非格式化文字
  - 使用 PSCustomObject 來處理結構化資料
  - 避免使用 Write-Host 進行資料輸出
  - 啟用下游指令程式處理

- **管線串流：**
  - 一次輸出一個物件
  - 使用 process 區塊進行串流
  - 避免收集大型陣列
  - 啟用立即處理

- **PassThru 模式：**
  - 動作指令程式預設不輸出
  - 實作 `-PassThru` 切換參數來傳回物件
  - 使用 `-PassThru` 傳回修改/建立的物件
  - 使用 verbose/warning 進行狀態更新

### 範例

```powershell
function Update-ResourceStatus {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
        [string]$Name,

        [Parameter(Mandatory)]
        [ValidateSet('Active', 'Inactive', 'Maintenance')]
        [string]$Status,

        [Parameter()]
        [switch]$PassThru
    )

    begin {
        Write-Verbose 'Starting resource status update process'
        $timestamp = Get-Date
    }

    process {
        # Process each resource individually
        Write-Verbose "Processing resource: $Name"

        $resource = [PSCustomObject]@{
            Name        = $Name
            Status      = $Status
            LastUpdated = $timestamp
            UpdatedBy   = $env:USERNAME
        }

        # Only output if PassThru is specified
        if ($PassThru.IsPresent) {
            Write-Output $resource
        }
    }

    end {
        Write-Verbose 'Resource status update process completed'
    }
}
```

## 錯誤處理與安全性

- **ShouldProcess 實作：**
  - 使用 `[CmdletBinding(SupportsShouldProcess = $true)]`
  - 設定適當的 `ConfirmImpact` 層級
  - 對於系統變更呼叫 `$PSCmdlet.ShouldProcess()`
  - 對於額外的確認使用 `ShouldContinue()`

- **訊息串流：**
  - `Write-Verbose` 用於使用 `-Verbose` 的操作詳細資訊
  - `Write-Warning` 用於警告狀態
  - `Write-Error` 用於非終止性錯誤
  - `throw` 用於終止性錯誤
  - 避免 `Write-Host`，除非用於使用者介面文字

- **錯誤處理模式：**
  - 使用 try/catch 區塊進行錯誤管理
  - 設定適當的 ErrorAction 偏好設定
  - 傳回有意義的錯誤訊息
  - 需要時使用 ErrorVariable
  - 包含適當的終止性與非終止性錯誤處理
  - 在具有 `[CmdletBinding()]` 的進階函式中，偏好使用 `$PSCmdlet.WriteError()` 而不是 `Write-Error`
  - 在具有 `[CmdletBinding()]` 的進階函式中，偏好使用 `$PSCmdlet.ThrowTerminatingError()` 而不是 `throw`
  - 構建適當的 ErrorRecord 物件，包括分類、目標和例外詳細資訊

- **非互動式設計：**
  - 透過參數接受輸入
  - 避免在指令碼中使用 `Read-Host`
  - 支援自動化案例
  - 記錄所有必要的輸入

### 範例

```powershell
function Remove-UserAccount {
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'High')]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [ValidateNotNullOrEmpty()]
        [string]$Username,

        [Parameter()]
        [switch]$Force
    )

    begin {
        Write-Verbose 'Starting user account removal process'
        $ErrorActionPreference = 'Stop'
    }

    process {
        try {
            # Validation
            if (-not (Test-UserExists -Username $Username)) {
                $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                    [System.Exception]::new("User account '$Username' not found"),
                    'UserNotFound',
                    [System.Management.Automation.ErrorCategory]::ObjectNotFound,
                    $Username
                )
                $PSCmdlet.WriteError($errorRecord)
                return
            }

            # Confirmation
            $shouldProcessMessage = "Remove user account '$Username'"
            if ($Force -or $PSCmdlet.ShouldProcess($Username, $shouldProcessMessage)) {
                Write-Verbose "Removing user account: $Username"

                # Main operation
                Remove-ADUser -Identity $Username -ErrorAction Stop
                Write-Warning "User account '$Username' has been removed"
            }
        } catch [Microsoft.ActiveDirectory.Management.ADException] {
            $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                $_.Exception,
                'ActiveDirectoryError',
                [System.Management.Automation.ErrorCategory]::NotSpecified,
                $Username
            )
            $PSCmdlet.ThrowTerminatingError($errorRecord)
        } catch {
            $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                $_.Exception,
                'UnexpectedError',
                [System.Management.Automation.ErrorCategory]::NotSpecified,
                $Username
            )
            $PSCmdlet.ThrowTerminatingError($errorRecord)
        }
    }

    end {
        Write-Verbose 'User account removal process completed'
    }
}
```

## 文件與風格

- **註解式說明：** 為任何公開的函式或指令程式包含註解式說明。在函式內，至少添加包含以下內容的 `<# ... #>` 說明註解：
  - `.SYNOPSIS` 簡短說明
  - `.DESCRIPTION` 詳細說明
  - `.EXAMPLE` 包含實務用法的區段
  - `.PARAMETER` 描述
  - `.OUTPUTS` 傳回的輸出類型
  - `.NOTES` 其他資訊

- **一致的格式：**
  - 遵循一致的 PowerShell 風格
  - 使用適當的縮排（建議 4 個空格）
  - 左大括號與陳述式在同一行
  - 右大括號在新行
  - 在管線運算子後使用換行
  - 函式和參數名稱使用 PascalCase
  - 避免不必要的空白

- **管線支援：**
  - 為管線函式實作 Begin/Process/End 區塊
  - 在適當時使用 ValueFromPipeline
  - 支援透過屬性名稱的管線輸入
  - 傳回適當的物件，而非格式化文字

- **避免使用別名：** 使用完整的指令程式名稱和參數
  - 避免在指令碼中使用別名（例如，使用 Get-ChildItem 而不是 gci）；別名可用於互動式 Shell 用法。
  - 使用 `Where-Object` 而不是 `?` 或 `where`
  - 使用 `ForEach-Object` 而不是 `%`
  - 使用 `Get-ChildItem` 而不是 `ls` 或 `dir`

## 完整範例：端到端指令程式模式

```powershell
function New-Resource {
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'Medium')]
    param(
        [Parameter(Mandatory = $true,
            ValueFromPipeline = $true,
            ValueFromPipelineByPropertyName = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$Name,

        [Parameter()]
        [ValidateSet('Development', 'Production')]
        [string]$Environment = 'Development'
    )

    begin {
        Write-Verbose 'Starting resource creation process'
    }

    process {
        try {
            if ($PSCmdlet.ShouldProcess($Name, 'Create new resource')) {
                # Resource creation logic here
                Write-Output ([PSCustomObject]@{
                        Name        = $Name
                        Environment = $Environment
                        Created     = Get-Date
                    })
            }
        } catch {
            $errorRecord = [System.Management.Automation.ErrorRecord]::new(
                $_.Exception,
                'ResourceCreationFailed',
                [System.Management.Automation.ErrorCategory]::NotSpecified,
                $Name
            )
            $PSCmdlet.ThrowTerminatingError($errorRecord)
        }
    }

    end {
        Write-Verbose 'Completed resource creation process'
    }
}
```
