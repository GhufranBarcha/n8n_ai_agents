# Sticky Note Autoannotator

A meta-workflow that documents other n8n workflows: paste any workflow's JSON in and it uses AI to group the nodes into logical clusters, writes a main overview sticky (how it works, setup checklist, customization tips) plus one titled sticky per cluster, sizes and places every sticky on the canvas without overlaps, and can optionally AI-rename every node to a descriptive name — returning import-ready JSON. It replicates n8n's real node-dimension math (trigger/sub-node/configurable widths, input/output slot heights) and runs an iterative collision solver with a best-result loop, which is what makes the generated canvases actually look hand-annotated.

## Trigger

Manual (`Start`).

## How it works

32 nodes total (22 functional + 10 sticky notes documenting the sections).

- `Set Workflow Variables` holds the input: the target workflow JSON (shipped with a sanitized demo "UGC Ads Factory" video pipeline as example payload), `MAX_RETRIES` (3) and a `renameNodes` flag.
- `Strip & Prepare` (Code) removes existing stickies and builds connection maps, AI sub-node parent relations (`ai_languageModel`/`ai_tool`/`ai_memory`/`ai_outputParser`) and input/output slot counts.
- `Parse Nodes` (Code) computes each node's on-canvas dimensions (ported from n8n's `nodeViewUtils`) and extracts a per-type context description (HTTP method+URL, Set fields, If conditions, Code hints, ...).
- `AI Groups Logically` (agent + Structured Output Parser with autoFix, OpenRouter model) groups nodes by purpose AND spatial proximity and writes the main overview (numbered how-it-works, checkbox setup steps).
- `Compute Bounding Boxes` → `Collision Resolution` (Code) turn each group into a sticky rectangle with dynamic title/description padding, then push overlapping stickies apart along the axis of least overlap (grid-snapped, up to 15 iterations), shifting member nodes along.
- `Generate Stickies` → `Merge & Export` build the main overview sticky (height matched to the workflow's vertical span) and reassemble the workflow with stickies rendered behind nodes.
- `Collision Detector` re-checks sticky-vs-sticky and foreign-node overlaps and repairs edges; `Loop Controller (Automatic Fixer)` uses workflow static data to re-run the whole annotation pass up to `MAX_RETRIES`, keeping the pass with the fewest remaining collisions; `If` + `Pick Best Result` close the loop.
- `Should Rename Nodes` optionally branches into `Parse for Renaming` → `AI Rename` (second OpenRouter model + structured parser, naming-convention prompt) → `Apply Renames` (Code), which renames nodes and rewrites every `$('Node Name')` expression reference, connection key and pinData key consistently → `Format for Export` → `Output Normalization` returns the final workflow JSON string.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| OpenRouter | `openRouterApi` | Grouping/overview model and node-renaming model |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `workflow` assignment (embedded demo JSON) | `Set Workflow Variables` | The JSON of the workflow you want annotated |
| `YOUR_GOOGLE_SPREADSHEET_ID`, `YOUR_DRIVE_FOLDER_ID`, `YOUR_CREDENTIAL_ID`, `YOUR_INSTANCE_ID`, `https://docs.google.com/.../YOUR_GOOGLE_SPREADSHEET_ID/...` | inside the embedded demo workflow string | Nothing — demo payload only; replaced automatically when you paste your own workflow |
| `renameNodes` | `Set Workflow Variables` | `true` to also AI-rename all nodes, `false` for stickies only |
| `MAX_RETRIES` | `Set Workflow Variables` | Max annotation passes when collisions remain |
| `YOUR_OPEN_ROUTER_API_CREDENTIAL` | both OpenRouter chat model nodes | Re-select your own credential (any chat-model provider works) |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
