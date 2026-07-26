# Twitter Trend Agent

A social media intelligence tool that searches and analyzes Twitter trends based on user queries. Sent a topic via chat, it queries TwitterAPI.io's advanced tweet search with pagination, deduplicates and normalizes the results in a Code node, and stores the top tweets with engagement metrics in Airtable.

## Trigger

Chat trigger ("When chat message received") — the chat input becomes the search query.

## How it works

5 nodes total (including 1 sticky note).

- The chat message is passed as the `query` parameter to TwitterAPI.io's `advanced_search` endpoint (`queryType: Top`).
- The HTTP Request node authenticates with an `X-API-Key` header and paginates via `next_cursor` for up to 3 pages.
- The Code node concatenates all pages, removes duplicate tweets by ID, and formats fields (content, URL, like/retweet/reply/quote/view/bookmark counts, human-readable date).
- "Create a record" writes one Airtable row per tweet.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| TwitterAPI.io | none (header API key on HTTP Request) | Advanced tweet search |
| Airtable | `airtableTokenApi` | Storing the cleaned tweet records |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_API_TOKEN` | HTTP Request `X-API-Key` header | Your TwitterAPI.io API key (better: move it into an n8n credential) |
| `YOUR_AIRTABLE_BASE_ID` | "Create a record" node | Your Airtable base ID (`app...`) |
| `YOUR_AIRTABLE_TABLE_ID` | "Create a record" node | Your table ID (`tbl...`) |
| `YOUR_AIRTABLE_TOKEN_API_CREDENTIAL` | Airtable credential block | Your Airtable token credential |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
