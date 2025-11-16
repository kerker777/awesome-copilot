---
description: '智慧 Git Flow 分支建立器,分析 git status/diff 並根據 nvie Git Flow 分支模型建立適當的分支。'
tools: ['runCommands/runInTerminal', 'runCommands/getTerminalOutput']
mode: 'agent'
---

### 指令

```xml
<instructions>
	<title>Git Flow 分支建立器</title>
	<description>此提示詞使用 git status 和 git diff（或 git diff --cached）分析您目前的 git 變更,然後根據 Git Flow 分支模型智慧地判斷適當的分支類型並建立語意化的分支名稱。</description>
	<note>
		只需執行此提示詞,Copilot 將分析您的變更並為您建立適當的 Git Flow 分支。
	</note>
</instructions>
```

### 工作流程

**遵循以下步驟：**

1. 執行 `git status` 以審查目前的儲存庫狀態和已變更的檔案。
2. 執行 `git diff`（對於未暫存的變更）或 `git diff --cached`（對於已暫存的變更）以分析變更的性質。
3. 使用下方的 Git Flow 分支分析框架分析變更。
4. 根據分析判斷適當的分支類型。
5. 產生遵循 Git Flow 慣例的語意化分支名稱。
6. 自動建立分支並切換到該分支。
7. 提供分析摘要和後續步驟。

### Git Flow 分支分析框架

```xml
<analysis-framework>
	<branch-types>
		<feature>
			<purpose>新功能、增強功能、非關鍵性改進</purpose>
			<branch-from>develop</branch-from>
			<merge-to>develop</merge-to>
			<naming>feature/描述性名稱 或 feature/票證編號-描述</naming>
			<indicators>
				<indicator>正在新增新功能</indicator>
				<indicator>UI/UX 改進</indicator>
				<indicator>新的 API 端點或方法</indicator>
				<indicator>資料庫架構新增（非破壞性）</indicator>
				<indicator>新的設定選項</indicator>
				<indicator>效能改進（非關鍵）</indicator>
			</indicators>
		</feature>

		<release>
			<purpose>發行準備、版本遞增、最終測試</purpose>
			<branch-from>develop</branch-from>
			<merge-to>develop 和 master</merge-to>
			<naming>release-X.Y.Z</naming>
			<indicators>
				<indicator>版本號變更</indicator>
				<indicator>建置設定更新</indicator>
				<indicator>文件最終化</indicator>
				<indicator>發行前的小錯誤修復</indicator>
				<indicator>發行說明更新</indicator>
				<indicator>相依性版本鎖定</indicator>
			</indicators>
		</release>

		<hotfix>
			<purpose>需要立即部署的關鍵生產錯誤修復</purpose>
			<branch-from>master</branch-from>
			<merge-to>develop 和 master</merge-to>
			<naming>hotfix-X.Y.Z 或 hotfix/關鍵問題描述</naming>
			<indicators>
				<indicator>安全性漏洞修復</indicator>
				<indicator>關鍵生產錯誤</indicator>
				<indicator>資料損毀修復</indicator>
				<indicator>服務中斷解決</indicator>
				<indicator>緊急設定變更</indicator>
			</indicators>
		</hotfix>
	</branch-types>
</analysis-framework>
```

### 分支命名慣例

```xml
<naming-conventions>
	<feature-branches>
		<format>feature/[票證編號-]描述性名稱</format>
		<examples>
			<example>feature/user-authentication</example>
			<example>feature/PROJ-123-shopping-cart</example>
			<example>feature/api-rate-limiting</example>
			<example>feature/dashboard-redesign</example>
		</examples>
	</feature-branches>

	<release-branches>
		<format>release-X.Y.Z</format>
		<examples>
			<example>release-1.2.0</example>
			<example>release-2.1.0</example>
			<example>release-1.0.0</example>
		</examples>
	</release-branches>

	<hotfix-branches>
		<format>hotfix-X.Y.Z 或 hotfix/關鍵描述</format>
		<examples>
			<example>hotfix-1.2.1</example>
			<example>hotfix/security-patch</example>
			<example>hotfix/payment-gateway-fix</example>
			<example>hotfix-2.1.1</example>
		</examples>
	</hotfix-branches>
</naming-conventions>
```

### 分析流程

```xml
<analysis-process>
	<step-1>
		<title>變更性質分析</title>
		<description>檢查修改的檔案類型和變更的性質</description>
		<criteria>
			<files-modified>查看副檔名、目錄結構和目的</files-modified>
			<change-scope>判斷變更是新增性、修正性還是準備性</change-scope>
			<urgency-level>評估變更是否處理關鍵問題或為開發性質</urgency-level>
		</criteria>
	</step-1>

	<step-2>
		<title>Git Flow 分類</title>
		<description>將變更對應到適當的 Git Flow 分支類型</description>
		<decision-tree>
			<question>這些是生產問題的關鍵修復嗎？</question>
			<if-yes>考慮 hotfix 分支</if-yes>
			<if-no>
				<question>這些是發行準備變更（版本遞增、最終調整）嗎？</question>
				<if-yes>考慮 release 分支</if-yes>
				<if-no>預設為 feature 分支</if-no>
			</if-no>
		</decision-tree>
	</step-2>

	<step-3>
		<title>分支名稱產生</title>
		<description>建立語意化、描述性的分支名稱</description>
		<guidelines>
			<use-kebab-case>使用小寫加連字號</use-kebab-case>
			<be-descriptive>名稱應清楚指示目的</be-descriptive>
			<include-context>可用時加入票證編號或專案情境</include-context>
			<keep-concise>避免過長的名稱</keep-concise>
		</guidelines>
	</step-3>
</analysis-process>
```

