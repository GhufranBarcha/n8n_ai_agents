# SSH Remote Video Renderer

Small helper workflow that offloads CPU-heavy ffmpeg video rendering to a remote Linux server over SSH. n8n only orchestrates: it ships a background image to the server, runs ffmpeg there to combine image + audio + burned-in subtitles into a video, and pulls the finished MP4 back. Useful as a building block for the YouTube/lofi video-generation workflows in this repo when the n8n host itself is too weak to render.

## Trigger

Manual ("Execute workflow").

## How it works

- Manual Trigger starts the run.
- "Read/Write Files from Disk" loads a local image (`/tmp/youtube_image.png`) as binary data.
- SSH "Upload a file" copies it to the server as `/home/ubuntu/background_image.png` (retries with a 5 s backoff).
- SSH "Execute a command" runs ffmpeg on the server: loops the image, muxes `/home/ubuntu/subtitle.mp3`, burns `/home/ubuntu/subtitles.ass` subtitles, and encodes `output_video.mp4` (libx264 + AAC, `-shortest`).
- SSH "Download a file" fetches `/home/ubuntu/output_video.mp4` back into the workflow as binary.
- 5 nodes total.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Remote Linux server (ffmpeg installed) | `sshPrivateKey` | File upload, remote ffmpeg execution, file download |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_SSH_PRIVATE_KEY_CREDENTIAL` | All three SSH nodes | SSH private-key credential (host, user, key) for your render server |
| `/tmp/youtube_image.png` | "Read/Write Files from Disk" | Path of the local image to render |
| `/home/ubuntu/...` paths | SSH nodes | Home directory of your server user; `subtitle.mp3` and `subtitles.ass` must already exist there |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing. The server address and user live entirely inside the SSH credential, not in the workflow JSON.
