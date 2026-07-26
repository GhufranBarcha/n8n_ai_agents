# YouTube WebSub Keepalive

Keepalive workflow for YouTube push notifications. Subscriptions to Google's PubSubHubbub (WebSub) hub expire after a few days, so this scheduled job re-subscribes a list of YouTube channels every 4 days, keeping a companion webhook workflow supplied with instant "new video" pings. It ran as an always-active production utility feeding an anime/manga channel monitoring pipeline.

## Trigger
Schedule — every 4 days. The subscribe call registers a callback at the placeholder URL `https://your-n8n-instance.com/webhook/your-youtube-push-webhook` (point it at the webhook of your receiving workflow).

## How it works
- 6 nodes total.
- **Schedule Trigger** fires every 4 days (comfortably inside the hub's lease window).
- **channel_details1** (Set) defines an object mapping 14 YouTube channel handles to their channel IDs; a second Set (**channel_details**) holds an older 7-channel list and is intentionally left unconnected as a kept-around variant.
- **Split Out** explodes the channels object into one item per channel ID.
- **HTTP Request** POSTs a form-encoded subscribe request per channel to `https://pubsubhubbub.appspot.com/subscribe` with `hub.mode=subscribe`, `hub.verify=sync`, `hub.topic` set to that channel's YouTube XML feed, and `hub.callback` set to the receiving webhook; retries up to 5 times with a 5 s wait.
- **Get a video** (YouTube, OAuth2) sits disconnected on the canvas — a leftover utility for exercising the YouTube credential; it never runs.

## Services & credentials
| Service | n8n credential type | Used for |
|---|---|---|
| YouTube | youTubeOAuth2Api | "Get a video" utility node (disconnected) |
| Google PubSubHubbub hub | none | public subscribe endpoint, no auth needed |

## Placeholders to fill after import
| Placeholder | Where | Replace with |
|---|---|---|
| `your-n8n-instance.com` | HTTP Request → body parameter `hub.callback` | your n8n domain + the webhook path of the workflow that should receive the pings |
| `YOUR_YOU_TUBE_OAUTH2_API_CREDENTIAL` | Get a video node | your own YouTube OAuth2 credential (or delete the node) |
| Channel ID map | channel_details1 (Set) | the YouTube channel IDs you want to monitor |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
