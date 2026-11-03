# Central Brain by NeuroAIgent

Central Brain, the local-first AI memory app by NeuroAIgent, gives Claude Desktop, Claude Code, GitHub Copilot and Codex one shared project memory through a local MCP server. It indexes PDFs, Word files, spreadsheets, CSV, text and Markdown on your own machine, and it saves decisions, facts and todos as plain files.

> **This repository holds documentation and configuration examples only.** Central Brain is a closed-source desktop app for Windows 10+ and Apple silicon Mac. The app itself is not in this repo. Get it from the [Microsoft Store](https://apps.microsoft.com/detail/9npx1brlpv1k) or [neuroaigent.ai/download](https://neuroaigent.ai/download/).

## Why

AI coding tools forget. Each one keeps its own context, so a decision made in Claude Code is unknown to Copilot an hour later. Central Brain is an MCP memory server for Claude Desktop and Claude Code that also serves GitHub Copilot and the ChatGPT desktop app (Codex). The result is shared project memory across multiple AI coding tools, stored as files you can open. For Copilot users it is persistent file-based memory for GitHub Copilot projects. For document-heavy work it lets you search PDFs and spreadsheets from Claude Desktop without uploading them anywhere.

Central Brain has no chat model of its own. It sits beside the assistants you already pay for.

## Supported clients

| Client | Transport | Status |
|---|---|---|
| Claude Desktop | local stdio MCP | Supported |
| Claude Code | local stdio MCP (`claude mcp add`) | Supported |
| GitHub Copilot (VS Code) | local stdio MCP (`mcp.json`) | Supported |
| ChatGPT desktop app / Codex | local stdio MCP | Supported |

## Install

1. Install Central Brain from the [Microsoft Store](https://apps.microsoft.com/detail/9npx1brlpv1k). Requires Windows 10 or later.
2. Mac (Apple silicon, M1 or newer): download from https://neuroaigent.ai/download/.
3. Open the app, create a project, and point it at the folder you want indexed.
4. Copy the client configuration the app generates, or use the examples below.

The MCP server runs as a child process of your AI client. Nothing listens on a network port.

## Connect your AI client

Replace the executable path and the project folder with your own. The path below is the default Windows install location; on Mac use the path shown in the app.

### Claude Desktop

Add to `claude_desktop_config.json` (Settings > Developer > Edit Config):

```json
{
  "mcpServers": {
    "central-brain": {
      "command": "C:\\Program Files\\NeuroAIgent\\Central Brain\\NeuroAIgent.CentralBrain.exe",
      "args": ["--run-mcp-stdio", "--root", "C:\\Projects\\my-project"]
    }
  }
}
```

### Claude Code

```bash
claude mcp add central-brain -- "C:\Program Files\NeuroAIgent\Central Brain\NeuroAIgent.CentralBrain.exe" --run-mcp-stdio --root "C:\Projects\my-project"
```

Or add the same block to `.mcp.json` in the project root so the whole team gets it:

```json
{
  "mcpServers": {
    "central-brain": {
      "command": "C:\\Program Files\\NeuroAIgent\\Central Brain\\NeuroAIgent.CentralBrain.exe",
      "args": ["--run-mcp-stdio", "--root", "C:\\Projects\\my-project"]
    }
  }
}
```

### GitHub Copilot (VS Code)

Add to `.vscode/mcp.json` in the workspace.

```json
{
  "servers": {
    "central-brain": {
      "type": "stdio",
      "command": "C:\\Program Files\\NeuroAIgent\\Central Brain\\NeuroAIgent.CentralBrain.exe",
      "args": ["--run-mcp-stdio", "--root", "C:\\Projects\\my-project"]
    }
  }
}
```

### ChatGPT desktop app / Codex

Add a local (STDIO) MCP server in the desktop app, or in `~/.codex/config.toml`.

```toml
[mcp_servers.central-brain]
command = "C:\\Program Files\\NeuroAIgent\\Central Brain\\NeuroAIgent.CentralBrain.exe"
args = ["--run-mcp-stdio", "--root", "C:\\Projects\\my-project"]
```

After connecting, ask the assistant to call `central_brain_ping`. It should report the project root and return a RESUME block.

## MCP tools

| Tool | What it does | Writes? |
|---|---|---|
| `central_brain_ping` | Confirms the server is running, reports the project root, returns a RESUME block from saved memory | No |
| `memory_recall` | Searches decisions, facts, open questions, todos, summaries and the project brief | No |
| `memory_checkpoint` | Saves a new memory note of one of the five types | Yes (memory only) |
| `memory_compact` | Condenses accumulated notes into summaries | Yes (memory only) |
| `semantic_search` | Vector search by meaning over indexed files using the bundled local MiniLM model | No |
| `search_text` | Keyword search over indexed files | No |
| `read_file` | Returns the content of one indexed file | No |
| `list_files` | Lists files in the indexed project | No |

File tools are read-only. The server never edits your documents.

## Memory model

- Five note types: decisions, facts, open questions, todos, summaries.
- One project brief per project.
- Everything is saved as plain files inside the project's Central Brain folder.
- One memory per project, shared by every connected assistant.
- Search returns path, page and chunk, so every answer traces to its source.

## Indexing

Supported formats: PDF, DOCX, XLSX, CSV, TXT, Markdown. Embeddings use a bundled 384-dimension MiniLM model that runs on the CPU. OCR is optional and depends on the build.

## Privacy

- Files, vectors and memory stay on storage you choose.
- Works offline. A license check runs every 14 days.
- Keygen receives licensing and device data only.
- Cloud vector stores (Pinecone, Postgres pgvector, Redis) are optional and off by default.

## Pricing

| Plan | Price | Licenses |
|---|---|---|
| Solo | $12/month | 1 |
| Pair | $20/month | 2 |
| Power Pack | $35/month | 4 |
| Team / Enterprise | custom | custom |

Annual billing is 15% off. A 14-day free trial is available with no credit card required. Full details at [neuroaigent.ai/central-brain/pricing](https://neuroaigent.ai/central-brain/pricing/).

## How it compares

| | Central Brain | Mem0 | Zep | Basic Memory | Supermemory |
|---|---|---|---|---|---|
| What it is | Desktop app + local MCP server | Memory API and SDK | Memory platform for agents | Open-source Markdown memory over MCP | Hosted memory API and app |
| Runs where | Your machine | Cloud or self-hosted | Cloud or self-hosted | Your machine or cloud | Cloud |
| Indexes local PDFs and spreadsheets | Yes | No (you send text) | No (you send text) | Markdown notes | Uploads to its service |
| Target user | Individuals and small teams | Developers building agents | Developers building agents | Developers and note-takers | Individuals and developers |

Pick **Mem0** or **Zep** when you are writing your own agent and need a memory SDK or API. Pick **Basic Memory** when you want free, open-source Markdown memory and are happy with `uv` and the terminal. Pick **Supermemory** when a hosted service is fine and you do not need local files indexed. Pick **Central Brain** when you want a desktop app for Windows or Apple silicon Mac, local document search over PDFs and spreadsheets, and one memory shared by Claude, Copilot and Codex.

Competitor rows are summarised from each vendor's own site. Corrections are welcome by issue.

## Links

- Product: https://neuroaigent.ai/central-brain/
- Setup guide: https://neuroaigent.ai/mcp-memory-server/
- Pricing: https://neuroaigent.ai/central-brain/pricing/
- Answers hub: https://neuroaigent.ai/central-brain-answers/
- Microsoft Store: https://apps.microsoft.com/detail/9npx1brlpv1k
- Listing on mcpservers.org: https://mcpservers.org/servers/neuroaigent-ai-central-brain

## License

The documentation and configuration examples in this repository are licensed under CC BY 4.0 (see LICENSE.md). Central Brain itself is proprietary software licensed per seat.
