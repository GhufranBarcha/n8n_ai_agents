# Car Body Shop Estimator Bot

A customer-facing Telegram chatbot for "Carbuddy", an auto body repair shop. The bot ("Marco", speaking Italian) accepts text, voice notes, or damage photos, walks the customer through a strict seven-step damage intake (photos, impact details, internal damage, adjacent parts, vehicle make/model/year), produces a preliminary repair-cost range from the photo analysis, and books an inspection appointment directly into Google Calendar. A separate webhook lane demonstrates Twilio SMS/WhatsApp handling.

## Trigger
Telegram Trigger (message updates) for the main bot. A secondary HTTP Webhook (POST, placeholder path `car-body-shop-estimator-bot-webhook`) receives Twilio inbound-SMS callbacks and answers with a Twilio SMS/WhatsApp message. Three more triggers (callback-query handling, inline-keyboard experiment) ship disabled.

## How it works
- 35 nodes total (25 enabled; several disabled experiments are kept: an allowlist "Authorize & Trace" Code node, an OpenRouter vision call over HTTP, and an inline-keyboard flow).
- Incoming Telegram messages pass an If gate meant for user-ID authorization (its companion allowlist Code node is disabled, so by default everything passes; unauthorized users would get an "Authorization Failed" reply).
- A Switch routes by payload type: voice notes are fetched and transcribed with OpenAI Whisper; plain text passes through a Set node; photos are downloaded and analyzed by GPT-4o (`chatgpt-4o-latest`) as base64.
- A 3-input Merge unifies the branches into one context object (text, image description, caption) and fires a Telegram "typing" chat action while the agent thinks.
- The MainAgent (Claude Sonnet 4.5 via OpenRouter) follows a detailed Italian system prompt: guided damage triage, price-range estimate, then appointment booking; a window-buffer memory (30 messages, keyed by Telegram user ID) keeps the multi-turn intake on track.
- Six Google Calendar tool nodes (`Availability`, `createEvent`, `UpdateEvent`, `GetManyEvents`, `GetEvent`, `DeleteEvent`) let the agent check slots, book, reschedule, and cancel inspections; the reply is sent back to the same Telegram chat.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| Telegram | `telegramApi` | Bot trigger, file downloads, typing action, sending replies |
| OpenAI | `openAiApi` | Whisper voice transcription and GPT-4o damage-photo analysis |
| OpenRouter | `openRouterApi` | Claude Sonnet 4.5 as the agent's chat model |
| Google Calendar | `googleCalendarOAuth2Api` | Six agent tools for availability checks and appointment CRUD |
| Twilio | `twilioApi` | SMS/WhatsApp reply lane behind the webhook |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `your-email@example.com` | `calendar` field in all six Google Calendar tool nodes | The Google Calendar ID (usually your Gmail address) |
| `YOUR_PHONE_NUMBER` | `to` in the Twilio "Send an SMS/MMS/WhatsApp message" node | Destination phone number in E.164 format |
| `YOUR_TELEGRAM_CHAT_ID` | ALLOW_LIST in the disabled "Authorize & Trace" Code node | Your numeric Telegram user ID(s) if you enable the allowlist |
| `car-body-shop-estimator-bot-webhook` | Webhook node `path` | Any webhook path; point your Twilio number's SMS webhook at it |
| `YOUR_*_CREDENTIAL` names | All Telegram / OpenAI / OpenRouter / Calendar / Twilio nodes | Re-select your own credentials (IDs were rewritten too) |
| `Your resource` | `cachedResultName` in calendar tool nodes | Auto-refreshes once you re-pick the calendar from the list |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
