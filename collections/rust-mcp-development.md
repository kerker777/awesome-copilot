# Rust MCP Server 開發

使用官方 rmcp SDK 與 async/await、程序巨集和型別安全實作建置高效能 Model Context Protocol server。

**標籤：** rust, mcp, model-context-protocol, server-development, sdk, tokio, async, macros, rmcp

## Collection 中的項目

| 標題 | 類型 | 說明 |
| ----- | ---- | ----------- |
| [Rust MCP Server Development Best Practices](../instructions/rust-mcp-server.instructions.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Frust-mcp-server.instructions.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode-insiders%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Frust-mcp-server.instructions.md) | Instruction | 使用官方 rmcp SDK 與 async/await 模式建置 Model Context Protocol server 的最佳實務 |
| [Rust Mcp Server Generator](../prompts/rust-mcp-server-generator.prompt.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Frust-mcp-server-generator.prompt.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Frust-mcp-server-generator.prompt.md) | Prompt | 使用官方 rmcp SDK 產生包含工具、prompts、資源和測試的完整 Rust Model Context Protocol server 專案 |
| [Rust MCP Expert](../chatmodes/rust-mcp-expert.chatmode.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Frust-mcp-expert.chatmode.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode-insiders%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Frust-mcp-expert.chatmode.md) | Chat Mode | 使用 rmcp SDK 與 tokio async runtime 進行 Rust MCP server 開發的專業助理 [參見使用方式](#rust-mcp-expert) |

## Collection 使用方式

### Rust MCP Expert

推薦

此 chat mode 提供建置 Rust MCP server 的專業指導。

此 chat mode 適用於：
- 使用 Rust 建立新的 MCP server 專案
- 使用 tokio runtime 實作 async handler
- 使用 rmcp 程序巨集實作工具
- 設定 stdio、SSE 或 HTTP 傳輸
- 偵錯 async Rust 和所有權問題
- 學習使用官方 rmcp SDK 的 Rust MCP 最佳實務
- 使用 Arc 和 RwLock 進行效能最佳化

若要取得最佳結果，建議：
- 使用 instruction 檔案設定 Rust MCP 開發的上下文
- 使用 prompt 產生初始專案結構
- 切換到專業 chat mode 以獲得詳細的實作協助
- 指定您需要哪種傳輸類型
- 提供您需要的工具或功能的詳細資訊
- 說明您是否需要 OAuth 身分驗證

---

*此 collection 包含 3 個精選項目，用於 **Rust MCP Server 開發**。*
