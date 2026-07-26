# YouTube Music Video Generation

Turns a single Telegram message into a published YouTube music video. An AI "composer" agent (REBL_X persona) invents a unique song concept, title, description, hashtags and a cinematic thumbnail prompt; kie.ai generates the Suno V5 music tracks and the thumbnail image; ffmpeg assembles a still-image music video; and the result is uploaded to YouTube with the AI-written metadata, with Telegram progress pings at every stage.

## Trigger

Telegram Trigger (any message to the bot) or Manual ("Execute workflow"). Both funnel into a "Task Started" Telegram ping before the agent runs.

## How it works

- The **Prompt Generator** agent (Groq `gpt-oss-120b` in v1; Google Gemini + Postgres chat memory in v2, so past titles/themes are remembered and never repeated) uses a Think tool and a structured output parser to emit `song_prompt`, `song_title`, `song_description`, `thumbnail_prompt` and `hashtags`.
- **Song**: POST to kie.ai `/api/v1/generate` (Suno V5), a `tokencheck` If-node catches credit errors (Telegram alert + Stop and Error), then a Wait/poll loop on `record-info` until `SUCCESS`; the two generated track variants are downloaded, written to `/tmp/song0.mp3` and `/tmp/song1.mp3`, and concatenated into one mp3 with ffmpeg.
- **Thumbnail**: POST to kie.ai `createTask` with `bytedance/seedream-v4-text-to-image` (v1 also keeps an alternative, unwired Imagen 4 Ultra branch), Wait/poll on `recordInfo`, download the 2K 16:9 image to `/tmp/thumbnail.png` and preview it in Telegram.
- **Video**: v1 renders locally with ffmpeg (looped still image + audio, 2048x1152, `libx264` + AAC 320k). v2 instead POSTs image+audio to an external ffmpeg rendering server, polls a status endpoint through a completed / in_progress / failed Switch (10-minute Wait between polls, retries on all HTTP nodes), then downloads the finished mp4.
- **Upload**: YouTube resumable upload in two steps (metadata POST returns an upload URL, then a binary PUT), followed by a cleanup command that deletes all `/tmp` intermediates and a "Task complete" Telegram message (v2 includes the final YouTube link).
- Error branches message Telegram ("Issue in song api" / "Issue in image api") and stop the run.
- v1 has 54 nodes, v2 has 56 nodes (sticky notes included).

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| kie.ai | httpHeaderAuth | Suno V5 music generation, Seedream 4 / Imagen 4 thumbnail tasks |
| Groq | groqApi | Prompt Generator LLM (v1) |
| OpenAI | openAiApi | structured-output auto-fix model (v1) |
| Google Gemini | googlePalmApi | Prompt Generator LLM (v2) |
| Postgres | postgres | agent chat memory for uniqueness (v2) |
| YouTube | youTubeOAuth2Api | resumable video upload |
| Telegram | telegramApi | trigger bot + progress/error notifications |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_TELEGRAM_CHAT_ID` | all Telegram send nodes (`chatId`) | your own Telegram chat ID |
| `YOUR_KIE_TASK_ID` | `GetTaskId` / `GetImageURL1` fallback expressions (v2 only) | remove the fallback or use a real kie.ai task ID for testing |
| `https://your-ffmpeg-server.example.com` | `HTTP Request1/2/3` (v2 only) | your own ffmpeg rendering server URL |
| `YOUR_*_CREDENTIAL` names | `credentials` blocks on kie.ai/Groq/OpenAI/Gemini/Postgres/YouTube/Telegram nodes | re-select your own credentials |

## Versions

- `ai_music_video_publisher_v1.json` — v1 (54 nodes): renders the video locally with ffmpeg; Groq-driven agent without memory; keeps an alternative Imagen 4 thumbnail branch (unwired) plus two stale connections from deleted webhook trigger nodes and two orphaned debug nodes.
- `ai_music_video_publisher_v2.json` — v2 (56 nodes): offloads rendering to an external ffmpeg server with a status-polling Switch loop and retries on every HTTP call; swaps the LLM to Google Gemini with Postgres chat memory so each run's song/title/thumbnail stays unique; completion message includes the YouTube link.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
