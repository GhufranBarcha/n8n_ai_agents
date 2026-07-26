# n8n AI Agentic Workflows

This repository contains all my AI agentic workflows and automation created in n8n. Each workflow demonstrates different use cases and capabilities of AI agents in automation scenarios.

## 1. Feedback Classifier

An intelligent feedback classification system that automatically categorizes customer feedback into three distinct categories: complaints, appreciation, or feature requests. This workflow uses a form trigger to collect user feedback and employs an AI agent to classify the feedback type automatically.

**Key Features:**
- Form-based feedback collection with fields for name, email, contact, and feedback
- AI-powered classification into complaint, appreciation, or feature request
- Automated workflow processing for instant feedback categorization

![Feedback Classifier Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/1.%20Feedback_Classifier/feedback_classifier_n8n_workflow.png)

---

## 2. n8n Webhook Chatbot

A simple yet powerful chatbot implementation using webhooks for real-time communication. This workflow creates an API endpoint that receives messages via POST requests and responds with AI-generated replies using the Groq Chat Model.

**Key Features:**
- Webhook-based message reception
- AI-powered response generation using Groq's LLaMA 3 70B model
- Real-time conversational capabilities
- RESTful API integration

![Webhook Chatbot Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/2.%20n8n_Webhook_Chatbot/chatbot_webhook.png)

---

## 3. Email Reply AI Agent

An automated email response system that intelligently processes incoming emails and generates appropriate replies. The workflow runs on a schedule (every 5 hours) to check for new emails and automatically crafts personalized responses based on the email content.

**Key Features:**
- Scheduled email processing (every 5 hours)
- AI-powered email analysis and response generation
- Personalized reply crafting based on email type (offers, advertisements, orders)
- Automated Gmail integration for seamless email handling

![Email Reply AI Agent Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/3.%20Email_Reply_AI_Agent/email_agent_workflow.png)

---

## 4. RAG Agent with Pinecone

A comprehensive Retrieval-Augmented Generation (RAG) system consisting of two workflows: a vector store builder and an AI agent. The system processes documents (like PDFs about climate change) and creates a searchable knowledge base using Pinecone vector database.

**Key Features:**
- **Vector Store Workflow**: Monitors Google Drive for document updates, processes PDFs, and stores embeddings in Pinecone
- **RAG Agent Workflow**: Provides intelligent question-answering capabilities using the vector store as a knowledge base
- Chat-based interface for querying the knowledge base
- Specialized in climate change information retrieval

![RAG Vector Store Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/4.%20Rag_Agent_Pinecone/vector_store_rag.png)

![RAG Chatbot Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/4.%20Rag_Agent_Pinecone/rag_chatbot.png)

---

## 5. AI Agent with MCP (Model Context Protocol)

A sophisticated AI agent system using the Model Context Protocol (MCP) that integrates YouTube and Gmail APIs as external tools. This implementation demonstrates how to create custom MCP servers and connect them with AI agents.

**Key Features:**
- **MCP Server**: Exposes YouTube and Gmail APIs as MCP tools
- **AI Agent**: Uses OpenAI's GPT-4o-mini model with MCP tool integration
- Chat-based interface for interacting with external APIs
- Demonstrates advanced AI agent architectures with external tool access

![MCP Agent Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/5.%20AI_Agent_Mcp/mcp_agent.png)
![MCP Server Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/5.%20AI_Agent_Mcp/mcp_server.png)


---

## 6. Twitter Trend Agent

A social media intelligence tool that searches and analyzes Twitter trends based on user queries. The workflow uses the Twitter API to fetch trending tweets and provides insights about current topics and discussions.

**Key Features:**
- Twitter API integration for advanced tweet search
- Query-based trend analysis
- Top tweets retrieval with pagination support
- Real-time social media monitoring capabilities

![Twitter Trend Agent Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/6.%20Twitter_Trend_Agent/twitter-trend-agent.png)

---

## 7. Instagram Lead Generator

A lead generation system that scrapes Instagram for potential business leads based on location and industry criteria. The workflow uses a form interface to collect search parameters and automatically finds relevant Instagram profiles.

**Key Features:**
- Form-based lead search with location and industry filters
- Instagram profile scraping and data extraction
- Lead information processing and organization
- Automated lead discovery for business development

