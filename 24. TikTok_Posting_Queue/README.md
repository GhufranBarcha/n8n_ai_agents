# TikTok Posting Queue

Helper that stages a video for TikTok publishing. It refreshes a staging file on Google Drive with new video content, makes it publicly downloadable, and appends a ready-to-post row (title, caption, public link, `Publish = Go`) to a `buffer_tiktok_posting` sheet. It does not call the TikTok API itself — the sheet acts as a posting queue that a downstream scheduler/integration (e.g. Buffer) consumes to publish the video.

## Trigger

Manual ("Execute workflow").

## How it works

- Manual Trigger starts the run.
- "HTTP Request" downloads the source video from a direct-download link (sample Google Drive URL — replace with your own source).
- Google Drive "Update file1" overwrites the content of an existing staging file (`test.mp4`) with the downloaded binary.
- Google Drive "Share file" sets `anyone / reader` permission on that file so the link is publicly fetchable.
- Google Sheets "AddTikTokVideo" appends Title, Caption, the file's `webContentLink`, and `Publish = Go` to the `buffer_tiktok_posting` tab, queueing it for the downstream posting automation.
- 5 nodes total.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Google Drive | `googleDriveOAuth2Api` | Updating the staging video file and sharing it publicly |
| Google Sheets | `googleSheetsOAuth2Api` | Appending the posting-queue row |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GOOGLE_DRIVE_OAUTH2_API_CREDENTIAL` | "Update file1", "Share file" | Your Google Drive OAuth2 credential |
| `YOUR_GOOGLE_SHEETS_OAUTH2_API_CREDENTIAL` | "AddTikTokVideo" | Your Google Sheets OAuth2 credential |
| `YOUR_GOOGLE_FILE_ID` | "Update file1" | ID of a staging video file in your Drive |
| `YOUR_GOOGLE_SHEET_ID` | "AddTikTokVideo" | Your spreadsheet with a `buffer_tiktok_posting` tab (Title / Caption / Link / Publish columns) |
| Source video URL | "HTTP Request" | Direct-download link of the video you want to stage |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
