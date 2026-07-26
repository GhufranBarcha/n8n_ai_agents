# UGC Ads Factory

A fully automated production line for UGC-style (user-generated-content) ad videos, driven from a Google Sheet. Each row describes a character, setting, and ad script; the workflow generates a photorealistic "selfie" reference image with Kie.ai Nano Banana, has Claude Opus plan the script into 8-second scenes, renders one VEO3 clip per scene, and stitches the clips into a final ad with fal.ai FFmpeg. Every stage writes its status, links, and errors back to the sheet, so the spreadsheet doubles as a campaign dashboard.

## Trigger
Three entry points: a Schedule Trigger every 30 minutes (main pipeline: pick up rows marked `Create` in the **Videos** tab), a second Schedule Trigger every 15 minutes (regeneration lane: re-queue scene groups that never finished), and an Execute Workflow Trigger so another workflow can start a run directly.

## How it works
- 61 nodes across three lanes: intake + image generation, scene scripting, and per-scene video generation + stitching.
- Rows from the **Videos** sheet are normalized (character description, script, aspect ratio, generated run ID) and the row is flipped to `Processing`.
- A "Nano Banana Prompt Agent" (Claude Opus 4.5 via OpenRouter, with a structured output parser) crafts an ultra-realistic mobile-selfie image prompt; the workflow submits it to Kie.ai's job API and polls `recordInfo` in a Wait loop until success or failure.
- The finished reference image is downloaded, uploaded to Google Drive, and shared publicly; GPT-4o-mini then analyzes it so every video scene keeps a consistent look.
- A second agent (Claude Opus 4.5 + structured parser) splits the ad script into ordered ~8-second scenes with identical visual descriptions and setting-aware movement prompts; each scene is appended to the **Video Data** tab.
- For each scene row, the workflow calls Kie.ai VEO3 (`veo3_fast`, first-and-last-frame mode, using the reference image), polls a status Switch (Success / Failed / Processing / Unknown) with a 10-minute Wait loop, uploads finished clips to Drive, and records clip links or error details per scene.
- When a Code node confirms all scenes for a video are `Completed`, clips are sorted by scene number, aggregated, merged via the fal.ai FFmpeg `merge-videos` model, and the final video is uploaded, shared, and written back to the **Videos** row as `Completed`.
- Failure branches update the sheet (`Failed` + error message) at every stage, and the 15-minute regeneration lane picks the least-finished scene group and re-runs it.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| Google Sheets | `googleSheetsOAuth2Api` | Reading video ideas, writing scene rows, status, links, and errors |
| Google Drive | `googleDriveOAuth2Api` | Uploading and publicly sharing reference images, scene clips, and final videos |
| Kie.ai | `httpHeaderAuth` (generic header auth) | Nano Banana image generation and VEO3 video generation + polling |
| OpenRouter | `openRouterApi` | Claude Opus 4.5 for the prompt agent and the scene-script agent |
| OpenAI | `openAiApi` | GPT-4o-mini vision analysis of the reference image |
| fal.ai | fal.ai community node credential | FFmpeg merge of scene clips into the final video |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GOOGLE_DOCUMENT_ID` | `documentId` in all Google Sheets nodes | Your spreadsheet ID (with **Videos** and **Video Data** tabs) |
| `YOUR_GOOGLE_FOLDER_ID` | `folderId` in the three Google Drive upload nodes | Drive folder IDs for images and rendered videos |
| `YOUR_GOOGLE_SHEETS_OAUTH2_API_CREDENTIAL` | 13 Google Sheets nodes | Your own Google Sheets credential (IDs were rewritten too) |
| `https://example.com/redacted` / `Your resource` | `cachedResultUrl` / `cachedResultName` fields | Auto-refreshes once you re-pick the sheet/tab from the list |
| `https://your-domain.com/api/callback` | Kie.ai `createTask` request body | Optional callback URL (polling covers completion either way) |
| Sheet tab selectors left blank | `Get Video Ideas` and `Get row(s) in sheet1` | Re-select the **Videos** tab after import |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
