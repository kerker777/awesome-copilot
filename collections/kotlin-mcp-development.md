# Kotlin MCP Server 開發

使用官方 io.modelcontextprotocol:kotlin-sdk 函式庫建置 Model Context Protocol (MCP) server 的完整工具組。包含最佳實務的 instructions、產生 server 的 prompt 以及提供指導的專業 chat mode。

**標籤：** kotlin, mcp, model-context-protocol, kotlin-multiplatform, server-development, ktor

## Collection 中的項目

| 標題 | 類型 | 說明 |
| ----- | ---- | ----------- |
| [Kotlin MCP Server Development Guidelines](../instructions/kotlin-mcp-server.instructions.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Fkotlin-mcp-server.instructions.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/instructions?url=vscode-insiders%3Achat-instructions%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Finstructions%2Fkotlin-mcp-server.instructions.md) | Instruction | 使用官方 io.modelcontextprotocol:kotlin-sdk 函式庫建置 Model Context Protocol (MCP) server 的最佳實務與模式。 |
| [Kotlin MCP Server Project Generator](../prompts/kotlin-mcp-server-generator.prompt.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Fkotlin-mcp-server-generator.prompt.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fprompts%2Fkotlin-mcp-server-generator.prompt.md) | Prompt | 使用官方 io.modelcontextprotocol:kotlin-sdk 函式庫產生具有適當結構、相依套件和實作的完整 Kotlin MCP server 專案。 |
| [Kotlin MCP Server Development Expert](../chatmodes/kotlin-mcp-expert.chatmode.md)<br />[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Fkotlin-mcp-expert.chatmode.md)<br />[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/chatmode?url=vscode-insiders%3Achat-mode%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fgithub%2Fawesome-copilot%2Fmain%2Fchatmodes%2Fkotlin-mcp-expert.chatmode.md) | Chat Mode | 使用官方 SDK 建置 Model Context Protocol (MCP) server 的 Kotlin 專業助理。[參見使用方式](#kotlin-mcp-server-development-expert) |

## Collection 使用方式

### Kotlin MCP Server Development Expert

推薦

此 chat mode 提供建置 Kotlin MCP server 的專業指導。

此 chat mode 適用於：
- 使用 Kotlin 建立新的 MCP server 專案
- 使用 coroutine 和 kotlinx.serialization 實作型別安全的工具
- 使用 Ktor 設定 stdio 或 SSE 傳輸
- 偵錯 coroutine 模式和 JSON schema 問題
- 學習使用官方 SDK 的 Kotlin MCP 最佳實務
- 建置多平台 MCP server（JVM、Wasm、iOS）

若要取得最佳結果，建議：
- 使用 instruction 檔案設定 Kotlin MCP 開發的上下文
- 使用 prompt 產生包含 Gradle 的初始專案結構
- 切換到專業 chat mode 以獲得詳細的實作協助
- 指定您是否需要 stdio 或 SSE/HTTP 傳輸
- 提供您需要的工具或功能的詳細資訊
- 說明您是否需要多平台支援或特定目標

---

*此 collection 包含 3 個精選項目，用於 **Kotlin MCP Server 開發**。*
