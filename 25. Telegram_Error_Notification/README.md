# Telegram Error Alerts

Tiny utility workflow that turns any failure across the n8n instance into an instant Telegram alert. Set it as the "Error Workflow" in other workflows' settings and every crash sends a message with the workflow name, HTTP code, error message, and the failing node type. A fallback branch guarantees you still get notified even when the detailed message itself cannot be sent.

## Trigger

Error Trigger (fires automatically when any workflow that registers this as its error workflow fails).

## How it works

- Error Trigger receives the failed execution's payload.
- "Send a text message" posts a detailed Telegram alert: workflow name, `execution.error.httpCode`, `execution.error.message`, and the failing node's type.
- That node runs with `onError: continueErrorOutput` — if composing/sending the detailed alert fails (e.g. the error payload lacks the expected fields), its error output feeds "Send a text message1", which sends a minimal alert with just the workflow name.
- 4 nodes total (including 1 sticky note).

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Telegram Bot | `telegramApi` | Sending the alert messages |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_TELEGRAM_CHAT_ID` | Both Telegram nodes | The chat/user ID that should receive alerts |
| `YOUR_TELEGRAM_API_CREDENTIAL` | Both Telegram nodes | Your Telegram bot credential |

After importing, open the settings of the workflows you want monitored and select this workflow under "Error Workflow".

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
