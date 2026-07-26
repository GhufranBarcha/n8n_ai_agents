# Telegram Personal Assistant

A personal AI assistant that lives in Telegram and understands text, voice notes, and photos. It manages the owner's Google Calendar (create, read, update, delete, availability) and Gmail (send, search, read, reply, delete) through eleven agent tools, powered by Claude Haiku 4.5 via OpenRouter with a 30-message conversation memory. Voice is transcribed with Whisper and photos are described by GPT-4o before reaching the agent, so all three input types land in one unified prompt.

## Trigger
Telegram Trigger (message updates) — the bot reacts to every message sent to it.

## How it works
- 31 nodes: an input-routing front end, one AI agent, and an 11-tool belt (6 Calendar + 5 Gmail).
- Incoming messages hit an If gate intended for user-ID authorization (failing users get an "Authorization Failed" reply; the sticky note tells you to plug your own Telegram numeric user ID into the check).
- A Switch classifies the update: voice notes are downloaded and transcribed by OpenAI Whisper; text passes through a Set node; photos are downloaded and explained by GPT-4o (`chatgpt-4o-latest`, base64 input).
- A 3-input Merge plus an "Input" Set node combine text, image description, and caption into one prompt for the agent.
- The MainAgent (Claude Haiku 4.5 via OpenRouter) runs a personal-assistant system prompt (persona: assistant to "Casonova Brook") with strict plain-text output rules, and a window-buffer memory of 30 messages keyed by the Telegram user ID.
- Calendar tools: `Availability`, `createEvent`, `UpdateEvent`, `GetManyEvents`, `GetEvent`, `DeleteEvent`; Gmail tools: `sendMessage`, `getEmails`, `getEmail`, `replyEmail`, `deleteEmail` — all parameters are filled by the model via `$fromAI`.
- The agent's answer is sent back to the same Telegram chat.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| Telegram | `telegramApi` | Trigger, file downloads, and sending replies |
| OpenAI | `openAiApi` | Whisper voice transcription and GPT-4o image analysis |
| OpenRouter | `openRouterApi` | Claude Haiku 4.5 as the agent's chat model |
| Google Calendar | `googleCalendarOAuth2Api` | Six agent tools for schedule management |
| Gmail | `gmailOAuth2` | Five agent tools for email management |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `your-email@example.com` | `calendar` field in all six Google Calendar tool nodes | The Google Calendar ID (usually your Gmail address) |
| `YOUR_*_CREDENTIAL` names | All Telegram / OpenAI / OpenRouter / Calendar / Gmail nodes | Re-select your own credentials (IDs were rewritten too) |
| `Your resource` | `cachedResultName` in calendar tool nodes | Auto-refreshes once you re-pick the calendar from the list |
| Authorization check | `If` node after the Telegram Trigger | Your own Telegram numeric user ID to restrict access |
| Agent persona | MainAgent system prompt | Rename "Casonova Brook" to yourself and adjust the rules |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
