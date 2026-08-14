# PolicySage â€” HR Policy Q&A Bot

A production-grade multi-agent RAG (Retrieval-Augmented Generation) system that gives employees instant, cited answers to HR policy questions. Built with **AWS Strands Agents SDK**, hybrid search, and real-time streaming.

## Architecture

```
User Query â†’ Input Validator â†’ Router Agent â†’ Retriever Agent â†’ Answer Agent â†’ Validator Agent â†’ Response
```

**Agents:**
- **Router Agent** â€” Classifies queries as HR or non-HR using Strands SDK + LLM
- **Retriever Agent** â€” Hybrid search (ChromaDB vector + BM25 keyword + RRF fusion)
- **Answer Agent** â€” Generates concise, cited answers grounded in policy chunks
- **Validator Agent** â€” Checks answer groundedness, retries if hallucination detected

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

- **Hybrid Search** â€” Vector similarity + BM25 keyword + Reciprocal Rank Fusion
- **Real-time Streaming** â€” Token-by-token WebSocket responses
- **Inline Citations** â€” Every answer cites [document - section]
- **JWT Authentication** â€” Role-based access (employee, hr_admin, admin)
- **Prompt Injection Guard** â€” Blocks adversarial inputs
- **PDF Upload** â€” Ask questions about uploaded documents
- **Feedback Collection** â€” Thumbs up/down stored in PostgreSQL
- **Rate Limiting** â€” 20 req/min per user (sliding window)
- **Admin Panel** â€” Analytics and user management
- **Session History** â€” Multi-turn conversation context

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
â”œâ”€â”€ backend/
â”‚   â”œâ”€â”€ agents/          # Router, Retriever, Answer, Validator agents
â”‚   â”œâ”€â”€ api/             # FastAPI routes (chat, auth, admin, PDF)
â”‚   â”œâ”€â”€ auth/            # JWT handler, password hashing, dependencies
â”‚   â”œâ”€â”€ ingestion/       # Document chunking, embedding, BM25 indexing
â”‚   â”œâ”€â”€ models/          # Pydantic schemas, SQLAlchemy ORM models
â”‚   â”œâ”€â”€ services/        # Session store, PDF store, injection guard
â”‚   â”œâ”€â”€ main.py          # FastAPI app entry point
â”‚   â””â”€â”€ requirements.txt
â”œâ”€â”€ frontend/
â”‚   â”œâ”€â”€ src/
â”‚   â”‚   â”œâ”€â”€ components/  # Chat, Login, Admin, Sidebar, etc.
â”‚   â”‚   â”œâ”€â”€ hooks/       # useAuth, useChat, useWebSocket, useSession
â”‚   â”‚   â””â”€â”€ types/       # TypeScript interfaces
â”‚   â””â”€â”€ package.json
â”œâ”€â”€ data/
â”‚   â””â”€â”€ policies/        # HR policy markdown documents
â””â”€â”€ chroma_data/         # Persistent vector store
```

## How It Works

1. **Ingestion** â€” Policy markdown files are chunked by section, embedded with sentence-transformers, and stored in ChromaDB + BM25 index
2. **Query** â€” User asks a question via WebSocket or HTTP
3. **Classification** â€” Router Agent determines if query is HR-related
4. **Retrieval** â€” Hybrid search finds top-5 relevant policy chunks
5. **Generation** â€” Answer Agent produces a cited answer from chunks + history
6. **Validation** â€” Validator Agent checks groundedness, retries if needed
7. **Streaming** â€” Response streams token-by-token to the frontend

## Why Strands SDK?

The agent pipeline is **sequential** (classify â†’ retrieve â†’ generate â†’ validate) with no cycles or dynamic branching. Strands provides a clean `Agent(model, prompt)` abstraction orchestrated with plain Python async/await â€” no graph boilerplate needed.

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