# PolicySage — HR Policy Q&A Bot

A production-grade multi-agent RAG (Retrieval-Augmented Generation) system that gives employees instant, cited answers to HR policy questions. Built with **AWS Strands Agents SDK**, hybrid search, and real-time streaming.

## Architecture

```
User Query → Input Validator → Router Agent → Retriever Agent → Answer Agent → Validator Agent → Response
```

**Agents:**
- **Router Agent** — Classifies queries as HR or non-HR using Strands SDK + LLM
- **Retriever Agent** — Hybrid search (ChromaDB vector + BM25 keyword + RRF fusion)
- **Answer Agent** — Generates concise, cited answers grounded in policy chunks
- **Validator Agent** — Checks answer groundedness, retries if hallucination detected

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, TypeScript, Tailwind CSS 4, Vite |
| Backend | FastAPI, Uvicorn, Python 3.14 |
| Agent Framework | AWS Strands Agents SDK |
| Vector Store | ChromaDB (persistent) |
| Embeddings | Sentence-Transformers (all-MiniLM-L6-v2) |
| Keyword Search | BM25 (rank-bm25) |
| Database | PostgreSQL (users, sessions, feedback) |
| Auth | JWT (PyJWT + bcrypt) |
| Streaming | WebSocket + HTTP fallback |
| Observability | Langfuse |

## Features

- **Hybrid Search** — Vector similarity + BM25 keyword + Reciprocal Rank Fusion
- **Real-time Streaming** — Token-by-token WebSocket responses
- **Inline Citations** — Every answer cites [document - section]
- **JWT Authentication** — Role-based access (employee, hr_admin, admin)
- **Prompt Injection Guard** — Blocks adversarial inputs
- **PDF Upload** — Ask questions about uploaded documents
- **Feedback Collection** — Thumbs up/down stored in PostgreSQL
- **Rate Limiting** — 20 req/min per user (sliding window)
- **Admin Panel** — Analytics and user management
- **Session History** — Multi-turn conversation context

## Quick Start

### Prerequisites
- Python 3.10+
- Node.js 18+
- PostgreSQL

### Backend

```bash
cd backend
pip install -r requirements.txt
# Configure .env (copy from .env.example)
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Environment Variables

```env
LLM_API_KEY=your-api-key
LLM_BASE_URL=your-openai-compatible-endpoint
MODEL_ID=sonnet
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hr_bot
JWT_SECRET_KEY=your-secret-key
```

### Demo Accounts

| Email | Password | Role |
|-------|----------|------|
| admin@company.com | admin123 | admin |
| hr@company.com | hr1234 | hr_admin |
| employee@company.com | emp123 | employee |

## Project Structure

```
├── backend/
│   ├── agents/          # Router, Retriever, Answer, Validator agents
│   ├── api/             # FastAPI routes (chat, auth, admin, PDF)
│   ├── auth/            # JWT handler, password hashing, dependencies
│   ├── ingestion/       # Document chunking, embedding, BM25 indexing
│   ├── models/          # Pydantic schemas, SQLAlchemy ORM models
│   ├── services/        # Session store, PDF store, injection guard
│   ├── main.py          # FastAPI app entry point
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/  # Chat, Login, Admin, Sidebar, etc.
│   │   ├── hooks/       # useAuth, useChat, useWebSocket, useSession
│   │   └── types/       # TypeScript interfaces
│   └── package.json
├── data/
│   └── policies/        # HR policy markdown documents
└── chroma_data/         # Persistent vector store
```

## How It Works

1. **Ingestion** — Policy markdown files are chunked by section, embedded with sentence-transformers, and stored in ChromaDB + BM25 index
2. **Query** — User asks a question via WebSocket or HTTP
3. **Classification** — Router Agent determines if query is HR-related
4. **Retrieval** — Hybrid search finds top-5 relevant policy chunks
5. **Generation** — Answer Agent produces a cited answer from chunks + history
6. **Validation** — Validator Agent checks groundedness, retries if needed
7. **Streaming** — Response streams token-by-token to the frontend

## Why Strands SDK?

The agent pipeline is **sequential** (classify → retrieve → generate → validate) with no cycles or dynamic branching. Strands provides a clean `Agent(model, prompt)` abstraction orchestrated with plain Python async/await — no graph boilerplate needed.

## License

MIT

## Author

**NagaVenkatesh Arigala** — AI/GenAI Engineer
[LinkedIn](https://www.linkedin.com/in/nv-arigala0801/) · [GitHub](https://github.com/Venki0987)

---

## 📂 About this repository

This is a **documentation and architecture showcase**. It covers the problem, the system design, the agent topology, and the engineering decisions behind the project.

**The source code is held in a private repository.** I'm glad to walk through the implementation in a technical conversation or screen-share — reach out via the links below.

All rights reserved — see [LICENSE](LICENSE).