![Instagram Lead Generator Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/7.%20Instagram_Lead_Generator/instagram-leads-generator.png)

---

## 8. Lofi YouTube Video Generation

Fully automated production line for multi-hour ambient/lofi YouTube videos: an LLM agent invents the video concept and writes every asset prompt, Suno V5 (via the kie.ai API) generates the music, an image model paints the background artwork, ffmpeg on a remote server assembles a roughly 2-hour video, and the finished file is published to YouTube with an AI-generated, title-branded thumbnail. Telegram acts as the remote control throughout: it can start a run and receives progress messages, song/artwork previews, error alerts, and the final YouTube link. Includes both the original experimental build and the streamlined rewrite that ran a live channel on a schedule.

**Key Features:**
- AI creative director agent (DeepSeek via OpenRouter + Gemini fallback) with Postgres chat memory and structured output for titles, music/image/animation prompts, descriptions, and hashtags
- Suno V5 music generation with polling loops, credit checks, and Telegram audio previews
- Animated looping background: nano-banana-2 painting turned into a seamless 6-second clip via grok-imagine image-to-video (fal.ai/Seedream experiments preserved in v1)
- Remote ffmpeg video assembly over SSH, looping the animation over each song into a ~2-hour file
- Hands-free YouTube publishing: upload, metadata from an n8n Data Table, and AI-generated branded thumbnail set via the thumbnails API
- Schedule, Telegram, and sub-workflow triggers with error alerts and server cleanup

**Workflow:** [8. Lofi_Youtube_Video_Generation](./8.%20Lofi_Youtube_Video_Generation/)

---

## 9. Islamic History Sleep Stories

A Telegram-driven video factory that turns a raw Urdu history transcript (the bundled sample is a ~55-minute chronicle of the Seljuk Empire) into a finished long-form "sleep story" video for a YouTube channel. A Gemini agent rewrites the transcript into a soothing English bedtime narration, a second agent generates the YouTube title, description and a cinematic thumbnail prompt, and the pipeline then produces TTS audio, an AI thumbnail (ByteDance Seedream v4 via kie.ai) and the final rendered video, uploading everything to Cloudinary. Four saved iterations show the build evolving from an all-inside-n8n ffmpeg pipeline to one that offloads TTS + rendering to an external server.

**Key Features:**
- Dual Gemini 2.5 Pro agents: sleep-story narration converter (Urdu → soothing English) and YouTube metadata generator with structured output (title / description / thumbnail prompt)
- Chunked Gemini Flash TTS loop (voice "Zephyr") with ffmpeg PCM→WAV conversion and concat (v1/v3), later replaced by an external script-to-video render server with async polling (v4/v5)
- AI thumbnail generation via kie.ai Seedream v4 (16:9, 2K) with task polling, credit check and stop-on-error path
- Cloudinary archiving of audio, thumbnail and final 1080p MP4, with `/tmp` cleanup
- Telegram bot as both trigger and live progress console (script, thumbnail preview, Cloudinary links, completion pings)
- Four versions (43 → 53 → 45 → 49 nodes) documenting the architecture's evolution, including an experimental multi-part script splitter in v5

**Workflow:** [9. Islamic_History_Sleep_Stories](./9.%20Islamic_History_Sleep_Stories/)

---

## 10. YouTube Music Video Generation

Turns a single Telegram message into a published YouTube music video. An AI "composer" agent invents a unique song concept, title, description, hashtags and a cinematic thumbnail prompt; kie.ai generates the Suno V5 music tracks and the thumbnail image; ffmpeg assembles a still-image music video; and the result is uploaded to YouTube with the AI-written metadata, with Telegram progress pings at every stage.

**Key Features:**
- REBL_X composer agent with Think tool and structured output — V2 adds Postgres chat memory so no song/title/thumbnail concept is ever repeated
- kie.ai integration for Suno V5 music and Seedream 4 / Imagen 4 thumbnails, with credit checks and Wait/poll loops
- ffmpeg pipeline: concatenates the two generated tracks and renders a 2K still-image music video (local in v1, external rendering server with status polling + retries in v2)
- Two-step YouTube resumable upload with AI-written title, description and hashtags
- Telegram bot as both trigger and live progress/error channel
- Automatic /tmp cleanup and completion message with the video link (V2)

