# n8n GitHub Backup

Automatically backs up every workflow from two n8n instances to a GitHub repository on a 12-hour schedule. It is smart about what it commits: a Code node deep-cleans volatile fields (IDs, positions, credentials, timestamps, metadata) and compares the result against the copy already on GitHub, so a commit only happens when the actual workflow logic changed. The workflow contains two parallel branches — one for the main self-hosted instance and one for a second cloud instance — each writing into its own folder of the backup repo.

## Trigger

Schedule (every 12 hours). A disabled Telegram Trigger is also wired to the second branch for manual kick-off of the cloud backup.

## How it works

- Schedule Trigger fires and each branch sets its GitHub target folder (`oracle-arm-n8n` / `cloud-workflows`) in an Input node.
- GitHub "List files from repo" + Aggregate collect the filenames already present in the backup folder.
- The n8n node ("Retrieve workflows", n8n API credential) fetches all workflows of the instance, excluding pinned data; each is converted to formatted JSON and base64-encoded for the GitHub API.
- A Set node builds a slugified filename from the workflow name and first tag, plus a timestamped commit message.
- An If node checks whether that file already exists in the repo: new files are uploaded directly ("Upload file", which also sets the commit author name/email).
- Existing files are fetched from GitHub, parsed, and diffed in a Code node (`deepClean` + key-sorted JSON stringify); only when `hasChanges` is true does the "Update file" node commit the new version.
- The second branch repeats the same pipeline against the second n8n instance with its own n8n API credential.
- 42 nodes total (including 10 sticky notes documenting the two sections and setup steps).

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| n8n API (x2 instances) | `n8nApi` | Retrieving all workflows of each instance |
| GitHub | `githubApi` | Listing, reading, uploading, and updating backup files |
| Telegram | `telegramApi` | Optional manual trigger of the cloud branch (node disabled) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_N8N_API_CREDENTIAL` | Both "Retrieve workflows" nodes | An n8n API key credential for each instance |
| `YOUR_GITHUB_API_CREDENTIAL` | All GitHub nodes | A GitHub personal-access-token credential |
| `YOUR_TELEGRAM_API_CREDENTIAL` | Telegram Trigger (disabled) | Your bot credential, if you enable it |
| `your-username` | GitHub `owner` fields and commit `author.name` in both "Upload file" nodes | Your GitHub username |
| `your-email@example.com` | Commit `author.email` in both "Upload file" nodes | Your GitHub commit email |
| `self-hosted-n8n-backup` | GitHub `repository` fields | The repo you created for backups |
| `oracle-arm-n8n` / `cloud-workflows` | The two Input nodes | Folder names you want inside the backup repo |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
