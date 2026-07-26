# Email to Notion Ticket Triage (WIP)

A small email-triage helper in progress: Gmail is polled every minute and an LLM-backed Text Classifier labels each incoming email "ticket" or "no ticket". A Notion "create database page" node sits on the canvas ready to log tickets into a Notion database, but its input is not yet wired to the classifier — exported honestly as the work-in-progress scaffold it is.

## Trigger
Gmail Trigger — polls the inbox every minute (full message format, spam/trash excluded, no attachment download).

## How it works
- 4 nodes total.
- **Gmail Trigger** polls for new mail every minute.
- **Text Classifier** sends the email snippet (`{{ $json.snippet }}`) to the LLM with two categories: `ticket` (email contains or requests a ticket) and `no ticket` (everything else).
- **OpenRouter Chat Model** (`google/gemini-2.5-flash-preview-09-2025`) is the classifier's language model.
- **Create a database page** (Notion) creates a page in a Notion database via its title property; the classifier's outputs currently dead-end, so this node still needs to be connected to the `ticket` branch.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| Gmail | gmailOAuth2 | polling trigger on the inbox |
| OpenRouter | openRouterApi | Gemini model behind the Text Classifier |
| Notion | notionApi | creating database pages for tickets |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_NOTION_DATABASE_ID` | Create a database page → databaseId | the ID of your Notion database |
| `YOUR_GMAIL_OAUTH2_CREDENTIAL` / `YOUR_OPEN_ROUTER_API_CREDENTIAL` / `YOUR_NOTION_API_CREDENTIAL` | respective nodes | your own credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