**Workflow:** [10. Youtube_Music_Video_Generation](./10.%20Youtube_Music_Video_Generation/)

---

## 11. Reddit Story Video Factory

A two-workflow content factory that turns viral Reddit posts into narrated "faceless" story videos. v1 is the full YouTube pipeline: it scrapes hot posts from drama/parenting subreddits via an Apify actor, rewrites them into a dramatic first-person script with an AI storyteller agent, voices them with Gemini TTS, burns word-by-word karaoke subtitles over looping background gameplay footage with ffmpeg on a remote server, and publishes to YouTube with AI-generated title, description, tags and thumbnail. v2 repurposes the same stories into 9:16 vertical clips and queues them for TikTok/Facebook posting, with Telegram progress notifications throughout.

**Key Features:**
- Apify Reddit scraping of 8 drama subreddits with score filtering into a Google Sheets content queue (plus a legacy native-Reddit branch with adaptive quality thresholds)
- AI storyteller agent (Gemini 2.5 Pro + OpenRouter fallback) rewrites posts into ~8-minute first-person narrations, with a gender-classifier agent picking a matching Gemini TTS voice (Orus/Zephyr)
- AssemblyAI word-level timestamps converted by Code nodes into ASS karaoke subtitles, rendered onto background video via SSH-driven ffmpeg on a remote server
- Full YouTube publishing: upload, SEO title/description/tags agents, and fal.ai z-image thumbnail set through the YouTube API (v1)
- TikTok/Facebook repurposing: reuses stored narration audio, renders a 1080x1920 vertical cut, publishes via a shared Google Drive file and a `buffer_tiktok_posting` sheet hand-off (v2)
- Sheet-driven state machine (`start_workflow`, `video_generation_completed`, `tiktok_upload_complete`) with Telegram stage notifications and error stops

**Workflow:** [11. Reddit_Story_Video_Factory](./11.%20Reddit_Story_Video_Factory/)

---

## 12. UGC AI Avatar Video Generator

Generates UGC-style "talking avatar" marketing videos end to end: an AI-generated (or uploaded) creator photo is analyzed, split into 8-second scene scripts by an LLM agent, rendered into per-scene clips, and stitched into one final video with ffmpeg. The folder holds two related workflows: a large multi-variant build that experiments with several generation pipelines (Veo 3 image-to-video, Nano Banana Pro image generation, ElevenLabs TTS + Kling AI Avatar lip-sync) behind six web forms, and a Telegram bot integration that productionizes the pipeline with Airtable state tracking and chat commands to re-run failed scenes.

**Key Features:**
- Six pipeline variants in one 188-node workflow: image upload vs. Nano Banana Pro "authentic UGC" photo generation, Veo 3 reference vs. first/last-frame video, and ElevenLabs TTS + Kling AI Avatar lip-sync
- LLM scene direction: GPT-4o-mini vision analysis feeds Claude (Sonnet/Opus via OpenRouter) agents that write 8-second scripts with byte-identical visual descriptions for frame consistency
- Async job orchestration against kie.ai with poll/Wait loops, success/failed/processing Switch routing and Stop-and-Error handling
- Automatic assembly: every clip is uploaded to ffmpeg-api.com and merged with an ffmpeg concat filter into one video
- Telegram front-end (60 nodes): native chat form, live chat-action indicators, per-clip delivery, and the final merged video sent back in the chat
- Airtable-backed resume: every scene tracked as a row; `/c <session-id>` regenerates missing scenes, `/r <video-id>` redoes a single clip

**Workflow:** [12. UGC_AI_Avatar_Video_Generator](./12.%20UGC_AI_Avatar_Video_Generator/)

---

## 13. UGC Ads Factory

A fully automated production line for UGC-style (user-generated-content) ad videos, driven from a Google Sheet. Each row describes a character, setting, and ad script; the workflow generates a photorealistic "selfie" reference image with Kie.ai Nano Banana, has Claude Opus plan the script into 8-second scenes, renders one VEO3 clip per scene, and stitches the clips into a final ad with fal.ai FFmpeg. Every stage writes its status, links, and errors back to the sheet, so the spreadsheet doubles as a campaign dashboard.