### 邊緣案例和驗證

```xml
<edge-cases>
	<mixed-changes>
		<scenario>變更同時包含功能和錯誤修復</scenario>
		<resolution>優先處理最重要的變更類型或建議拆分為多個分支</resolution>
	</mixed-changes>

	<no-changes>
		<scenario>git status/diff 中未偵測到變更</scenario>
		<resolution>通知使用者並建議先檢查 git status 或進行變更</resolution>
	</no-changes>

	<existing-branch>
		<scenario>已經在 feature/hotfix/release 分支上</scenario>
		<resolution>分析是否需要新分支或當前分支是否適當</resolution>
	</existing-branch>

	<conflicting-names>
		<scenario>建議的分支名稱已存在</scenario>
		<resolution>附加遞增後綴或建議替代名稱</resolution>
	</conflicting-names>
</edge-cases>
```

### 範例

```xml
<examples>
	<example-1>
		<scenario>新增新的使用者註冊 API 端點</scenario>
		<analysis>新功能、新增性變更、非關鍵</analysis>
		<branch-type>feature</branch-type>
		<branch-name>feature/user-registration-api</branch-name>
		<command>git checkout -b feature/user-registration-api develop</command>
	</example-1>

	<example-2>
		<scenario>修復驗證中的關鍵安全性漏洞</scenario>
		<analysis>安全性修復、對生產環境關鍵、需要立即部署</analysis>
		<branch-type>hotfix</branch-type>
		<branch-name>hotfix/auth-security-patch</branch-name>
		<command>git checkout -b hotfix/auth-security-patch master</command>
	</example-2>

	<example-3>
		<scenario>將版本更新至 2.1.0 並最終化發行說明</scenario>
		<analysis>發行準備、版本遞增、文件</analysis>
		<branch-type>release</branch-type>
		<branch-name>release-2.1.0</branch-name>
		<command>git checkout -b release-2.1.0 develop</command>
	</example-3>

	<example-4>
		<scenario>改進資料庫查詢效能並更新快取</scenario>
		<analysis>效能改進、非關鍵增強</analysis>
		<branch-type>feature</branch-type>
		<branch-name>feature/database-performance-optimization</branch-name>
		<command>git checkout -b feature/database-performance-optimization develop</command>
	</example-4>
</examples>
```

### 驗證檢查清單

```xml
<validation>
	<pre-analysis>
		<check>儲存庫處於乾淨狀態（沒有會衝突的未提交變更）</check>
		<check>當前分支是適當的起點（feature/release 用 develop，hotfix 用 master）</check>
		<check>遠端儲存庫是最新的</check>
	</pre-analysis>

	<analysis-quality>
		<check>變更分析涵蓋所有修改的檔案</check>
		<check>分支類型選擇遵循 Git Flow 原則</check>
		<check>分支名稱具有語意且遵循慣例</check>
		<check>考慮並處理邊緣案例</check>
	</analysis-quality>

	<execution-safety>
		<check>目標分支（develop/master）存在且可存取</check>
		<check>建議的分支名稱不與現有分支衝突</check>
		<check>使用者具有建立分支的適當權限</check>
	</execution-safety>
</validation>
```

### 最終執行

```xml
<execution-protocol>
	<analysis-summary>
		<git-status>git status 命令的輸出</git-status>
		<git-diff>git diff 輸出的相關部分</git-diff>
		<change-analysis>變更代表什麼的詳細分析</change-analysis>
		<branch-decision>為何選擇特定分支類型的說明</branch-decision>
	</analysis-summary>

	<branch-creation>
		<command>git checkout -b [分支名稱] [來源分支]</command>
		<confirmation>驗證分支建立和當前分支狀態</confirmation>
		<next-steps>提供後續動作的指引（提交變更、推送分支等）</next-steps>
	</branch-creation>

	<fallback-options>
		<alternative-names>如果主要建議不合適,建議 2-3 個替代分支名稱</alternative-names>
		<manual-override>如果分析似乎不正確,允許使用者指定不同的分支類型</manual-override>
	</fallback-options>
</execution-protocol>
```

### Git Flow 參考

```xml
<gitflow-reference>
	<main-branches>
		<master>可用於生產環境的程式碼,每次提交都是一個發行版本</master>
		<develop>功能的整合分支,最新的開發變更</develop>
	</main-branches>

	<supporting-branches>
		<feature>從 develop 分支,合併回 develop</feature>
		<release>從 develop 分支,合併到 develop 和 master</release>
		<hotfix>從 master 分支,合併到 develop 和 master</hotfix>
	</supporting-branches>

	<merge-strategy>
		<flag>始終使用 --no-ff 旗標以保留分支歷史記錄</flag>
		<tagging>在 master 分支上標記發行版本</tagging>
		<cleanup>成功合併後刪除分支</cleanup>
	</merge-strategy>
</gitflow-reference>
```
