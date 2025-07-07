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

![MCP Server Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/5.%20AI_Agent_Mcp/mcp_server.png)

![MCP Agent Workflow](https://github.com/GhufranBarcha/n8n_ai_agents/blob/main/5.%20AI_Agent_Mcp/mcp_agent.png)

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

## Getting Started

To use these workflows:

1. Import the JSON files into your n8n instance
2. Configure the required credentials (API keys, OAuth tokens)
3. Set up any necessary external services (Pinecone, Gmail, Twitter API, etc.)
4. Test the workflows with sample data
5. Deploy and monitor your AI agents

## Technologies Used

- **n8n**: Workflow automation platform
- **AI Models**: OpenAI GPT-4o-mini, Groq LLaMA 3 70B
- **Vector Database**: Pinecone
- **APIs**: Gmail, YouTube, Twitter, Instagram
- **Protocols**: MCP (Model Context Protocol)
- **Storage**: Google Drive integration

## Contributing

Feel free to fork this repository and submit pull requests with improvements or new workflow ideas!