**Key Features:**
- Sheet-driven pipeline: mark a row `Create` and collect a finished ad video link in the same row
- AI prompt agent (Claude Opus 4.5) crafts anti-AI-detection selfie prompts for Kie.ai Nano Banana
- GPT-4o-mini image analysis locks visual consistency before scene scripting
- One VEO3 clip per 8-second scene, generated from the shared reference image
- fal.ai FFmpeg merges sorted clips into the final video, uploaded and shared on Google Drive
- Status/error tracking per scene plus a 15-minute regeneration lane for failed scene groups

**Workflow:** [13. UGC_Ads_Factory](./13.%20UGC_Ads_Factory/)

---

## 14. Email Ticket Classifier

AI ticket triage for a multi-client dev team's Gmail inbox. New mail is filtered against an allow-list of client domains, an LLM agent decides whether the email is actionable work, a second agent extracts a fully structured ticket (title, description, goal, current/expected situation, priority, issue type), attachments are uploaded to Google Drive and shared, and the ticket is created in the right client's Notion database — chosen automatically from the sender's domain.

**Key Features:**
- Gmail polling trigger with a client-domain allow-list gate (English + Dutch actionable-work detection)
- Two-stage AI: boolean classifier agent, then a ticket extractor with a strict structured-output schema and autoFix
- Per-client routing: a Code node maps sender domain to the matching Notion database ID
- Attachment handling: re-fetch full message, fan out binaries, upload to Google Drive, share public-read, attach URLs to the ticket
- v4 routes by attachment count through a Switch into dedicated Notion create nodes with emoji priority labels
- Goal / Current Situation / Expected Situation fields make every ticket immediately workable

**Workflow:** [14. Email_Ticket_Classifier](./14.%20Email_Ticket_Classifier/)

---

## 15. Car Body Shop Estimator Bot

A customer-facing Telegram chatbot for "Carbuddy", an auto body repair shop. The bot ("Marco", speaking Italian) accepts text, voice notes, or damage photos, walks the customer through a strict seven-step damage intake (photos, impact details, internal damage, adjacent parts, vehicle make/model/year), produces a preliminary repair-cost range from the photo analysis, and books an inspection appointment directly into Google Calendar. A separate webhook lane demonstrates Twilio SMS/WhatsApp handling.

**Key Features:**
- Multimodal intake: Whisper voice transcription, GPT-4o damage-photo analysis, plain text
- Scripted seven-step damage triage that ends in a preliminary price-range estimate
- Claude Sonnet 4.5 agent with 30-message per-user conversation memory
- Six Google Calendar tools for availability checks and appointment booking/rescheduling
- Twilio webhook lane for SMS/WhatsApp, plus a Telegram user-ID allowlist stub
- Typing indicator and Telegram-native replies for a polished customer experience

**Workflow:** [15. Car_Body_Shop_Estimator_Bot](./15.%20Car_Body_Shop_Estimator_Bot/)

---

## 16. Telegram Personal Assistant

A personal AI assistant that lives in Telegram and understands text, voice notes, and photos. It manages the owner's Google Calendar (create, read, update, delete, availability) and Gmail (send, search, read, reply, delete) through eleven agent tools, powered by Claude Haiku 4.5 via OpenRouter with a 30-message conversation memory. Voice is transcribed with Whisper and photos are described by GPT-4o before reaching the agent, so all three input types land in one unified prompt.

**Key Features:**
- Text, voice, and photo input unified into a single agent prompt via a 3-way Switch + Merge
- Eleven agent tools: full Google Calendar CRUD + availability and five Gmail operations
- Claude Haiku 4.5 via OpenRouter keeps replies fast and cheap
- Per-user session memory (30-message window keyed by Telegram user ID)
- Personal-assistant persona with strict plain-text, short-answer output rules
- Authorization gate for restricting the bot to your own Telegram user ID

**Workflow:** [16. Telegram_Personal_Assistant](./16.%20Telegram_Personal_Assistant/)

---

## 17. AI Appointment Scheduler

A voice-agent appointment backend for a dental-clinic receptionist bot. Three webhook endpoints implement the tool calls of a VAPI-style voice assistant (check availability, book, cancel): they read the assistant's `toolCalls` payload, work against Google Calendar, log bookings in an n8n Data Table, and answer in the tool-result format the voice platform expects. When a requested slot is taken, a Code node computes the free slots for the next 7 working days and speaks them back as alternatives.

