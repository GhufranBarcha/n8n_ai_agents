# FFmpeg GIF to MP4

Media-conversion demo built on the hosted [FFmpegAPI](https://ffmpeg-api.com/) service — no local ffmpeg install needed. It contains two independent pipelines: a form-based GIF-to-MP4 converter, and a manual pipeline that merges several source videos into one MP4 using an ffmpeg `concat` filter graph running in the cloud.

## Trigger

Two triggers: an n8n Form Trigger ("Attach file", path = the node's `webhookId` placeholder) that serves a public upload form with no authentication, and a Manual Trigger for the video-merge pipeline.

## How it works

- Pipeline 1 (GIF to MP4): the "Attach file" form accepts a `.gif` upload; "Get Upload URL" requests a signed upload slot from FFmpegAPI; Merge pairs the binary with the slot; "Upload File" PUTs the GIF; "Process File" submits an ffmpeg task producing `output.mp4`; the "Download URL" form completion page shows the result's download link.
- Pipeline 2 (merge videos): Manual Trigger creates a working directory on FFmpegAPI; a Code node lists sample source video links (Google Drive direct-download URLs — swap in your own); each is downloaded and re-uploaded to FFmpegAPI; a Code node aggregates all `file_path`s; after a 3-minute Wait, "ffmpeg-process" runs a `concat=n=N:v=1:a=1` filter to produce `merged_video.mp4`, which is then downloaded.
- Auth is passed two ways on purpose, as a demo: an n8n Variable `{{ $vars.FFMPEG_API_KEY }}` in Authorization headers, and an HTTP Header Auth credential on other nodes.
- 20 nodes total (including 3 sticky notes with step-by-step setup instructions and 1 disabled trailing HTTP Request).

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| FFmpegAPI | `httpHeaderAuth` | Directory/file management and ffmpeg processing calls |
| FFmpegAPI | n8n Variable `FFMPEG_API_KEY` | `Basic` Authorization header on the GIF pipeline |
| Google Drive (public links) | none | Sample source videos for the merge pipeline |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `FFMPEG_API_KEY` | n8n Variables section | Your FFmpegAPI key from the dashboard |
| `YOUR_HTTP_HEADER_AUTH_CREDENTIAL` | "Process File", "get_url", "HTTP Request2", "ffmpeg-process" | Header credential `Authorization: Basic <your key>` |
| Sample Drive links | "Code in JavaScript" node | Direct-download URLs of your own source videos |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing. Note: the upload form is unauthenticated by design — anyone with the URL can submit files while the workflow is active.
