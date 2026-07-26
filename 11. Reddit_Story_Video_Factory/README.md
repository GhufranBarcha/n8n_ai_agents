# Reddit Story Video Factory

A two-workflow content factory that turns viral Reddit posts into narrated "faceless" story videos. v1 is the full YouTube pipeline: it scrapes hot posts from drama/parenting subreddits, rewrites them into a dramatic first-person script with an AI storyteller agent, voices them with Gemini TTS, burns word-by-word karaoke subtitles over looping background gameplay footage with ffmpeg on a remote server, and publishes to YouTube with AI-generated title, description, tags and thumbnail. v2 repurposes the same stories into 9:16 vertical clips and queues them for TikTok/Facebook posting, with Telegram progress notifications throughout.

## Trigger
- **Execute Workflow Trigger** (active entry point in both versions — designed to be kicked off by a parent/controller workflow).
- Disabled alternates kept in the canvas: Manual Trigger, Telegram Trigger, and Schedule Triggers (weekly scrape run; v2 also has an every-8-hours variant). A separate weekly Schedule Trigger feeds the Apify scraping branch.

## How it works
- **Ingestion (shared by both versions):** an Apify Reddit Scraper actor pulls top posts of the week from 8 subreddits (AmItheAsshole, ProRevenge, revengestories, MaliciousCompliance, pettyrevenge, blackholerevenge, regretfulparents, family); a Code node keeps only posts above a minimum score and upserts them into a Google Sheets content queue. A disabled older branch does the same with 7 native Reddit nodes plus an adaptive quality filter (upvote ratio, keyword list, top-quartile comments/score per subreddit).
- **v1 — story generation:** `GetStories` picks the next queued sheet row; `TrueStoryAgent1` (Gemini 2.5 Pro with OpenRouter Gemini fallback and a long storyteller system prompt) rewrites the post into an ~8-minute first-person narration; a `GenderClassifer` agent detects the narrator's gender and stores it in the sheet.
- **v1 — voice-over:** the script is chunked (~10k chars), looped through Gemini 2.5 Flash TTS (voice Orus for male, Zephyr for female), the PCM chunks are merged to WAV with ffmpeg, uploaded to Cloudinary, and the audio URL is written back to the sheet.
- **v1 — subtitles:** AssemblyAI transcribes the audio (polling loop with a Switch on completed/error/processing); Code nodes convert the word timestamps into two ASS subtitle files — a yellow word-highlight "karaoke" style for YouTube and a one-word-at-a-time style for vertical video.
- **v1 — rendering and publish:** subtitles and audio are pushed over SSH to a remote ffmpeg server, which loops a random background gameplay video (Dropbox-hosted), muxes the narration and burns the subtitles; the file is downloaded, uploaded to YouTube, then AI agents generate the SEO title/description, hashtags/tags and a thumbnail prompt rendered via fal.ai z-image (a kie.ai Seedream 4 branch is kept but disabled), the thumbnail is set via the YouTube API, the sheet is marked complete, and server temp files are cleaned up.
- **v2 — short-form repurposer (79 nodes):** picks sheet rows where `tiktok_upload_complete` is false, reuses the already-generated Cloudinary audio (no story/TTS agents), re-transcribes it with AssemblyAI, renders a 1080x1920 9:16 crop with burned subtitles via SSH ffmpeg, overwrites a fixed Google Drive file (`tiktokform.mp4`) and shares it publicly, then appends Title/Description/Caption + the Drive direct-download link with `Publish=Go` to a `buffer_tiktok_posting` sheet that a Buffer-style TikTok/Facebook posting integration consumes; finally flags the row complete.
- Telegram messages report every stage (workflow started, story/audio/subtitles generated, video rendering, upload done); failures in the transcript poll raise Stop-and-Error. v1 has 121 nodes, v2 has 79.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| Apify | `apifyApi` | Reddit Scraper actor + dataset download (both) |
| Google Sheets | `googleSheetsOAuth2Api` | Content queue, story/audio/upload state tracking, TikTok posting buffer (both) |
| Google Gemini (PaLM API) | `googlePalmApi` | Story/classifier/SEO agents and Gemini TTS (both) |
| OpenRouter | `openRouterApi` | Fallback LLM for the AI agents (both) |
| AssemblyAI | `httpHeaderAuth` | Speech-to-text word timestamps for subtitles (both) |
| SSH remote server | `sshPrivateKey` | ffmpeg rendering, file transfer and cleanup (both) |
| Telegram | `telegramApi` | Progress notifications (both) |
| Google Drive | `googleDriveOAuth2Api` | Hosting the final vertical video for TikTok/FB posting (both, main path in v2) |
| Cloudinary | `cloudinaryApi` | Hosting narration audio and thumbnail image (v1) |
| YouTube | `youTubeOAuth2Api` | Video upload, metadata update, thumbnail set (v1) |
| fal.ai | `httpHeaderAuth` / `httpBearerAuth` | z-image thumbnail generation (v1) |
| kie.ai | `httpHeaderAuth` | Seedream 4 thumbnail generation (v1, disabled branch) |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_*_CREDENTIAL` | every node's credential selector | re-select your own credentials of the same type |
| `YOUR_GOOGLE_DOCUMENT_ID` | all Google Sheets nodes (`documentId`) | your content-queue spreadsheet ID (columns per the mapped schemas) |
| `YOUR_GOOGLE_FILE_ID` | Google Drive `Update file1` | ID of a placeholder MP4 in your Drive that gets overwritten each run |
| `YOUR_FILE_ID` / `rlkey=REDACTED` | `Setting` node (Dropbox URLs) | direct-download links to your own background gameplay videos |
| `YOUR_TELEGRAM_CHAT_ID` | all Telegram `SndMsg*` nodes | your Telegram chat ID |
| `Your resource` / blank `cachedResultUrl` | cached UI labels on Sheets/Drive nodes | cosmetic — repopulated when you re-pick the resource |
| webhook IDs (regenerated UUIDs) | Wait / Telegram trigger nodes | none — n8n assigns fresh IDs on import |
| SSH paths `/home/ubuntu/reddit-stories` | SSH command/file nodes | working directory on your own ffmpeg-capable server |

## Versions
- `reddit_story_video_generator_youtube.json` — full long-form YouTube pipeline (121 nodes): scrape → AI story + gender-matched Gemini TTS → AssemblyAI karaoke subtitles → SSH ffmpeg render → YouTube upload with AI title/description/tags and fal.ai thumbnail.
- `reddit_story_repurposer_tiktok_facebook.json` — TikTok/Facebook repurposer (79 nodes): reuses v1's stored audio, re-subtitles it, renders a 9:16 vertical cut, publishes the file to Google Drive and queues it (Title/Caption/Link, `Publish=Go`) in a `buffer_tiktok_posting` sheet for the social posting integration; drops the story/TTS/YouTube/thumbnail stages.

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
