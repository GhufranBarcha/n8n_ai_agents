# n8n Webhook Chatbot

A simple yet powerful chatbot implementation exposed as a REST endpoint. The workflow receives a message via a POST webhook, generates a reply with Groq's LLaMA 3 70B model through an AI Agent, and returns the answer in the HTTP response — a minimal pattern for wiring any frontend or external app to an LLM.

## Trigger

Webhook — `POST` on the placeholder path `chatbot-webhook-webhook` (rename after import), with `responseMode: responseNode`.

## How it works

4 nodes total.

- The Webhook node accepts a POST request whose JSON body carries the user message (`body.message`).
- The AI Agent passes the message as the prompt to the connected chat model.
- The Groq Chat Model (`llama3-70b-8192`) generates the reply.
- Respond to Webhook returns the agent output as the HTTP response, enabling synchronous request/response chat.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Groq | `groqApi` | LLM response generation (llama3-70b-8192) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `chatbot-webhook-webhook` | Webhook node `path` | Your own webhook path |
| `YOUR_GROQ_API_CREDENTIAL` | Groq Chat Model node | Your Groq API credential |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
