# AI Appointment Scheduler

A voice-agent appointment backend for a dental-clinic receptionist bot. Three webhook endpoints implement the tool calls of a VAPI-style voice assistant (check availability, book, cancel): they read the assistant's `toolCalls` payload, work against Google Calendar, log bookings in an n8n Data Table, and answer in the `{"results":[{"toolCallId":…,"result":…}]}` format the voice platform expects. When a requested slot is taken, a Code node computes the free slots for the next 7 working days (9:00-18:00, Mon-Fri, 2-hour booking buffer) and speaks them back as alternatives.

## Trigger

Three Webhook endpoints (POST), placeholder paths after export:
- `appointment-scheduler-checkavailability` — availability check tool
- `appointment-scheduler-createappointment` — booking tool
- `appointment-scheduler-deleteappointment` — cancellation tool

A disabled Chat trigger (`When chat message received`) belongs to an experimental chat-agent variant, see below.

## How it works

30 nodes total (23 enabled, 7 disabled experimental nodes).

- **Check availability**: `checkAvailability` webhook → `Input_data` extracts `tool_id` + `date_time` from the voice platform's tool call → `Get availability in a calendar` queries Google Calendar for the requested hour → `If` the slot is free, `respondAvailable` returns immediately.
- If busy: `Get many events` pulls the full calendar → `Edit Fields` / `Sort` / `Aggregate` normalize the events → `configure slots` (Code) computes free windows for the next 7 working days (business hours, timezone `Asia/Karachi`, 2h minimum lead time) → `respondNotAvailable` returns a human-readable list of alternative slots.
- **Create appointment**: `createAppointment` webhook → `Input_data1` extracts name/email/phone/service/start/end → `Create an event` books a 1-hour Google Calendar event with the caller as attendee → `Respond to Webhook1` confirms → `Insert row` stores the booking (incl. `eventId`) in an n8n Data Table.
- **Cancel appointment**: `deleteAppointment` webhook → `Input_data2` extracts caller identity → `Get row(s)` looks the booking up in the Data Table by phone + email → `Delete an event` removes it from Google Calendar (`onError: continueErrorOutput`) → success path responds and `Delete row(s)` cleans up the table; the error path still answers the voice agent via `Respond to Webhook2`.
- **Disabled experiment**: a chat-triggered AI Agent (`AI Agent1`, OpenRouter model) with four Google Calendar tool nodes (`getManyEvents`, `createEvent`, `checkAvailability1`, `updateEvent`) — an alternative agentic implementation of the same capabilities, kept on the canvas but switched off.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Google Calendar | `googleCalendarOAuth2Api` | Availability lookups, event create/delete (8 calendar nodes) |
| n8n Data Tables | (built-in, no credential) | Booking log: insert / lookup / delete rows |
| OpenRouter | `openRouterApi` | Chat model of the disabled experimental agent |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `your-email@example.com` | `calendar` resource in all 8 Google Calendar / Calendar Tool nodes | Your Google Calendar ID (usually your Gmail address) |
| `YOUR_DATATABLE_ID` | `dataTableId` in `Insert row`, `Get row(s)`, `Delete row(s)` | ID of your own n8n Data Table (columns: full_name, phone_number, email, service, eventId, description, summary) |
| `appointment-scheduler-checkavailability` / `-createappointment` / `-deleteappointment` | Webhook `path` | Your own webhook paths; point your voice platform's tool server URLs at them |
| `YOUR_GOOGLE_CALENDAR_OAUTH2_API_CREDENTIAL`, `YOUR_OPEN_ROUTER_API_CREDENTIAL` | `credentials` blocks | Re-select your own credentials |
| Webhook security | all 3 webhooks | Endpoints ship unauthenticated — add header auth (e.g. verify `x-vapi-secret`) before production use |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
