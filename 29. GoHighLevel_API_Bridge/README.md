# GoHighLevel API Bridge

Bridges a GoHighLevel (GHL) funnel form to an external browser-automation task API. A GHL workflow webhook fires into n8n with contact fields and uploaded document URLs; n8n normalizes the data, downloads the documents, submits everything as a multipart task to the backend service, then polls the task every minute until it finishes and pushes a completed/failed status back into GHL so the client-side automation can continue. Notable details: pure-expression data cleaning (US state name to 2-letter code mapping, multi-format date-of-birth parsing that fails fast on bad input), a retry-with-idempotency-key submission path, and three parallel submit/poll/callback pipelines pointing at different environments of the same API.

## Trigger

Webhook (POST). Placeholder path: `gohighlevel-api-bridge-webhook` — put the resulting production URL into your GHL workflow's webhook action.

## How it works

- The Webhook node receives the GHL POST; `body.customData` carries the form answers (name, DOB, address, portal login, narrative fields, three uploaded-document URLs).
- **Edit Fields** normalizes 16 fields. Two inline JS expressions do the heavy lifting: full state names are mapped to 2-letter codes, and the DOB is parsed from several accepted formats into `MMddyyyy` — both throw on invalid input so bad submissions fail loudly.
- Three HTTP nodes (`file1`, `file`, `file2`) download the uploaded documents from GHL using a private-integration bearer token; **Merge** (3 inputs) + **Aggregate** (with binaries) combine everything into a single item.
- **Edit Fields1** builds the `variables_json` task payload (portal login, narrative, per-bureau sections, mailing address).
- The task is POSTed as `multipart/form-data` (payload + 3 file binaries) to the task API at `/api/external/tasks`. The hardened variant (`HTTP Request4`/`HTTP Request5`) adds `retryOnFail` (5 tries, 5 s apart) and an `Idempotency-Key` header so retries never create duplicate tasks.
- Poll loop: GET the task by id, **Switch** on `running / failed / completed / cancelled / queued`; `running`/`queued` go to a 1-minute **Wait** and re-poll, terminal states exit the loop.
- On `completed` or `failed`, a status callback (status, contact id, phone, name) is POSTed to a GHL inbound webhook trigger, resuming the GHL-side workflow (e.g. notifying the contact).
- 26 nodes total: one webhook entry plus three parallel submit-poll-callback pipelines (raw-IP, tunnel, and a hardened tunnel variant with its callbacks disabled) reflecting dev/prod endpoints of the same backend.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| GoHighLevel (LeadConnector) | — (inline `Authorization: Bearer` header) | Downloading form-uploaded documents; posting status callbacks to an inbound workflow webhook |
| Custom task API (self-hosted) | — (inline `Authorization: Bearer` header) | Creating browser-automation tasks and polling their status |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GHL_PRIVATE_TOKEN` | `Authorization` header in 9 nodes (`file`, `file1`, `file2`, `sucess`, `failed`, `sucess1`, `failed1`, `sucess2`, `failed2`) | Your GHL Private Integration token (`pit-…`) |
| `YOUR_API_TOKEN` | `Authorization` header in `HTTP Request` – `HTTP Request5` | Your task API bearer token |
| `YOUR_SERVER_IP` | URLs in `HTTP Request`, `HTTP Request2` | Host/IP where the task API runs |
| `https://your-backend.example.com` | URLs in `HTTP Request1`, `HTTP Request3`, `HTTP Request4`, `HTTP Request5` | Public URL (e.g. a tunnel) of the task API |
| `YOUR_GHL_LOCATION_ID`, `YOUR_GHL_TRIGGER_ID` | Callback URLs in the `sucess*` / `failed*` nodes | Your GHL location id and inbound webhook trigger id |
| `gohighlevel-api-bridge-webhook` | Webhook trigger path | Any path you like; update the URL in your GHL workflow |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
