# Crystal View Chatbot (n8n workflow)

Crystal View Chatbot is an n8n workflow that implements a Telegram-based AI assistant for Crystal View Real Estate. The assistant answers knowledge-base-driven questions, helps schedule consultations with a senior consultant (Max) via Google Calendar, and uses vector search to respond from a curated knowledge base.

This repo contains:
- `crystal_view_chatbot.sanitized.json` — the exported n8n workflow with all credentials removed (import into n8n to run).
- `README.md` — setup, configuration, architecture, and usage guidance.
- `CONTRIBUTING.md` — contribution guidelines and security rules.
- `LICENSE` — MIT license.
- `.gitignore` — ignores common secrets and local files.

Core features
- Telegram-triggered conversational assistant.
- Knowledge-base retrieval (Pinecone vector index) with embeddings (Google Gemini / PaLM).
- Language model for chat (Groq or substitute LLM).
- Booking integration (Google Calendar) to schedule meetings with a human agent.
- Google Drive integration to ingest documents into the knowledge base.
- Session memory (n8n memory buffer) to maintain chat context.

High-level architecture
- Telegram Trigger -> AI Agent node (LangChain wrapper)
  - Tools used by the agent:
    - Pinecone Vector Store (retrieve knowledge)
    - Google Gemini embeddings (embeddings creation)
    - Groq Chat Model (LLM responses)
    - Google Drive (knowledge ingestion / document download)
    - Google Calendar (event creation for bookings)
  - Memory buffer for per-chat session context
- Final response delivered via n8n Telegram node

Security & secrets (IMPORTANT)
- The exported workflow in this repo has had all credential entries removed. You must create credentials inside your n8n instance and attach them to the nodes after importing the workflow.
- Before publishing or sharing any additional exports:
  - Never commit tokens, client secrets, or private keys.
  - Use n8n's built-in credential storage for runtime secrets.
  - If any tokens or keys were previously committed, rotate them immediately.

Prerequisites
- n8n (self-hosted or n8n cloud) — tested with n8n versions that support LangChain nodes
- A Telegram Bot and its token (BotFather)
- Pinecone account & index (vector index name used in workflow: `crystal001` — you can rename)
- Google Cloud account & APIs for:
  - Google Drive (to download source docs)
  - Google Calendar (event creation)
  - Google PaLM / Gemini embeddings access (if using Google embeddings)
- Groq API account (or your preferred LLM provider; the workflow references a Groq chat model)
- Access to the training / knowledge documents to seed the Pinecone index

Setup & configuration (import & connect)
1. Import the workflow into n8n:
   - n8n UI -> Workflows -> Import from file -> choose `crystal_view_chatbot.sanitized.json`.
2. Create the required credentials in your n8n instance:
   - Telegram Bot (token)
   - Google Drive OAuth2
   - Google Calendar OAuth2
   - Pinecone API key & index
   - Google PaLM / Gemini API key (if using)
   - Groq API key (or substitute your LLM provider)
3. Attach credentials to the corresponding nodes in the n8n editor.
4. Set the Pinecone index name (if you renamed it) in both the indexing and retrieval nodes.
5. Update the Google Calendar node to point at the correct calendar ID if needed.

Seeding the knowledge base
- Use the `Download file` node and the `Default Data Loader` + `Embeddings` + `Pinecone Vector Store` nodes to ingest documents.
- Typical steps:
  1. Place knowledge base documents (PDF/MD/DOCX) into a Drive folder.
  2. Use n8n to download and chunk documents (the included data loader node is wired for this).
  3. Generate embeddings and insert into Pinecone index `crystal001` (or your chosen name).

Testing
- With the workflow active and credentials configured:
  1. Send a message to your Telegram bot.
  2. Observe n8n workflow execution and logs.
  3. Confirm that the AI Agent searches the vector store (Pinecone) and returns responses.
  4. Test booking flow: ask to schedule a viewing; provide required booking fields and confirm the Google Calendar event is created.

Behavior & compliance notes
- The AI Agent node includes instructions for Fair Housing compliance and escalation rules (e.g., when to book a meeting with Max). Review the node's prompt text if you want to customize tone or escalation triggers.

Contributing
- Please open issues or PRs to suggest improvements (sanitization helpers, sample documents, better architecture diagrams).
- When contributing, avoid adding sensitive data; use placeholders for credentials.

License
- MIT. See LICENSE file for full text.

Contact / Maintainer
- Owner: Crystal View Real Estate / mbk-ai
