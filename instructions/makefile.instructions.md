---
description: "Best practices for authoring GNU Make Makefiles"
applyTo: "**/Makefile, **/makefile, **/*.mk, **/GNUmakefile"
---

# Makefile 開發說明

編寫乾淨、易於維護且具有可攜性的 GNU Make Makefile 的指導。這些說明基於 [GNU Make 手冊](https://www.gnu.org/software/make/manual/)。

## 通用原則

- 編寫清晰且易於維護的 Makefile，遵循 GNU Make 約定
- 使用具有描述性的目標名稱，清楚指示其用途
- 將預設目標（首個目標）設定為最常見的建置操作
- 編寫規則和配方時，優先考慮可讀性而非簡潔性
- 為複雜的規則、變數或非明顯的行為添加註解

## 命名約定

- 將 Makefile 命名為 `Makefile`（推薦用於清晰可見）或 `makefile`
- 只在 GNU Make 特定功能與其他 make 實作不相容時才使用 `GNUmakefile`
- 為目錄檔案清單使用標準變數名稱：`objects`、`OBJECTS`、`objs`、`OBJS`、`obj` 或 `OBJ`
- 對內建變數名稱使用大寫（例如 `CC`、`CFLAGS`、`LDFLAGS`）
- 使用描述性的目標名稱，反映其動作（例如 `clean`、`install`、`test`）

## 檔案結構

- 在 Makefile 中將預設目標（主要建置目標）作為第一個規則放置
- 邏輯上將相關目標組合在一起
- 在規則之前，在 Makefile 的頂部定義變數
- 使用 `.PHONY` 聲明不代表檔案的目標
- 結構化 Makefile：變數、規則、然後是虛擬目標

```makefile
# Variables
CC = gcc
CFLAGS = -Wall -g
objects = main.o utils.o

# Default goal
all: program

# Rules
program: $(objects)
	$(CC) -o program $(objects)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Phony targets
.PHONY: clean all
clean:
	rm -f program $(objects)
```

## 變數和替換

- 使用變數來避免重複並改進可維護性
- 使用 `:=`（簡單展開）定義變數以進行即時評估，`=` 用於遞迴展開
- 使用 `?=` 設定可被覆蓋的預設值
- 使用 `+=` 追加到現有變數
- 參考變數時使用 `$(VARIABLE)` 而不是 `$VARIABLE`（除非單個字元）
- 在配方中使用自動變數（`$@`、`$<`、`$^`、`$?`、`$*`）以使規則更加通用

```makefile
# Simple expansion (evaluates immediately)
CC := gcc

# Recursive expansion (evaluates when used)
CFLAGS = -Wall $(EXTRA_FLAGS)

# Conditional assignment
PREFIX ?= /usr/local

# Append to variable
CFLAGS += -g
```

## 規則和先決條件

- 清楚分離目標、先決條件和配方
- 為標準編譯使用隱含規則（例如 `.c` 到 `.o`）
- 以邏輯順序列出先決條件（普通先決條件在順序相關之前）
- 對不應該觸發重新建置的目錄和依賴項使用順序相關先決條件（`|` 後面）
- 包括所有實際依賴項以確保正確的重新建置
- 避免目標之間的圓形依賴項
- 記住順序相關先決條件會從自動變數（如 `$^`）中省略，所以需要時要明確參考它們

下面的範例展示了一個模式規則，將物件編譯到 `obj/` 目錄。該目錄本身被列為順序相關先決條件，所以它在編譯前建立，但當其時間戳改變時不會強制重新編譯。

```makefile
# Normal prerequisites
program: main.o utils.o
	$(CC) -o $@ $^

# Order-only prerequisites (directory creation)
obj/%.o: %.c | obj
	$(CC) $(CFLAGS) -c $< -o $@

obj:
	mkdir -p obj
```

## 配方和命令

- 每條配方行都必須以 **tab 字元**（不是空格）開始，除非 `.RECIPEPREFIX` 已改變
- 在適當時使用 `@` 前綴來抑制命令回顯
- 使用 `-` 前綴來忽略特定命令的錯誤（謹慎使用）
- 在同一行中使用 `&&` 或 `;` 組合相關命令，當它們必須一起執行時
- 保持配方可讀性；使用反斜線延續符號將長命令分解為多行
- 在需要時在配方中使用 shell 條件和迴圈

```makefile
# Silent command
clean:
	@echo "Cleaning up..."
	@rm -f $(objects)

# Ignore errors
.PHONY: clean-all
clean-all:
	-rm -rf build/
	-rm -rf dist/

# Multi-line recipe with proper continuation
install: program
	install -d $(PREFIX)/bin && \
		install -m 755 program $(PREFIX)/bin
```

## 虛擬目標

- 始終使用 `.PHONY` 聲明虛擬目標以避免與同名檔案衝突
- 為 `clean`、`install`、`test`、`all` 等動作使用虛擬目標
- 將虛擬目標聲明放在其規則定義附近或 Makefile 結尾

```makefile
.PHONY: all clean test install

all: program

clean:
	rm -f program $(objects)

test: program
	./run-tests.sh

install: program
	install -m 755 program $(PREFIX)/bin
```

## 模式規則和隱含規則

- 為通用轉換使用模式規則（`%.o: %.c`）
- 在適當時利用內建隱含規則（GNU Make 知道如何編譯 `.c` 到 `.o`）
- 覆蓋隱含規則變數（如 `CC`、`CFLAGS`）而不是重寫規則
- 只在內建規則不足時才定義自訂模式規則

```makefile
# Use built-in implicit rules by setting variables
CC = gcc
CFLAGS = -Wall -O2

# Custom pattern rule for special cases
%.pdf: %.md
	pandoc $< -o $@
```

## 拆分長行

- 使用反斜線-換行（`\`）拆分長行以提高可讀性
- 注意在非配方上下文中，反斜線-換行會轉換為單個空格
- 在配方中，反斜線-換行會保留 shell 的行延續
- 避免在反斜線後面有尾隨空格

### 拆分而不添加空格

如果需要在不添加空格的情況下拆分行，可以使用特殊技術：插入 `$ `（美元-空格），後面跟著反斜線-換行。`$ ` 參考一個單個空格名稱的變數，該變數不存在並展開為無，有效地連接行而不插入空格。

```makefile
# Concatenate strings without adding whitespace
# The following creates the value "oneword"
var := one$ \
       word

# This is equivalent to:
# var := oneword
```

```makefile
# Variable definition split across lines
sources = main.c \
          utils.c \
          parser.c \
          handler.c

# Recipe with long command
build: $(objects)
	$(CC) -o program $(objects) \
	      $(LDFLAGS) \
	      -lm -lpthread
```

## 包含其他 Makefile

- 使用 `include` 指令在 Makefile 間共享通用定義
- 使用 `-include`（或 `sinclude`）來包含可選 Makefile 而不出錯
- 將 `include` 指令放在可能影響被包含檔案的變數定義之後
- 為共享變數、模式規則或通用目標使用 `include`

```makefile
# Include common settings
include config.mk

# Include optional local configuration
-include local.mk
```

## 條件指令

- 為平台或配置特定規則使用條件指令（`ifeq`、`ifneq`、`ifdef`、`ifndef`）
- 在 Makefile 層級放置條件，不在配方中（在配方中使用 shell 條件）
- 保持條件簡單且文檔完善

```makefile
# Platform-specific settings
ifeq ($(OS),Windows_NT)
    EXE_EXT = .exe
else
    EXE_EXT =
endif

program: main.o
	$(CC) -o program$(EXE_EXT) main.o
```

## 自動先決條件

- 自動生成標頭依賴項而非手動維護
- 使用編譯器旗標如 `-MMD` 和 `-MP` 來生成包含依賴項的 `.d` 檔案
- 使用 `-include $(deps)` 包含生成的依賴項檔案，以避免在其不存在時出錯

```makefile
objects = main.o utils.o
deps = $(objects:.o=.d)

# Include dependency files
-include $(deps)

# Compile with automatic dependency generation
%.o: %.c
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@
```

## 錯誤處理和除錯

- 使用 `$(error text)` 或 `$(warning text)` 函數進行建置時診斷
- 使用 `make -n`（乾式執行）測試 Makefile 以查看命令而不執行
- 使用 `make -p` 列印規則和變數的資料庫以進行除錯
- 在 Makefile 開始時驗證所需的變數和工具

```makefile
# Check for required tools
ifeq ($(shell which gcc),)
    $(error "gcc is not installed or not in PATH")
endif

# Validate required variables
ifndef VERSION
    $(error VERSION is not defined)
endif
```

## 清潔目標

- 始終提供 `clean` 目標以移除生成的檔案
- 將 `clean` 聲明為虛擬以避免與名為「clean」的檔案衝突
- 對 `rm` 命令使用 `-` 前綴以在檔案不存在時忽略錯誤
- 考慮分離的 `clean`（移除目錄檔案）和 `distclean`（移除所有生成檔案）目標

```makefile
.PHONY: clean distclean

clean:
	-rm -f $(objects)
	-rm -f $(deps)

distclean: clean
	-rm -f program config.mk
```

## 可攜性考慮

- 如果需要可攜到其他 make 實作，避免使用 GNU Make 特定功能
- 使用標準 shell 命令（優先使用 POSIX shell 結構）
- 使用 `make -B` 測試以強制重新建置所有目標
- 記錄任何平台特定要求或所用的 GNU Make 擴展

## 效能最佳化

- 對不需要遞迴展開的變數使用 `:=`（更快）
- 避免不必要地使用 `$(shell ...)`，這會建立子程序
- 有效地排序先決條件（最常更改的檔案放在最後）
- 通過確保目標不衝突來安全地使用平行建置（`make -j`）

## 文件和註解

- 添加標頭註解解釋 Makefile 的目的
- 記錄非明顯的變數設定及其效果
- 在註解中包括使用範例或目標
- 為複雜規則或平台特定的解決方案添加內聯註解

```makefile
# Makefile for building the example application
#
# Usage:
#   make          - Build the program
#   make clean    - Remove generated files
#   make install  - Install to $(PREFIX)
#
# Variables:
#   CC       - C compiler (default: gcc)
#   PREFIX   - Installation prefix (default: /usr/local)

# Compiler and flags
CC ?= gcc
CFLAGS = -Wall -Wextra -O2

# Installation directory
PREFIX ?= /usr/local
```

## 特殊目標

- 對非檔案目標使用 `.PHONY`
- 使用 `.PRECIOUS` 保留中間檔案
- 使用 `.INTERMEDIATE` 將檔案標記為中間（自動刪除）
- 使用 `.SECONDARY` 防止中間檔案被刪除
- 使用 `.DELETE_ON_ERROR` 在配方失敗時移除目標
- 使用 `.SILENT` 對所有配方抑制回顯（謹慎使用）

```makefile
# Don't delete intermediate files
.SECONDARY:

# Delete targets if recipe fails
.DELETE_ON_ERROR:

# Preserve specific files
.PRECIOUS: %.o
```

## 常見模式

### 標準專案結構

```makefile
CC = gcc
CFLAGS = -Wall -O2
objects = main.o utils.o parser.o

.PHONY: all clean install

all: program

program: $(objects)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	-rm -f program $(objects)

install: program
	install -d $(PREFIX)/bin
	install -m 755 program $(PREFIX)/bin
```

### 管理多個程式

```makefile
programs = prog1 prog2 prog3

.PHONY: all clean

all: $(programs)

prog1: prog1.o common.o
	$(CC) -o $@ $^

prog2: prog2.o common.o
	$(CC) -o $@ $^

prog3: prog3.o
	$(CC) -o $@ $^

clean:
	-rm -f $(programs) *.o
```

## 要避免的反模式

- 不要用空格而是 tab 開始配方行
- 避免當可以用萬用字符或函數生成時硬編碼檔案清單
- 不要使用 `$(shell ls ...)` 取得檔案清單（改用 `$(wildcard ...)`）
- 避免在配方中使用複雜的 shell 指令碼（移到單獨的指令碼檔案）
- 不要忘記將虛擬目標聲明為 `.PHONY`
- 避免目標之間的圓形依賴項
- 除非絕對必要，否則不要使用遞迴 make（`$(MAKE) -C subdir`）
