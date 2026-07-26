# YouTube Shorts Cross-Poster

A cross-posting pipeline that turns every new YouTube Short into scheduled posts on Pinterest, Facebook and TikTok. YouTube's WebSub (PubSubHubbub) push notifies the workflow the moment a Short is published; the video is downloaded in up to 4K via an Apify actor, analyzed frame-by-frame by Gemini, and an LLM chain writes platform-specific SEO metadata (Pinterest pin + board choice, Facebook reel caption, titles, hashtags) before scheduling everything through a social-publishing API — fully hands-off.

## Trigger

Two Webhooks on the YouTube WebSub feed (placeholder paths after export):
- `youtube-shorts-cross-poster-webhook` (GET) — answers the `hub.challenge` subscription handshake
- `youtube-shorts-cross-poster-webhook-listen-for-videos` (POST) — receives Atom feed pushes for new uploads

## How it works

23 nodes total (20 functional + 3 sticky notes).

- `Webhook` echoes YouTube's `hub.challenge` so the WebSub subscription stays verified; `Webhook (Listen for Videos)` receives new-video notifications.
- `short checker` (If) only lets URLs containing `shorts` through; `rename` extracts title, video ID, channel ID and URL from the Atom payload.
- `Run an Actor and get dataset` calls an Apify video-downloader actor (mp4, up to 2160p, stored in Apify's key-value store) and returns a direct file URL plus duration.
- `HTTP Request` sends the video to the Gemini API (`gemini-2.5-flash`) with a Pinterest-SEO analyst prompt; `Basic LLM Chain` (primary model with a Gemini fallback and a Structured Output Parser) turns the analysis into strict JSON: media type, genre, titles, descriptions, hashtags, the best-matching Pinterest board and Facebook reel copy.
- `Edit Fields` maps the chosen board name to a board ID (5-entry lookup with a default) and collects the video URL.
- Fan-out: `pinterest posting` schedules the video pin (+5 min); `If` splits by duration — under 60 s goes to `Facebook Page` as a reel, otherwise `Facebook Page1` as a normal video (+10 min); `Check TikTok` reads creator-info and `If1` posts immediately (+3 min) when the account can still post, or defers a day (`Post Tiktok1`) when the daily limit is hit.
- All publishing goes through a Late-compatible social-scheduling API (`zernio.com`), so nothing is posted directly from n8n.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Apify | `apifyApi` | Downloading the YouTube Short as an mp4 file |
| Google Gemini | `googlePalmApi` | Video content analysis (HTTP node + fallback chat model) |
| OpenAI-compatible endpoint | `openAiApi` | Primary metadata-writing model of the LLM chain |
| Late-compatible scheduler (zernio.com) | `lateApi` | Scheduling Pinterest, Facebook and TikTok posts (two accounts) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `youtube-shorts-cross-poster-webhook*` | Webhook `path` on both webhooks | Your own paths; register them as the WebSub callback for your channel feed |
| `YOUR_BOARD_ID_1` … `YOUR_BOARD_ID_5` | board map in `Edit Fields` | Your Pinterest board IDs (order matches the 5 board names in the prompt) |
| `YOUR-PINTEREST-ACCOUNT-ID`, `YOUR-FACEBOOK-ACCOUNT-ID`, `YOUR-TIKTOK-ACCOUNT-ID` | `accountId` in the posting nodes and the TikTok creator-info URL | Account IDs from your scheduling-API dashboard |
| `https://zernio.com/api/v1/...` | HTTP posting nodes | Your scheduling API base URL if you use a different Late-compatible provider |
| `your-username` / `~your-upwork-profile-id` / `Your Name` | promo sticky note | Your own profile links (or delete the note) |
| `YOUR_*_CREDENTIAL` | `credentials` blocks | Re-select your own credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
