# UGC AI Avatar Video Generator

Generates UGC-style "talking avatar" marketing videos end to end: an AI-generated (or uploaded) creator photo is analyzed, split into 8-second scene scripts by an LLM agent, rendered into per-scene clips, and stitched into one final video with ffmpeg. The folder holds two related workflows: a large multi-variant build that experiments with several generation pipelines (Veo 3 image-to-video, Nano Banana Pro image generation, ElevenLabs TTS + Kling AI Avatar lip-sync) behind six web forms, and a Telegram bot integration that productionizes the pipeline with Airtable state tracking and chat commands to re-run failed scenes.

## Trigger

- **Main workflow:** six n8n Form triggers, one per pipeline variant (upload-your-image forms and describe-your-creator forms).
- **Telegram workflow:** Telegram Trigger (bot messages / `/c` and `/r` commands) which replies with a native Telegram custom form (Send-and-Wait), plus an equivalent n8n Form trigger.

## How it works

- The main workflow (188 nodes) bundles six variants of the same pipeline, labeled by sticky notes ("First and Last Frame", "Nano Banana Pro - Version 1/2", "Kling AI Avatar").
- Upload variants: the form image is hosted on tempfile.org (24 h expiry), analyzed by GPT-4o-mini vision, and a Claude Sonnet 4.5 agent (OpenRouter) writes 8-second scene scripts with a byte-identical visual description per scene (structured output parser enforced).
- Nano Banana variants: an agent crafts a photorealistic "authentic UGC" image prompt (anti-AI-detection rules, negative prompt), kie.ai `nano-banana-pro` generates the creator photo, and Switch nodes route success / failed / processing poll states into Stop-and-Error nodes or Wait-and-retry loops.
- Each scene is turned into a clip via kie.ai Veo 3 fast (`REFERENCE_2_VIDEO` or `FIRST_AND_LAST_FRAMES_2_VIDEO`), polled in a Wait loop, and the clip URL is stored in an n8n Data Table.
- The Kling variant instead formats the script into TTS chunks under 14 seconds, generates speech with ElevenLabs multilingual v2 (voice picked in the form), then lip-syncs the photo to each audio chunk with `kling/ai-avatar-pro`.
- All variants finish on ffmpeg-api.com: create a directory, request upload URLs, download and re-upload every clip, then run an ffmpeg concat filter and download the merged video.
- The Telegram workflow (60 nodes) runs one pipeline (Nano Banana Pro image, Claude Opus 4.5 scene agent, Veo 3 clips) and writes every scene to Airtable (`unique_id`, `videoNumber`, `script_object`, `image_url`); the generated photo, each finished clip, and the final merged video are sent back to the chat, with chat-action indicators while jobs run.
- Telegram commands resume from Airtable state: `/c <session-id>` regenerates all scenes still marked `video_generated = False`, `/r <video-id>` redoes a single scene; the final Airtable record gets the `complete_video` URL.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| kie.ai | Header Auth (`httpHeaderAuth`) in the Telegram workflow; inline `Bearer YOUR_API_TOKEN` header in the main workflow | Veo 3 video jobs, Nano Banana Pro images, ElevenLabs TTS, Kling AI Avatar lip-sync |
| ffmpeg-api.com | Header Auth (`httpHeaderAuth`) | upload URLs and ffmpeg concat merge of the scene clips |
| OpenAI | OpenAI API (`openAiApi`) | GPT-4o-mini vision analysis of the reference image |
| OpenRouter | OpenRouter API (`openRouterApi`) | Claude Sonnet 4.5 / Claude Opus 4.5 chat models behind the agents |
| Telegram | Telegram API (`telegramApi`) | bot trigger, Send-and-Wait form, photo/clip/video delivery (Telegram workflow) |
| Airtable | Airtable Token (`airtableTokenApi`) | per-scene state tracking and resume commands (Telegram workflow) |
| tempfile.org | none | temporary hosting of uploaded reference images (main workflow) |
| n8n Data Table | built-in | stores generated clip URLs per scene (main workflow) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_API_TOKEN` | `Authorization` header on the 20 kie.ai HTTP Request nodes in the main workflow (Generate UGC Image*, Poll * Status, ElevenLabs TTS, Kling AI Avatar, HTTP Request*) | your kie.ai API key (better: move it into a Header Auth credential) |
| `YOUR_TELEGRAM_API_CREDENTIAL` | `credentials.telegramApi` on all 11 Telegram nodes | your Telegram bot credential |
| `YOUR_TELEGRAM_CHAT_ID` | fallback chat id inside `chatId` expressions on 9 Telegram nodes | your numeric Telegram chat id |
| `YOUR_AIRTABLE_BASE_ID` / `YOUR_AIRTABLE_TABLE_ID` | the 5 Airtable nodes (create / search / update) | your base and table; columns: `unique_id`, `videoNumber`, `script`, `character_description`, `script_object`, `image_url`, `video_url`, `complete_video`, `video_generated` (True/False) |
| Data Table selection | `Insert row*` / `Save to Data Table` nodes (main workflow) | your own n8n Data Table with columns `prompt`, `videoUrl`, `refImg`, `scene` |
| `https://example.com/redacted` / `Your resource` | cached picker labels on Airtable and Data Table nodes | cosmetic; re-pick the resource in the UI |

## Workflows in this folder

- `ugc_ai_avatar_video_generator.json` — the 188-node multi-variant build: six form-triggered pipelines covering image upload vs. Nano Banana Pro generation, Veo 3 reference vs. first/last-frame video, and the ElevenLabs + Kling AI Avatar lip-sync route, all ending in the ffmpeg merge.
- `ugc_ai_avatar_telegram_bot.json` — the 60-node Telegram front-end: one hardened pipeline with Airtable scene tracking, per-clip delivery to the chat, and `/c` / `/r` resume commands.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
