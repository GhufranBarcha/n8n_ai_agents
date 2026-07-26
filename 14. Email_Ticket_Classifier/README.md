# Email Ticket Classifier

AI ticket triage for a multi-client dev team's Gmail inbox. New mail is filtered against an allow-list of client domains, an LLM agent decides whether the email is actionable work, a second agent extracts a fully structured ticket (title, description, goal, current/expected situation, priority, issue type), attachments are re-fetched, uploaded to Google Drive and shared, and the ticket is created in the right client's Notion database — chosen automatically from the sender's domain.

## Trigger

Gmail Trigger, polling every minute (v3 additionally contains two older Gmail trigger copies from previous iterations, one of them disabled).

## How it works

- **clientCheck** (If) keeps only mail whose sender matches an allow-list of client domains / exact addresses.
- **DatabaseID** (Code) maps the sender's domain (or exact address) to that client's Notion database ID via a lookup table of `NOTION_DB_ID_*` values.
- **E-mail Classifier Agent v3** (OpenAI `gpt-4`, boolean structured output with autoFix) decides ticket / no ticket using explicit English and Dutch "actionable work" indicators; the false branch ends in a No-Op.
- **Ticket Details Extractor** (OpenAI `gpt-5.3-chat-latest`, structured output) produces `title`, `description`, `goal`, `currentSituation`, `expectedSituation`, `priority`, `issueType`, `client`, `sprint` — always translated to English.
- The full message is re-fetched with attachments; an If on binary count splits the flow. **fanOut** (Code) emits one item per attachment, each is uploaded to a Google Drive folder, shared as public-read, and its links are collected with Set + Aggregate nodes.
- v3 creates the Notion page in a single fixed database (Sprint/Priority/Issue Type/Description properties plus attachment file URLs). v4 counts the attachments and routes through a Switch (one / two / three / four+) to `Create Ticket in Notion (Met Bijlagen)` variants that write to the per-client database resolved by DatabaseID, with emoji priority labels and a Goal / Current Situation / Expected Situation description block.
- v3 has 42 nodes (three coexisting iterations, the oldest fully disabled); v4 is the cleaned rebuild with 27 nodes.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Gmail | gmailOAuth2 | inbox polling trigger + full-message fetch with attachments |
| OpenAI | openAiApi | classifier and ticket-extractor agents (+ parser autoFix) |
| Groq | groqApi | LLMs of the older, disabled classifier path (v3 only) |
| Notion | notionApi | creating ticket database pages |
| Google Drive | googleDriveOAuth2Api | attachment upload + public share links |

Note: the v4 export carries no credential blocks on its nodes (they were never re-selected in that copy) — attach Gmail, OpenAI, Notion and Drive credentials after importing.

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `NOTION_DB_ID_<CLIENT>` | `DatabaseID` Code node lookup table | your per-client Notion database IDs |
| `YOUR_NOTION_DATABASE_ID` | Notion "Create Ticket" nodes (v3) | your Notion database |
| `YOUR_GOOGLE_FOLDER_ID` | `Upload file` (Google Drive) | Drive folder that stores ticket attachments |
| `your-email@example.com` | `clientCheck` allow-lists and `DatabaseID` exact-match map | real client addresses to allow |
| `YOUR_*_CREDENTIAL` names | `credentials` blocks (v3) | re-select your own credentials |

## Versions

- `email_ticket_classifier_v3.json` — v3 (42 nodes): three iterations in one canvas — the earliest Groq + tempfile.org attachment path (disabled), and the current OpenAI + Google Drive path writing to a single fixed Notion database; simpler ticket schema in the disabled path.
- `email_ticket_classifier_v4.json` — v4, "Final Workflow" (27 nodes): cleaned rebuild; every Notion node targets the per-client database from `DatabaseID`, attachments are routed by count through a Switch, the ticket schema adds goal/current/expected-situation fields and emoji priority levels.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