**Key Features:**
- Three webhook tool endpoints (check availability / create / cancel) speaking the VAPI tool-call protocol
- Smart alternative-slot engine: business hours, working days, timezone and 2-hour lead time in one Code node
- Google Calendar event lifecycle with attendee invites and cancellation emails (`sendUpdates: all`)
- n8n Data Table as booking ledger — cancellations look up the event by caller phone + email
- Error-tolerant cancel path that still answers the voice agent when the calendar delete fails
- Bonus disabled chat-agent variant with four Google Calendar AI tools

**Workflow:** [17. AI_Appointment_Scheduler](./17.%20AI_Appointment_Scheduler/)

---

## 18. YouTube Shorts Cross-Poster

A cross-posting pipeline that turns every new YouTube Short into scheduled posts on Pinterest, Facebook and TikTok. YouTube's WebSub (PubSubHubbub) push notifies the workflow the moment a Short is published; the video is downloaded in up to 4K via an Apify actor, analyzed frame-by-frame by Gemini, and an LLM chain writes platform-specific SEO metadata before scheduling everything through a social-publishing API — fully hands-off.

**Key Features:**
- Real-time YouTube WebSub trigger with `hub.challenge` handshake handling
- Apify actor downloads the Short as mp4 (up to 2160p) for re-publishing
- Gemini video analysis + structured-output LLM chain (with model fallback) for Pinterest/Facebook SEO copy
- AI board selection mapped to Pinterest board IDs with a safe default
- Duration-aware Facebook branch (reel vs video) and TikTok rate-limit check with next-day deferral
- All posts scheduled through a Late-compatible publishing API instead of raw platform APIs

**Workflow:** [18. Shorts_Cross_Poster](./18.%20Shorts_Cross_Poster/)

---

## 19. LinkedIn Post & PDF Newsletter Generator

A one-click AI-news publishing pipeline: it scrapes the transcript of a weekly AI-updates YouTube video, has an LLM write a clean, anti-hype LinkedIn post from it, then expands the same material into a multi-page A4 PDF newsletter (cover page, one deep-dive card per tool, CTA page) rendered from generated HTML through a self-hosted Gotenberg service. The post text and the PDF are produced in a single run, ready to publish as a LinkedIn document post.

**Key Features:**
- Apify YouTube Scraper pulls plaintext subtitles as the primary source of truth
- Gemini 3 Pro post-writer with strict anti-hype tone and copy-paste-ready formatting rules
- Three parallel LLM/Set branches build cover, body cards and CTA, merged by position into one HTML document
- Print-grade A4 layout with inline CSS, converted to PDF by a self-hosted Gotenberg Chromium endpoint
- Dynamic cover date stamping via regex post-processing of the LLM output
- Disabled LinkedIn node marks the publish hand-off point

**Workflow:** [19. LinkedIn_Post_Generator](./19.%20LinkedIn_Post_Generator/)

---

## 20. Sticky Note Autoannotator

A meta-workflow that documents other n8n workflows: paste any workflow's JSON in and it uses AI to group the nodes into logical clusters, writes a main overview sticky plus one titled sticky per cluster, sizes and places every sticky on the canvas without overlaps, and can optionally AI-rename every node to a descriptive name — returning import-ready JSON. It replicates n8n's real node-dimension math and runs an iterative collision solver with a best-result loop, so the generated canvases look hand-annotated.

**Key Features:**
- AI grouping that respects both logical purpose and spatial layout, with structured output + autoFix
- Node dimensions ported from n8n's own `nodeViewUtils` (sub-nodes, configurable AI parents, slot heights)
- Grid-snapped collision resolution plus a static-data retry loop that keeps the best of up to 3 passes
- Optional AI node renaming that safely rewrites `$('Node')` expression references, connections and pinData keys
- Generates the main overview sticky with how-it-works list and checkbox setup steps
- Ships with a sanitized demo workflow as example input

**Workflow:** [20. Sticky_Note_Autoannotator](./20.%20Sticky_Note_Autoannotator/)

---

## 21. n8n GitHub Backup

Automatically backs up every workflow from two n8n instances to a GitHub repository on a 12-hour schedule. A Code node strips volatile fields (IDs, positions, credentials, metadata) and deep-diffs each workflow against the copy already on GitHub, so it only commits when the actual logic changed. Two parallel branches back up the main self-hosted instance and a second cloud instance into separate repo folders.

