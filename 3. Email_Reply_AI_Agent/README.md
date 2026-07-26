# Email Reply AI Agent

An automated email response system that intelligently processes incoming emails and generates appropriate replies. Every 5 hours it pulls unread Gmail messages, has a Groq-powered agent draft a personalized reply (declining offers, rejecting advertisements, acknowledging order opportunities), sends the reply, logs the exchange to Airtable, and marks the original as read.

## Trigger

Schedule — every 5 hours.

## How it works

8 nodes total.

- Schedule Trigger fires the run every 5 hours.
- "Gmail AII Mails" fetches up to 10 unread messages.
- "Gmail Mail Details" loads the full message for each thread.
- The AI Agent (Groq chat model) writes a reply body following prompt rules: acknowledge receipt, politely decline offers, reject advertisements, defer on order opportunities. The prompt signs as "Your Name" — edit it to yours.
- "Gmail2" sends the reply, extracting the sender's address from the `From` header with a regex.
- The Airtable node logs sender name, from-address, original body, and the generated reply.
- The final Gmail node marks the processed message as read so it is not answered twice.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Groq | `groqApi` | LLM reply drafting |
| Gmail | `gmailOAuth2` | Fetching unread mail, sending replies, marking as read |
| Airtable | `airtableTokenApi` | Logging processed emails and replies |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `Your Name` | AI Agent prompt text | The name replies should be signed with |
| `YOUR_AIRTABLE_BASE_ID` | Airtable node | Your Airtable base ID (`app...`) |
| `YOUR_AIRTABLE_TABLE_ID` | Airtable node | Your log table ID (`tbl...`) |
| `YOUR_GROQ_API_CREDENTIAL` etc. | All credential blocks | Re-select your own Groq / Gmail / Airtable credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
