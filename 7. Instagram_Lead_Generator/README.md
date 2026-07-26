# Instagram Lead Generator

A lead generation system that finds potential business leads on Instagram based on an industry and a location. A form collects the search criteria, a Google-search scraping API is queried with a `site:instagram.com` query, and the matching profiles/reels are flattened and saved to Airtable as lead records.

## Trigger

Form trigger ("Instragram Leads Scrapper") with two required fields: the lead type (e.g. "Dentist") and the location (e.g. "Lahore").

## How it works

6 nodes total (including 1 sticky note).

- The form submission provides the lead keyword and location.
- "Search on google" POSTs a search job to a Google-search scraper API with the query `site:instagram.com + <lead> <location>` (100 results per page, 1 page). The endpoint URL is intentionally blank in this export — see placeholders.
- A Wait node pauses 15 seconds so the scrape can finish.
- The Code node flattens each response's `organicResults` into individual lead items (title, URL, description, followers, likes, channel name, type, and more).
- "Lead Airtable" creates one record per lead with the key fields.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Google-search scraper API (e.g. an Apify Google Search Scraper task) | none (URL on HTTP Request) | Running the `site:instagram.com` search |
| Airtable | `airtableTokenApi` | Storing the extracted leads |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| (empty URL) | "Search on google" HTTP Request node | Your scraper endpoint URL (the request body matches Apify's Google Search Scraper input schema); keep any API token in a credential, not the URL |
| `YOUR_AIRTABLE_BASE_ID` | "Lead Airtable" node | Your Airtable base ID (`app...`) |
| `YOUR_AIRTABLE_TABLE_ID` | "Lead Airtable" node | Your leads table ID (`tbl...`) |
| `YOUR_AIRTABLE_TOKEN_API_CREDENTIAL` | Airtable credential block | Your Airtable token credential |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
