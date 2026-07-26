# AI Agent with MCP (Model Context Protocol)

A sophisticated AI agent system using the Model Context Protocol (MCP) that exposes YouTube and Gmail as external tools. One workflow runs an MCP server inside n8n that publishes the tools; a second workflow runs a chat agent that consumes those tools through an MCP client over SSE — a clean demonstration of decoupling tool hosting from the agent that uses them.

## Workflows in this folder

| File | Role |
|---|---|
| `4_1_Mcp_Server.json` | MCP Server Trigger (path `mcp-tool`) exposing YouTube and Gmail tool nodes |
| `4_2_MCP_agent.json` | Chat agent (OpenAI `gpt-4o-mini`) that connects to that server via the MCP Client tool |

## Trigger

- 4.1: MCP Server Trigger — SSE endpoint at path `mcp-tool` (served by your n8n instance; this workflow must be active for the client to connect).
- 4.2: Chat trigger ("When chat message received").

## How it works

4.1 MCP server (5 nodes, including 1 sticky note):

- MCP Server Trigger publishes an MCP tool endpoint from your n8n instance.
- YouTube tool node exposes YouTube API operations.
- "Gmail Get Message" exposes reading mail (getAll).
- "Gmail Send Mail" exposes sending mail, with To / Subject / Message wired via `$fromAI()` so the calling model fills them in.

4.2 MCP agent (6 nodes, including 1 sticky note):

- Chat trigger feeds the AI Agent (OpenAI `gpt-4o-mini`) with Simple Memory for conversation context.
- The MCP Client tool connects to the server's SSE endpoint (`http://0.0.0.0:5678/mcp/mcp-tool/sse`) and surfaces the YouTube/Gmail tools to the agent at runtime.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| YouTube | `youTubeOAuth2Api` | YouTube tool operations (4.1) |
| Gmail | `gmailOAuth2` | Reading and sending mail as MCP tools (4.1) |
| OpenAI | `openAiApi` | Chat model gpt-4o-mini for the agent (4.2) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `http://0.0.0.0:5678/mcp/mcp-tool/sse` | 4.2 MCP Client `sseEndpoint` | Your n8n instance URL + the MCP server path (keep in sync with the 4.1 trigger path) |
| `mcp-tool` | 4.1 MCP Server Trigger `path` | Optionally your own path (update the client endpoint to match) |
| `YOUR_GMAIL_OAUTH2_CREDENTIAL` etc. | All credential blocks | Re-select your own YouTube / Gmail / OpenAI credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
