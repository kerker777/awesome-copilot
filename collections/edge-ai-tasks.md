# microsoft/edge-ai 的任務

中級到專業使用者和大型程式庫的任務研究員和任務規劃師 - 由 microsoft/edge-ai 提供

**標籤：** architecture, planning, research, tasks, implementation

## Collection 中的項目

| 標題 | 類型 | 說明 |
| ----- | ---- | ----------- |
| [Task Plan Implementation Instructions](../instructions/task-implementation.instructions.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Ftask-implementation.instructions.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode-insiders%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Ftask-implementation.instructions.md) | Instruction | 實作具有漸進式追蹤和變更記錄的任務計畫的 instructions - 由 microsoft/edge-ai 提供 [參見使用方式](#task-plan-implementation-instructions) |
| [Task Planner Instructions](../chatmodes/task-planner.chatmode.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Ftask-planner.chatmode.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode-insiders%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Ftask-planner.chatmode.md) | Chat Mode | 建立可執行實作計畫的任務規劃師 - 由 microsoft/edge-ai 提供 [參見使用方式](#task-planner-instructions) |
| [Task Researcher Instructions](../chatmodes/task-researcher.chatmode.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Ftask-researcher.chatmode.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode-insiders%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Ftask-researcher.chatmode.md) | Chat Mode | 進行全面專案分析的任務研究專家 - 由 microsoft/edge-ai 提供 [參見使用方式](#task-researcher-instructions) |

## Collection 使用方式

### Task Plan Implementation Instructions

繼續使用 `task-planner` 來迭代計畫，直到您確切知道要對程式庫進行什麼操作。

當您準備好實作計畫時，**建立新的對話**並切換到 `Agent` 模式，然後執行新產生的 prompt。

```markdown, implement-fabric-rti-changes.prompt.md
---
mode: agent
title: Implement microsoft fabric realtime intelligence terraform support
---
/implement-fabric-rti-blueprint-modification phaseStop=true
```

此 prompt 還有一個額外的好處，就是將計畫作為 instructions 附加，這有助於在整個對話過程中保持計畫在上下文中。

**專家警告** ->>使用 `phaseStop=false` 讓 Copilot 無需停止地實作整個計畫。此外，您可以使用 `taskStop=true` 讓 Copilot 在每個任務實作後停止，以獲得更精細的控制。

要使用這些產生的 instructions 和 prompts，您需要相應地更新 `settings.json`：

```json
    "chat.instructionsFilesLocations": {
        // Existing instructions folders...
        ".copilot-tracking/plans": true
    },
    "chat.promptFilesLocations": {
        // Existing prompts folders...
        ".copilot-tracking/prompts": true
    },
```

---

### Task Planner Instructions

此外，task-researcher 將提供額外的實作想法，您可以與 GitHub Copilot 一起選擇正確的重點。

```markdown, task-plan.prompt.md
---
mode: task-planner
title: Plan microsoft fabric realtime intelligence terraform support
---
#file: .copilot-tracking/research/*-fabric-rti-blueprint-modification-research.md
Build a plan to support adding fabric rti to this project
```

`task-planner` 將協助您建立實作任務的計畫。它將使用您已完整研究的想法，或在尚未提供時建立新的研究。

`task-planner` 將產生三個 (3) 檔案，這些檔案將被 `task-implementation.instructions.md` 使用。

* `.copilot-tracking/plan/*-plan.instructions.md`

  * 新產生的 instructions 檔案，包含作為階段和任務檢查清單的計畫。
* `.copilot-tracking/details/*-details.md`

  * 實作的詳細資訊，計畫檔案會參考此檔案以獲取具體細節（如果您有大型計畫，這很重要）。
* `.copilot-tracking/prompts/implement-*.prompt.md`

  * 新產生的 prompt 檔案，將建立 `.copilot-tracking/changes/*-changes.md` 檔案並繼續實作變更。

繼續使用 `task-planner` 來迭代計畫，直到您確切知道要對程式庫進行什麼操作。

---

### Task Researcher Instructions

現在您可以迭代研究您的任務！

```markdown, research.prompt.md
---
mode: task-researcher
title: Research microsoft fabric realtime intelligence terraform support
---
Review the microsoft documentation for fabric realtime intelligence
and come up with ideas on how to implement this support into our terraform components.
```

研究會輸出到 .copilot-tracking/research/*-research.md 檔案，並將包含 GHCP 的發現以及在實作期間有用的範例和結構定義。

此外，task-researcher 將提供額外的實作想法，您可以與 GitHub Copilot 一起選擇正確的重點。

---

