# Islamic History Sleep Stories

A Telegram-driven video factory that turns a raw Urdu history transcript (the bundled sample is a ~55-minute chronicle of the Seljuk Empire) into a finished long-form "sleep story" video for a YouTube channel. A Gemini agent rewrites the transcript into a soothing English bedtime narration, a second agent generates the YouTube title, description and a cinematic thumbnail prompt, and the pipeline then produces TTS audio, an AI thumbnail (ByteDance Seedream v4 via kie.ai) and the final rendered video, uploading everything to Cloudinary. Four saved iterations show the build evolving from an all-inside-n8n ffmpeg pipeline to one that offloads TTS + rendering to an external server.

## Trigger

Telegram Trigger (any message to the bot starts a run) in all four versions. V5 additionally contains a Manual Trigger ("When clicking 'Execute workflow'") that drives an experimental script-splitting branch. Wait nodes use n8n resume-webhook ids (regenerated placeholders in these exports).

## How it works

- Telegram message arrives; the bot immediately replies "Automation Started".
- **NarationGenerator** (AI Agent + Gemini 2.5 Pro, "Boring History Narrative Converter" system prompt) converts the embedded Urdu timestamped transcript into a calm long-form English sleep-story script.
- **Youtube Details** (AI Agent + Gemini 2.5 Pro with a Structured Output Parser) analyzes the script and returns `video_title`, `thumbnail_prompt` and `video_description`, which are sent to Telegram.
- The thumbnail prompt goes to kie.ai (`bytedance/seedream-v4-text-to-image`, 16:9, 2K); a createTask / recordInfo poll loop (If + Wait) downloads the image when ready; a token check stops the run with "Credit Issue" if credits are exhausted.
- Audio/video production differs per version (see Versions): v1/v3 chunk the script (≤10,000 chars) and loop it through the Gemini 2.5 Flash TTS API (voice "Zephyr"), converting PCM→WAV with ffmpeg via Execute Command; v4/v5 POST the script + thumbnail to an external render server that does TTS and video assembly, polled via `/status/{task_id}` and fetched via `/download/{task_id}`.
- The finished audio, thumbnail and MP4 are uploaded to Cloudinary (folder `youtube-automation-data`); temp files in `/tmp` are cleaned up; Telegram receives progress messages at every stage ("Audio Generation Completed", thumbnail photo, Cloudinary link, "Completed Task").
- Node counts: v1 = 43, v3 = 53, v4 = 45, v5 = 49.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Google Gemini (PaLM API) | `googlePalmApi` | Narration + YouTube-metadata agents (Gemini 2.5 Pro), Flash TTS (v1/v3), image-gen test node (v4) |
| kie.ai | `httpHeaderAuth` | Seedream v4 text-to-image thumbnail generation (createTask / recordInfo) |
| Telegram | `telegramApi` | Trigger, progress notifications, thumbnail preview, final links |
| Cloudinary | `cloudinaryApi` | Storing generated audio, thumbnail and final video |
| External FFmpeg/render server | none (plain HTTP) | v3–v5: server-side video assembly (and TTS in v4/v5); URL is a placeholder |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GOOGLE_PALM_API_CREDENTIAL` | Gemini chat/TTS/image nodes | Your Google Gemini (PaLM) credential |
| `YOUR_HTTP_HEADER_AUTH_CREDENTIAL` | GenerateImage / GetImageURL | Header-auth credential holding your kie.ai API key |
| `YOUR_TELEGRAM_API_CREDENTIAL` | Telegram Trigger + all Telegram nodes | Your Telegram bot credential |
| `YOUR_CLOUDINARY_API_CREDENTIAL` | Upload* nodes | Your Cloudinary credential |
| `YOUR_TELEGRAM_CHAT_ID` | `chatId` of every Telegram node | Your own chat id |
| `YOUR_GOOGLE_API_KEY` | `api_key` form field of "HTTP Request - Async Script to Video" (v4/v5) | Your Google API key (was hardcoded; now sanitized) |
| `your-ffmpeg-server.example.com` | HTTP Request1/2/3 and Async Script to Video URLs (v3–v5) | Base URL of your own render/TTS server |
| `https://your-domain.com/api/callback` | kie.ai `callBackUrl` body field | Your callback URL (optional; polling is used) |
| Embedded Urdu transcript | `NarationGenerator` prompt text | Your own episode transcript |
| Credential ids / webhook ids | throughout | Regenerated automatically when you re-select credentials after import |

## Versions

- `sleep_with_islamic_history.json` (v1, 43 nodes) — everything inside n8n: Gemini Flash TTS chunk loop (Code splitter → SplitInBatches → HTTP TTS → PCM→WAV via Execute Command ffmpeg), ffmpeg concat of WAV chunks, ffmpeg still-image + audio → 1080p MP4, then Cloudinary uploads.
- `sleep_with_islamic_history_v3.json` (53 nodes) — adds an external render server: thumbnail + merged WAV are aggregated as binaries and POSTed to `/async_test_quality_video`, with a Switch + 10-min Wait polling loop and `/download` fetch. TTS retry logic added, single-pass PCM concat command, thumbnail generation node disabled (reuses a cached kie.ai task id); the old local-ffmpeg upload chain is kept but disconnected.
- `sleep_with_islamic_history_v4.json` (45 nodes) — drops the in-n8n TTS loop entirely: the English script plus thumbnail go to the server's `/async_script_to_video` endpoint (quality "best", Google API key as a form field — sanitized here), which handles TTS and rendering; also contains a stray unconnected "Generate an image" Gemini test node.
- `sleep_with_islamic_history_v5.json` (49 nodes) — v4 cleaned up (test node removed, fresh webhook ids) plus an experimental manually-triggered branch: Set `parts=4` → NarationGenerator1 (Gemini agent with a "dynamic splitting" prompt + structured parser) that splits the narration into N sequential scripts for multi-part videos.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
