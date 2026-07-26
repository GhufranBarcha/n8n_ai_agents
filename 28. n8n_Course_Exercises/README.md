# n8n Course Exercises (Level 1 & 2)

The complete exercise set from n8n's official Level 1 (beginner) and Level 2 (intermediate) courses, kept on one canvas as four independent sections: scheduled ERP order processing with conditional routing, Airtable + HTTP merging and sorting, regional report generation with Gmail and Discord notifications plus batch looping, and country-data enrichment upserted back into Airtable. Useful as a compact reference for core n8n primitives: IF, Merge (combine and enrich modes), Sort, Loop Over Items, Convert to File, Date & Time, and Code.

## Trigger
Schedule (Mondays 09:00) drives the Level 1 section; a Manual trigger drives the Level 2 warm-up; the reporting and enrichment sections have no trigger of their own and were executed manually node-by-node during the course (hence the "not reachable from any trigger" validator warnings).

## How it works
- 30 nodes total, four sections.
- **Level 1 (scheduled):** Schedule Trigger → HTTP Request fetches orders from the course ERP webhook (header auth + `unique_id` header) → IF `orderStatus=processing` AND `employeeName=Mario` → matching orders go to Edit Fields → Airtable create (left disabled) and to a Code aggregator; non-matching orders go to Code, which sums `totalBooked`/`bookedSum` → Discord posts the weekly totals plus the course unique ID.
- **Level 2 warm-up (manual):** Manual Trigger → Airtable "Search records" and HTTP Request1 (level 2 ERP) in parallel → Merge combines both streams on `customerID` → Sort by `orderPrice` descending.
- **Reporting section:** Airtable1 + HTTP Request2 → Merge1 → Sort1 → If1 splits on `region=Americas`; true: Convert to File writes one JSON per order → Gmail emails the "Regional Sales Report - Americas" with attachment → Discord2 confirms; false: Loop Over Items (batches of 5) → Edit Fields1 → Date & Time formats `customerSince` → Convert to File1 names the sheet `file_high_orders`/`file_low_orders` via `$runIndex` → Discord1 announces each file → loops back.
- **Enrichment section:** Airtable2 + HTTP Request3 (`restcountries.com/v3.1/all`) → Merge2 enriches each customer with `region`/`subregion` by matching `customerCountry` to country names → Airtable3 upserts the enriched rows back into the base.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| n8n course ERP | httpHeaderAuth | authenticated GETs of course order/customer data |
| Airtable | airtableTokenApi | searching, creating, and upserting records |
| Discord | discordWebhookApi | posting totals, file names, and confirmations |
| Gmail | gmailOAuth2 | emailing the regional sales report |
| REST Countries API | none | public country metadata for enrichment |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_COURSE_UNIQUE_ID` | `unique_id` header in HTTP Request / HTTP Request1 / HTTP Request2 and in the Discord1/Discord2 message texts | the unique ID issued to you by the n8n course |
| `your-email@example.com` | Gmail → sendTo | the report recipient address |
| `YOUR_AIRTABLE_BASE_ID` / `YOUR_AIRTABLE_TABLE_ID` | all Airtable nodes | your own base and table IDs |
| `YOUR_*_CREDENTIAL` names | HTTP header auth, Airtable, Discord, Gmail nodes | re-select your own credentials |
| `https://example.com/redacted` | `cachedResultUrl` on Airtable resource pickers | refreshes automatically once you re-pick base/table |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
