# Cold Email Outreach

A complete cold-email machine for a bookkeeping-services business. It scrapes local-business leads from Google Maps via Apify (with contact enrichment), stores businesses and decision-maker contacts in Airtable, scrapes each lead's website with Firecrawl for personalization context, then drip-sends AI-written, individually personalized HTML emails through Gmail on a weekday schedule — pacing sends minutes apart and logging every email back to Airtable. Two LLM stages split the job cleanly: one agent writes the plain-text email from a detailed persona prompt, a second converts it to email-client-safe HTML through a structured output parser with auto-fix.

## Trigger

Three entry points: an Execute Workflow Trigger (passthrough) starts the lead-collection pipeline from a parent workflow; a Schedule Trigger (cron `0 9 * * 1-5`, timezone-aware) runs the daily send pipeline; a disabled Manual Trigger drives an optional Airtable-to-Google-Sheets backup branch.

## How it works

- **Lead collection:** the Apify node runs the Google Maps Scraper actor (several service-business search strings for one US state, `scrapeContacts` + leads-enrichment enabled) → short Wait → a Set node normalizes ~35 fields (title, phones, emails, socials, location, categories, enrichment records).
- Airtable upserts: the **Businesses** table is keyed on `placeId`; **Split Out** explodes `leadsEnrichment` and upserts each decision-maker into the **Contacts** table keyed on `PersonID`.
- **Website context:** a batch loop walks the businesses; an IF gate (has website, not yet scraped) sends each to Firecrawl `/scrape`, and the resulting markdown + page metadata are written back onto the Airtable business record. Error outputs feed back into the loop so one bad site never stalls the run.
- **Daily send pipeline:** fetch Contacts where `Email Status = 'Not Sent'` → look up the linked Business by `Place ID` → a Set node merges contact + business + scraped-site data → IF (email exists) → Limit 200 → batch loop.
- **AI Agent** (OpenRouter chat model) writes a plain-text cold email from a long system prompt encoding the sender persona, per-industry pain points, review-count-based angles, pricing talk-track, and strict formatting rules.
- **AI Agent1** + Structured Output Parser (`autoFix` enabled) converts that text into a `{ html_email }` object with inline-styled, Gmail-safe HTML.
- Gmail sends the email; the Airtable contact is marked `Sent` (storing both text and HTML bodies); a 2-minute Wait paces the loop before the next contact (a 90-second Wait branch exists as an alternative pacing path).
- Legacy branches kept for reference: a static HTML template blast loop (sends a fixed designed template to a test inbox) and a disabled Airtable-to-Sheets contact backup. 39 nodes total.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Apify | `apifyApi` | Google Maps scraping + lead enrichment |
| Airtable | `airtableTokenApi` | Businesses/Contacts CRM (upsert, search, update) |
| Firecrawl | `firecrawlApi` | Scraping lead websites to markdown |
| OpenRouter | `openRouterApi` | LLM for email writing and HTML formatting |
| Gmail | `gmailOAuth2` | Sending outreach and test emails |
| Google Sheets | `googleSheetsOAuth2Api` | Optional contact backup (disabled branch) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_AIRTABLE_BASE_ID`, `YOUR_AIRTABLE_TABLE_ID` | All Airtable nodes | Your base id and the Businesses / Contacts table ids |
| `YOUR_GOOGLE_DOCUMENT_ID` | `Append or update row in sheet` | Spreadsheet id for the backup sheet |
| `your-email@example.com` | Gmail `sendTo` fields and `mailto:` links in the email templates | Your test inbox / public contact address |
| `Your Name`, `your-linkedin-profile`, `your-upwork-profile-id`, `your-username`, `wa.me/YOUR_WHATSAPP_NUMBER`, `+00 000 000 0000`, `ExampleCo` | Email templates (`Edit Fields3`, `Edit Fields4`) and both AI system prompts | Your own sender identity, profile URLs, phone number, and testimonial reference |
| Search strings / `locationQuery` | `Run Google Maps Extractor` custom body | The niches and region you want to prospect |
| Credentials named `YOUR_*_CREDENTIAL` | All service nodes | Re-select your own credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