**Key Features:**
- Scheduled every 12 hours, fully unattended, with retry on the n8n API calls
- Change-aware commits: deep-clean + key-sorted JSON diff avoids noise commits
- Auto-generated slugified filenames from workflow name + first tag, timestamped commit messages
- Create-vs-update branch: new workflows uploaded, existing files edited only on real change
- Dual-instance support with independent n8n API credentials and target folders
- 42 nodes with sticky-note documentation of every section

**Workflow:** [21. n8n_GitHub_Backup](./21.%20n8n_GitHub_Backup/)

---

## 22. FFmpeg Gif To Mp4

Media-conversion demo built on the hosted FFmpegAPI service — no local ffmpeg install needed. A public n8n form accepts a GIF and returns an MP4 download link, and a second manual pipeline merges multiple source videos into one MP4 with an ffmpeg concat filter graph running in the cloud.

**Key Features:**
- Form Trigger upload page: attach a `.gif`, get a "Done!" page with the MP4 download link
- Signed upload-slot flow: request URL, PUT binary, submit ffmpeg processing task
- Multi-video merge pipeline with `concat=n=N:v=1:a=1` filter and Wait-based job pacing
- API auth shown two ways: n8n Variable (`$vars.FFMPEG_API_KEY`) and HTTP Header Auth credential
- Sticky-note walkthrough for account setup and usage
- 20 nodes across two independent demo pipelines

**Workflow:** [22. FFmpeg_Gif_To_Mp4](./22.%20FFmpeg_Gif_To_Mp4/)

---

## 23. SSH Remote Video Renderer

Helper workflow that offloads CPU-heavy ffmpeg rendering to a remote Linux server over SSH. n8n ships a background image to the server, runs ffmpeg there to combine image, audio, and burned-in subtitles into a video, then pulls the finished MP4 back — a building block for the video-generation workflows when the n8n host is too weak to render.

**Key Features:**
- Remote rendering over SSH with a private-key credential (host/user never in the JSON)
- Uploads the local background image, downloads the finished MP4 as workflow binary
- Single ffmpeg command: looped image + MP3 audio + `.ass` subtitle burn-in, libx264/AAC
- Retry with backoff on the file upload step
- Minimal 5-node design, easy to embed in bigger pipelines

**Workflow:** [23. SSH_Video_Rendering](./23.%20SSH_Video_Rendering/)

---

## 24. TikTok Posting Queue

Helper that stages a video for TikTok publishing. It refreshes a staging file on Google Drive with new video content, shares it publicly, and appends a ready-to-post row (title, caption, public link, `Publish = Go`) to a `buffer_tiktok_posting` Google Sheet that a downstream posting integration consumes as a queue.

**Key Features:**
- Downloads the source video from any direct-download URL
- Overwrites a fixed Drive staging file so the public link stays stable
- Sets `anyone / reader` sharing to make the video fetchable by the poster
- Appends Title / Caption / Link / Publish columns to the posting-queue sheet
- Clean 5-node handoff pattern between n8n and an external publishing tool

**Workflow:** [24. TikTok_Posting_Queue](./24.%20TikTok_Posting_Queue/)

---

## 25. Telegram Error Alerts

Tiny utility that turns any failure across the n8n instance into an instant Telegram alert. Registered as the "Error Workflow" of other workflows, it messages the workflow name, HTTP code, error message, and failing node type — with a fallback branch that still notifies you when even the detailed alert cannot be sent.

**Key Features:**
- Error Trigger fires automatically for every monitored workflow failure
- Detailed alert: workflow name, HTTP code, error message, failing node type
- Two-tier fallback via `continueErrorOutput` — a minimal alert always gets through
- One credential, one chat ID: drop-in monitoring for the whole instance
- 4 nodes; pair it with any workflow via Settings → Error Workflow

**Workflow:** [25. Telegram_Error_Notification](./25.%20Telegram_Error_Notification/)

---

## 26. YouTube WebSub Keepalive

Keepalive workflow for YouTube push notifications. Subscriptions to Google's PubSubHubbub (WebSub) hub expire after a few days, so this scheduled job re-subscribes a list of YouTube channels every 4 days, keeping a companion webhook workflow supplied with instant "new video" pings. It ran as an always-active production utility feeding an anime/manga channel monitoring pipeline.

