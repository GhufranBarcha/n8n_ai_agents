# LinkedIn Post & PDF Newsletter Generator

A one-click AI-news publishing pipeline: it scrapes the transcript of a weekly AI-updates YouTube video, has an LLM write a clean, anti-hype LinkedIn post from it, then expands the same material into a multi-page A4 PDF newsletter (cover page, one deep-dive card per tool, CTA page) rendered from generated HTML through a self-hosted Gotenberg service. The post text and the PDF are produced in a single run, ready to publish as a LinkedIn document post.

## Trigger

Manual (`When clicking 'Execute workflow'`).

## How it works

18 nodes total (16 active, 2 disabled: a YouTube API `Get a video` node and the final `Create a post` LinkedIn node).

- `Run an Actor and get dataset` calls the Apify YouTube Scraper for the target video with `downloadSubtitles: true` (plaintext, English) — the video URL is set in the actor's JSON body.
- `fields` (Set) extracts title, description and the plaintext transcript.
- `AI Agent` (Gemini 3 Pro via OpenRouter) writes the LinkedIn post: headline, intro, 5-7 update blocks with feature bullets and links, outro, hashtags — with strict tone rules (no hype words, no markdown asterisks).
- `Edit Fields` passes the post downstream, where three branches build the newsletter in parallel:
  - `MainBody` (agent, Gemini 3 Flash) turns transcript + post into per-tool HTML `update-card` pages, using the LinkedIn post for ordering and the transcript for technical depth;
  - `CoverPage` (LLM chain) generates a unique cover title/subtitle, and `CoverPage1` stamps today's date via a regex replace;
  - `CTA` (Set) holds the static closing page.
- `Merge` (3 inputs, combine by position) + `Merge1` assemble the full HTML document (inline CSS, A4 print layout) and keep the plain-text post in a `linkedln_post` field.
- `html` (Convert to File) writes `index.html`, and `gostenbergAPI` POSTs it to a Gotenberg Chromium endpoint (`/pdf/forms/chromium/convert/html`) with zero margins and CSS page size, returning the finished PDF binary.
- `Create a post` (disabled) is the hand-off point for actually publishing to LinkedIn.
- A large sticky note documents the full PDF-generator prompt and HTML template used to design the newsletter.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Apify | `apifyApi` | YouTube Scraper actor (transcript download) |
| OpenRouter | `openRouterApi` | Both LLM steps (post writer + newsletter writer/cover) |
| Gotenberg (self-hosted) | none (plain HTTP) | HTML to PDF conversion |
| YouTube | `youTubeOAuth2Api` | Disabled `Get a video` node |
| LinkedIn | (select on node) | Disabled `Create a post` node |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `https://your-n8n-instance.com/pdf/forms/chromium/convert/html` | `gostenbergAPI` HTTP node | URL of your own Gotenberg instance |
| `https://youtu.be/8grIT-xK50M` | `Run an Actor and get dataset` custom body (`startUrls`) | The YouTube video you want to summarize |
| `Your Name` | CTA page, cover template, sticky-note template | Your own byline for the post and PDF |
| `YOUR_OPEN_ROUTER_API_CREDENTIAL`, `YOUR_APIFY_API_CREDENTIAL`, `YOUR_YOU_TUBE_OAUTH2_API_CREDENTIAL` | `credentials` blocks | Re-select your own credentials |

Note: the `fonts.googleapis.com/css2?family=Inter:wght@400;...` URL in the CSS is the public Google Fonts API (the `@` is font-weight syntax, not credentials).

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
