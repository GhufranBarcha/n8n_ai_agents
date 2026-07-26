# Lofi YouTube Video Generation

Fully automated production line for multi-hour ambient/lofi YouTube videos: an LLM agent invents the video concept and writes every asset prompt, Suno V5 (via the kie.ai API) generates the music, an image model paints the background artwork, ffmpeg on a remote server (reached over SSH; earlier iterations also experimented with fal.ai's ffmpeg API) assembles a roughly 2-hour video, and the finished file is published to YouTube with an AI-generated, title-branded thumbnail. Telegram is woven through the whole run as the remote control: it can start the pipeline, and it receives progress messages, song/artwork previews, error alerts, and the final YouTube link. Two generations of the build are included, showing the evolution from a sprawling experimental canvas to a lean production workflow that ran on a live channel.

## Trigger
Schedule (Mon/Wed/Fri at 06:00), Telegram message (any message to the bot starts a run), or Execute Workflow (callable as a sub-workflow from another workflow). The v1 file additionally contains a Manual trigger in its legacy sections.

## How it works
- A trigger fires and a "Task Started" note is sent to Telegram.
- The Prompt Generator AI agent (v2: DeepSeek v3.2 via OpenRouter with Gemini as fallback model, Postgres chat memory so past videos are not repeated, structured output parser) produces the video title, Suno music prompt, thumbnail prompt, image-to-video animation prompt (v2), description, and hashtags, and upserts them into an n8n Data Table.
- Music branch: a Suno V5 job is created on kie.ai, a Wait/If loop polls until it completes, both generated songs are downloaded (previews sent to Telegram) and pushed to the video server over SSH.
- Visual branch: an image job (v2: nano-banana-2, v1: Seedream, driven by style-reference images) paints the background; in v2 a grok-imagine image-to-video job then turns it into a seamless 6-second loop, managed by Switch-based polling with explicit Success/Failed/Processing routes and descriptive Stop-and-Error nodes.
- A Merge + Limit pair synchronizes the two branches, then SSH-executed ffmpeg loops the animation over each song and concatenates six song-pairs into a ~2-hour MP4, which is downloaded back into n8n.
- The video is uploaded to YouTube (title, description, tags read from the Data Table), a second image job renders a thumbnail with the title lettered onto the painting, and it is set via the YouTube thumbnails API.
- Telegram receives the published video link and temporary files are deleted from the server.
- File sizes: v1 is 222 nodes (three stacked pipeline generations, most disabled), v2 is 77 nodes (single clean pipeline).

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| kie.ai API | httpHeaderAuth | Suno V5 music jobs, nano-banana-2 / Seedream image jobs, grok-imagine image-to-video |
| OpenRouter | openRouterApi | Primary LLM (deepseek/deepseek-v3.2) for the prompt agent |
| Google Gemini | googlePalmApi | Fallback / earlier primary chat model |
| Telegram Bot | telegramApi | Trigger, progress notifications, audio/image previews, final link |
| YouTube | youTubeOAuth2Api | Video upload and thumbnail set |
| Postgres (hosted) | postgres | Agent chat memory (session table) |
| Remote Linux server | sshPrivateKey | Uploading assets, running ffmpeg, downloading the finished video |
| fal.ai ffmpeg API (v1 only) | inline header (placeholder) | Experimental cloud-side video merge |
| Google Sheets (v1 only, disabled branch) | Google Sheets OAuth | Logging uploaded video URLs |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| YOUR_HTTP_HEADER_AUTH_CREDENTIAL (and other YOUR_*_CREDENTIAL names, ids scrambled) | credentials blocks on most nodes | Re-select your own credentials in the n8n UI |
| YOUR_TELEGRAM_CHAT_ID | chatId of every Telegram node; also the v2 memory sessionKey | Your Telegram chat id |
| YOUR_SESSION_KEY | v1 ChatMemory nodes | Any stable session key for agent memory |
| /home/user/video-workspace | SSH file nodes and remote ffmpeg commands | Working directory on your video server |
| Key YOUR_FALAI_API_KEY | v1 MergeVideo / checkStatus / HTTP Request4 Authorization headers | fal.ai API key (v1 experiment only) |
| YOUR_GOOGLE_DOCUMENT_ID | v1 UpdateYoutubeSheet | Your Google Sheet document id |
| https://www.dropbox.com/scl/fi/YOUR_FILE_ID/... | v1 MergeVideo test body | Your own test video URLs |
| https://example.com/redacted | cachedResultUrl of Data Table / Sheets resource pickers | Re-pick the resource in the UI (Data Table `scrolls-of-dreams` must exist) |
| Regenerated webhookIds | Wait and Telegram nodes | Nothing; n8n handles these on import |

## Versions
- `lofi_video_generator_v1.json` - v1, the original 222-node canvas containing three stacked generations of the pipeline (mostly disabled): the first build (manual/Telegram trigger, still-image background, fal.ai ffmpeg merge experiment, a Deepnote-hosted FFmpeg API, raw YouTube resumable-upload HTTP calls, and a disabled Google Sheets + tags-agent SEO branch), a "Version 1" refactor that moved video assembly to SSH ffmpeg on a remote server, and an enabled "Version 2" section that introduced OpenRouter DeepSeek with Gemini fallback and hosted-Postgres memory.
- `lofi_video_generator_v2.json` - the streamlined 77-node rewrite of that final section on a clean canvas: single pipeline, animated looping background via grok-imagine image-to-video (instead of a still image), nano-banana-2 artwork and thumbnails, robust Switch-based job polling with detailed error stops, retryOnFail on the HTTP/SSH steps, and Merge + Limit branch synchronization; this is the version that ran on a schedule as the live channel pipeline.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
