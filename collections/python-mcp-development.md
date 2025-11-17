# Python MCP Server 開發

使用官方 SDK 與 FastMCP 建置 Model Context Protocol (MCP) server 的完整工具組。包含最佳實務的 instructions、產生 server 的 prompt 以及提供指導的專業 chat mode。

**標籤：** python, mcp, model-context-protocol, fastmcp, server-development

## Collection 中的項目

| 標題 | 類型 | 說明 |
| ----- | ---- | ----------- |
| [Python MCP Server Development](../instructions/python-mcp-server.instructions.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Fpython-mcp-server.instructions.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode-insiders%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Fpython-mcp-server.instructions.md) | Instruction | 使用 Python SDK 建置 Model Context Protocol (MCP) server 的 instructions |
| [Generate Python MCP Server](../prompts/python-mcp-server-generator.prompt.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Fpython-mcp-server-generator.prompt.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Fpython-mcp-server-generator.prompt.md) | Prompt | 產生包含工具、資源和適當組態的完整 Python MCP server 專案 |
| [Python MCP Server Expert](../chatmodes/python-mcp-expert.chatmode.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Fpython-mcp-expert.chatmode.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode-insiders%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Fpython-mcp-expert.chatmode.md) | Chat Mode | 開發 Python Model Context Protocol (MCP) server 的專業助理 [參見使用方式](#python-mcp-server-expert) |

## Collection 使用方式

### Python MCP Server Expert

推薦

此 chat mode 提供使用 FastMCP 建置 Python MCP server 的專業指導。

此 chat mode 適用於：
- 使用 Python 建立新的 MCP server 專案
- 使用 Pydantic 模型和結構化輸出實作型別化工具
- 設定 stdio 或可串流 HTTP 傳輸
- 偵錯型別提示和 schema 驗證問題
- 學習使用 FastMCP 的 Python MCP 最佳實務
- 最佳化 server 效能和資源管理

若要取得最佳結果，建議：
- 使用 instruction 檔案設定 Python/FastMCP 開發的上下文
- 使用 prompt 產生包含 uv 的初始專案結構
- 切換到專業 chat mode 以獲得詳細的實作協助
- 指定您是否需要 stdio 或 HTTP 傳輸
- 提供您需要的工具或功能的詳細資訊
- 說明您是否需要結構化輸出、取樣或引出

---

*此 collection 包含 3 個精選項目，用於 **Python MCP Server 開發**。*