**Key Features:**
- Schedule Trigger every 4 days, safely inside the WebSub lease window
- Set node maps 14 YouTube channel handles to channel IDs; Split Out fans out one item per channel
- Form-encoded POST per channel to Google's PubSubHubbub subscribe endpoint (`hub.mode=subscribe`, `hub.verify=sync`)
- `hub.callback` points at the receiving n8n webhook (placeholdered)
- Built-in retry: 5 attempts with 5 s backoff on failed subscribe calls

**Workflow:** [26. YouTube_WebSub_Keepalive](./26.%20YouTube_WebSub_Keepalive/)

---

## 27. Email to Notion Ticket Triage

A small email-triage helper in progress: Gmail is polled every minute and an LLM-backed Text Classifier labels each incoming email "ticket" or "no ticket". A Notion "create database page" node sits on the canvas ready to log tickets into a Notion database, but its input is not yet wired to the classifier — exported honestly as the work-in-progress scaffold it is.

**Key Features:**
- Gmail Trigger polling every minute (full message format, spam/trash excluded)
- Text Classifier with two categories: `ticket` vs `no ticket`, driven by the email snippet
- OpenRouter Chat Model (Gemini 2.5 Flash) as the classification LLM
- Notion database-page creation node staged for the ticket branch
- Minimal 4-node scaffold showing an email-to-ticket triage pattern

**Workflow:** [27. Email_To_Notion_Triage](./27.%20Email_To_Notion_Triage/)

---

## 28. n8n Course Exercises

The complete exercise set from n8n's official Level 1 (beginner) and Level 2 (intermediate) courses, kept on one canvas as four independent sections: scheduled ERP order processing with conditional routing, Airtable + HTTP merging and sorting, regional report generation with Gmail and Discord notifications plus batch looping, and country-data enrichment upserted back into Airtable. Useful as a compact reference for core n8n primitives: IF, Merge (combine and enrich modes), Sort, Loop Over Items, Convert to File, Date & Time, and Code.

**Key Features:**
- Weekly scheduled ERP pull with header-auth HTTP and IF-based order routing
- Merge nodes joining Airtable records with API data on `customerID`, plus enrich-mode join against REST Countries
- Discord webhook reporting of aggregated totals computed in Code nodes
- Regional split: per-order JSON files emailed via Gmail vs batched spreadsheet loop (Loop Over Items, batches of 5)
- Date & Time formatting and `$runIndex`-based dynamic file naming
- Airtable upsert writing enriched region/subregion data back to the base

**Workflow:** [28. n8n_Course_Exercises](./28.%20n8n_Course_Exercises/)

---

## 29. GoHighLevel API Bridge

Bridges a GoHighLevel (GHL) funnel form to an external browser-automation task API. A GHL workflow webhook fires into n8n with contact fields and uploaded document URLs; n8n normalizes the data, downloads the documents, submits everything as a multipart task to the backend service, then polls the task every minute until it finishes and pushes a completed/failed status back into GHL so the client-side automation can continue.

**Key Features:**
- Webhook intake of GHL `customData` with pure-expression cleaning: US state name to 2-letter code mapping and multi-format DOB parsing that fails fast on bad input
- Downloads three form-uploaded documents with an authenticated request, then Merge + Aggregate packs data and binaries into a single multipart task submission
- Hardened submission path with `retryOnFail` (5 tries) and an `Idempotency-Key` header so retries never duplicate tasks
- Wait + Switch polling loop over five task states (running/queued/failed/completed/cancelled)
- Status callback to a GHL inbound webhook trigger resumes the client-side workflow with contact id, phone and name
- Three parallel submit/poll/callback pipelines targeting dev/prod endpoints of the same API

**Workflow:** [29. GoHighLevel_API_Bridge](./29.%20GoHighLevel_API_Bridge/)

---

## 30. Cold Email Outreach

A complete cold-email machine for a bookkeeping-services business. It scrapes local-business leads from Google Maps via Apify (with contact enrichment), stores businesses and decision-maker contacts in Airtable, scrapes each lead's website with Firecrawl for personalization context, then drip-sends AI-written, individually personalized HTML emails through Gmail on a weekday schedule — pacing sends minutes apart and logging every email back to Airtable.

