# Feedback Classifier

An intelligent feedback classification system that automatically categorizes customer feedback into three categories: complaint, appreciation, or feature request. A form collects the feedback, a Groq-powered AI agent classifies it, and each category gets its own follow-up: an Airtable record, a Slack alert (for complaints and feature requests), and an automatic thank-you / acknowledgment email back to the submitter.

## Trigger

Form trigger ("Feedback Form") with fields: Full Name, Email Address, Contact No, Feedback.

## How it works

13 nodes total.

- "On form submission" captures the feedback and fans out to the AI Agent and a Merge node.
- The AI Agent (Groq `llama-3.1-8b-instant`) outputs exactly one label: `complaint`, `appreciation`, or `feature`.
- Merge (combine by position) joins the label with the original form fields.
- A Switch routes on the label into three branches.
- Each branch creates a record in a category-specific Airtable table.
- Complaint and feature branches also post a summary message to a Slack channel.
- Every branch ends with a Gmail node that sends a tailored acknowledgment email to the submitter's address.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Groq | `groqApi` | LLM classification (llama-3.1-8b-instant) |
| Airtable | `airtableTokenApi` | Storing feedback per category (3 create nodes) |
| Slack | `slackOAuth2Api` | Posting complaint / feature-request alerts |
| Gmail | `gmailOAuth2` | Sending acknowledgment emails to submitters |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_AIRTABLE_BASE_ID` | Complaint / Appreciation / feature nodes | Your Airtable base ID (`app...`) |
| `YOUR_AIRTABLE_TABLE_ID` | Complaint / Appreciation / feature nodes | The table ID (`tbl...`) for each category |
| `YOUR_SLACK_CHANNEL_ID` | Slack and Slack1 nodes | The Slack channel ID to post alerts to |
| `YOUR_GROQ_API_CREDENTIAL` etc. | All credential blocks | Re-select your own Groq / Airtable / Slack / Gmail credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
