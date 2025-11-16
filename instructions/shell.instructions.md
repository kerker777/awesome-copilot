---
description: 'bash、sh、zsh 和其他 shell 的 Shell 腳本最佳實踐和慣例'
applyTo: '**/*.sh'
---

# Shell 腳本指南

撰寫乾淨、安全且可維護的 bash、sh、zsh 和其他 shell 腳本的指引。

## 一般原則

- 生成乾淨、簡單和簡潔的程式碼
- 確保腳本易於閱讀和理解
- 在有助於理解腳本如何運作的地方新增註解
- 生成簡潔簡單的 echo 輸出以提供執行狀態
- 避免不必要的 echo 輸出和過多的日誌記錄
- 在可用時使用 shellcheck 進行靜態分析
- 假設腳本用於自動化和測試而非生產系統，除非另有說明
- 優先使用安全擴展：雙引號變數引用（`"$var"`），使用 `${var}` 以提高清晰度，並避免使用 `eval`
- 在可移植性要求允許時使用現代 Bash 功能（`[[ ]]`、`local`、陣列）；僅在需要時回退到 POSIX 結構
- 選擇可靠的解析器處理結構化資料，而非臨時文字處理

## 錯誤處理和安全性

- 始終啟用 `set -euo pipefail` 以在錯誤時快速失敗、捕獲未設定的變數並顯示管道失敗
- 在執行前驗證所有必需的參數
- 提供具有上下文的清晰錯誤訊息
- 使用 `trap` 在腳本終止時清理臨時資源或處理意外退出
- 使用 `readonly`（或 `declare -r`）宣告不可變值以防止意外重新賦值
- 使用 `mktemp` 安全地建立臨時檔案或目錄，並確保在清理處理程序中刪除它們

## 腳本結構

- 以清晰的 shebang 開始：`#!/bin/bash`，除非另有說明
- 包含解釋腳本目的的標頭註解
- 在頂部定義所有變數的預設值
- 為可重用的程式碼區塊使用函式
- 建立可重用的函式，而非重複類似的程式碼區塊
- 保持主執行流程乾淨且易讀

## 處理 JSON 和 YAML

- 優先使用專用解析器（JSON 使用 `jq`，YAML 使用 `yq`—或透過 `yq` 轉換為 JSON 後使用 `jq`）而非使用 `grep`、`awk` 或 shell 字串分割進行臨時文字處理
- 當 `jq`/`yq` 不可用或不適合時，選擇環境中下一個最可靠的解析器，並明確說明如何安全使用
- 驗證所需欄位是否存在，並明確處理缺失/無效的資料路徑（例如透過檢查 `jq` 退出狀態或使用 `// empty`）
- 引用 jq/yq 過濾器以防止 shell 擴展，並在需要純字串時優先使用 `--raw-output`
- 將解析器錯誤視為致命錯誤：與 `set -euo pipefail` 結合使用或在使用結果之前測試命令成功
- 在腳本頂部記錄解析器依賴項，如果需要但未安裝 `jq`/`yq`（或替代工具），則以有用的訊息快速失敗

```bash
#!/bin/bash

# ============================================================================
# 腳本描述在此
# ============================================================================

set -euo pipefail

cleanup() {
    # 根據需要刪除臨時資源或執行其他清理步驟
    if [[ -n "${TEMP_DIR:-}" && -d "$TEMP_DIR" ]]; then
        rm -rf "$TEMP_DIR"
    fi
}

trap cleanup EXIT

# 預設值
RESOURCE_GROUP=""
REQUIRED_PARAM=""
OPTIONAL_PARAM="default-value"
readonly SCRIPT_NAME="$(basename "$0")"

TEMP_DIR=""

# 函式
usage() {
    echo "Usage: $SCRIPT_NAME [OPTIONS]"
    echo "Options:"
    echo "  -g, --resource-group   資源群組（必需）"
    echo "  -h, --help            顯示此說明"
    exit 0
}

validate_requirements() {
    if [[ -z "$RESOURCE_GROUP" ]]; then
        echo "Error: 需要資源群組"
        exit 1
    fi
}

main() {
    validate_requirements

    TEMP_DIR="$(mktemp -d)"
    if [[ ! -d "$TEMP_DIR" ]]; then
        echo "Error: 無法建立臨時目錄" >&2
        exit 1
    fi

    echo "============================================================================"
    echo "腳本執行開始"
    echo "============================================================================"

    # 主邏輯在此

    echo "============================================================================"
    echo "腳本執行完成"
    echo "============================================================================"
}

# 解析參數
while [[ $# -gt 0 ]]; do
    case $1 in
        -g|--resource-group)
            RESOURCE_GROUP="$2"
            shift 2
            ;;
        -h|--help)
            usage
            ;;
        *)
            echo "未知選項: $1"
            exit 1
            ;;
    esac
done

# 執行主函式
main "$@"

```