**Key Features:**
- Apify Google Maps Scraper actor with contact scraping + leads enrichment across multiple service-business niches
- Two-table Airtable CRM: Businesses upserted by `placeId`, enriched decision-maker Contacts by `PersonID`
- Firecrawl website scraping loop with IF gating and error branches that never stall the batch
- Two-stage LLM pipeline: persona-driven plain-text email writer, then an HTML formatter with Structured Output Parser + auto-fix
- Cron-scheduled weekday sending with a 200-contact cap and multi-minute waits between emails for deliverability
- Every send logged back to Airtable with both text and HTML bodies for auditability

**Workflow:** [30. Cold_Email_Outreach](./30.%20Cold_Email_Outreach/)

---

## 31. Customer Survey Forms

Three production customer-survey flows for an e-commerce brand, built entirely with n8n's native Form nodes — no external form tool. There is an exit-intent survey (incentivized with a discount code), a returning-customer survey, and a branching new-customer survey whose page sequence adapts to how the shopper found the store. Every path ends by shaping a normalized analytics payload (survey slug, timestamp, path taken, answers).

**Key Features:**
- Three independent Form Triggers in one workflow: exit-intent, returning-customer and new-customer surveys
- Conditional required-field enforcement: choosing "Other" without write-in text routes to a "cannot be left blank" completion page
- Switch-based branching: attribution channel decides whether the purchase-reason page is shown
- Mirrored multi-page question tracks (solutions tried, first heard, purchase friction, medication check)
- Uniform `{ surveySlug, submittedAt, pathTaken, answers }` payload Set nodes, ready to wire to any datastore
- Zero credentials — runs on built-in Form, Set, IF and Switch nodes only

**Workflow:** [31. Customer_Survey_Forms](./31.%20Customer_Survey_Forms/)

---

## Getting Started

To use these workflows:

1. Import the JSON files into your n8n instance
2. Configure the required credentials (API keys, OAuth tokens) — every workflow folder has its own README listing the credential types and placeholders to fill
3. Set up any necessary external services (Pinecone, Gmail, Twitter API, kie.ai, fal.ai, etc.)
4. Test the workflows with sample data
5. Deploy and monitor your AI agents

## Technologies Used

- **n8n**: Workflow automation platform (self-hosted), including AI Agents, Data Tables, Forms, MCP and sub-workflows
- **AI Models**: OpenAI GPT-4o / GPT-4o-mini / Whisper, Claude (Opus, Sonnet, Haiku), Gemini 2.5 Pro / Flash / TTS, DeepSeek, Groq LLaMA 3 70B — mostly via OpenRouter
- **Media Generation**: kie.ai (Suno V5 music, Seedream, Veo 3, Nano Banana), fal.ai, ElevenLabs TTS, Kling AI Avatar
- **Video/Docs Tooling**: ffmpeg (local, over SSH, FFmpegAPI, fal.ai FFmpeg), Gotenberg PDF rendering, Cloudinary
- **Vector Database**: Pinecone
- **APIs & Services**: Gmail, Google Calendar / Sheets / Drive, YouTube (uploads + WebSub push), Telegram, Notion, Airtable, Slack, Discord, Twitter, Instagram, Pinterest / Facebook / TikTok scheduling, GoHighLevel, Twilio, Apify, Firecrawl, AssemblyAI
- **Protocols**: MCP (Model Context Protocol), WebSub/PubSubHubbub
- **State & Memory**: Postgres chat memory, n8n Data Tables, Google Sheets state machines

## Security & Sanitization Note

All workflow JSONs in this repository are **sanitized exports** from a live n8n instance:

- API keys, tokens, credential IDs, chat IDs, emails, phone numbers, server addresses and personal/client identifiers are replaced with readable placeholders like `YOUR_API_KEY`, `YOUR_TELEGRAM_CHAT_ID`, `your-email@example.com`
- `pinData` (pinned test executions) and instance metadata are removed, and every workflow is exported with `active: false`
- Credential blocks keep their **type** so n8n shows which credential to connect, but you must re-select your own credentials after importing
- Each folder's README lists the exact placeholders to fill before the workflow will run

## Contributing

Feel free to fork this repository and submit pull requests with improvements or new workflow ideas!
