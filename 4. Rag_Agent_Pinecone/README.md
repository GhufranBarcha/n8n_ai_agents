# RAG Agent with Pinecone

A comprehensive Retrieval-Augmented Generation (RAG) system consisting of two workflows: a vector store builder and a chat agent. The builder watches a document on Google Drive (originally a climate-change PDF), chunks and embeds it with OpenAI embeddings, and stores the vectors in a Pinecone index. The agent then answers chat questions using that index as a knowledge base tool.

## Workflows in this folder

| File | Role |
|---|---|
| `3_1_Rag_Vector_Store.json` | Ingestion: Google Drive file to chunks to embeddings to Pinecone index `ai-agent-rag` |
| `3_2_AI_Rag_Agent.json` | Chat agent: answers questions with the Pinecone index attached as a retrieval tool |

## Trigger

- 3.1: Google Drive Trigger — polls every minute for updates to one specific file.
- 3.2: Chat trigger ("When chat message received").

## How it works

3.1 Vector store builder (7 nodes, including 1 sticky note):

- Google Drive Trigger fires when the watched file changes.
- Google Drive node downloads the PDF as binary.
- Default Data Loader reads the binary and attaches a `file-name` metadata field.
- Recursive Character Text Splitter chunks the text (chunk size 500, overlap 100).
- Embeddings OpenAI vectorizes the chunks; Pinecone Vector Store inserts them into index `ai-agent-rag`.

3.2 RAG chat agent (7 nodes, including 1 sticky note):

- Chat trigger feeds the AI Agent (OpenAI `gpt-4o-mini`).
- The Pinecone Vector Store runs in retrieve-as-tool mode as `climateknowledgebase` (topK 7), sharing the same OpenAI embeddings.
- Simple Memory (buffer window) keeps conversation context between messages.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| Google Drive | `googleDriveOAuth2Api` | Watching and downloading the source document (3.1) |
| OpenAI | `openAiApi` | Embeddings (both) and chat model gpt-4o-mini (3.2) |
| Pinecone | `pineconeApi` | Vector storage and retrieval (index `ai-agent-rag`) |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GOOGLE_FILE_ID` | 3.1 Google Drive Trigger (`fileToWatch`) and Google Drive download (`fileId`) | The Drive file ID of your source document |
| `ai-agent-rag` | Pinecone Vector Store nodes (both workflows) | Your own Pinecone index name (create it first; use matching embedding dimensions) |
| `YOUR_OPEN_AI_API_CREDENTIAL` etc. | All credential blocks | Re-select your own Google Drive / OpenAI / Pinecone credentials |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.
