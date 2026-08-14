# Sage — Setup & Deployment Guide

## Prerequisites

- Python 3.10+
- Node.js 18+
- PostgreSQL 14+
- Git

---

## 1. Clone the Repository

```bash
git clone https://github.com/sujal-31/Saige.git
cd Saige
```

---

## 2. Backend Setup

### Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

### Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
LLM_API_KEY=your-openai-compatible-api-key
LLM_BASE_URL=https://your-llm-proxy-endpoint
MODEL_ID=sonnet

DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hr_bot

JWT_SECRET_KEY=your-random-secret-key-here
JWT_ALGORITHM=HS256
JWT_EXPIRY_HOURS=24

APP_HOST=127.0.0.1
APP_PORT=8000
FRONTEND_ORIGIN=http://localhost:5173

# Optional — Langfuse observability
LANGFUSE_SECRET_KEY=sk-lf-xxx
LANGFUSE_PUBLIC_KEY=pk-lf-xxx
LANGFUSE_HOST=https://cloud.langfuse.com
```

### Create the Database

```bash
psql -U postgres -c "CREATE DATABASE hr_bot;"
```

Tables are auto-created on first startup.

### Start the Backend

```bash
cd ..
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

The backend will:
1. Load environment variables
2. Initialize the embedding model (first run downloads ~80MB)
3. Ingest policy documents from `data/policies/`
4. Create database tables and seed demo users
5. Start serving on http://localhost:8000

### Verify Backend is Running

```bash
curl http://localhost:8000/
# → {"message":"HR Policy Q&A Bot API","version":"1.0.0"}
```

---

## 3. Frontend Setup

### Install Dependencies

```bash
cd frontend
npm install
```

### Start the Frontend

```bash
npm run dev
```

Frontend runs on http://localhost:5173

### Demo Login Credentials

| Email | Password | Role |
|-------|----------|------|
| admin@company.com | admin123 | admin |
| hr@company.com | hr1234 | hr_admin |
| employee@company.com | emp123 | employee |

---

## 4. Production Deployment

### Option A: Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - LLM_API_KEY=${LLM_API_KEY}
      - LLM_BASE_URL=${LLM_BASE_URL}
      - MODEL_ID=sonnet
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/hr_bot
      - JWT_SECRET_KEY=${JWT_SECRET_KEY}
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "5173:80"

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: hr_bot
      POSTGRES_PASSWORD: postgres
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
docker-compose up --build
```

### Option B: Manual Deployment

**Backend (any Linux server / AWS EC2 / Railway):**

```bash
pip install -r backend/requirements.txt
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000 --workers 2
```

**Frontend (Vercel / Netlify / Cloudflare Pages):**

```bash
cd frontend
npm run build
# Deploy the dist/ folder
```

Update the WebSocket URL in `frontend/src/hooks/useWebSocket.ts`:
```typescript
const DEFAULT_WS_URL = 'wss://your-backend-domain.com/api/chat/stream';
```

And the API URL in `frontend/src/hooks/useAuth.ts`:
```typescript
const API_URL = 'https://your-backend-domain.com/api/auth';
```

### Option C: AWS Deployment

| Component | AWS Service |
|-----------|-------------|
| Backend | ECS Fargate / EC2 / Lambda (with Mangum) |
| Frontend | S3 + CloudFront |
| Database | RDS PostgreSQL |
| Vector Store | ChromaDB on EFS or use OpenSearch |
| LLM | Bedrock / SageMaker endpoint |

---

## 5. Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `LLM_API_KEY` | ✅ | API key for OpenAI-compatible LLM endpoint |
| `LLM_BASE_URL` | ✅ | Base URL for LLM proxy |
| `MODEL_ID` | ✅ | Model identifier (e.g., "sonnet", "gpt-4") |
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `JWT_SECRET_KEY` | ✅ | Secret for signing JWT tokens |
| `JWT_ALGORITHM` | ❌ | Default: HS256 |
| `JWT_EXPIRY_HOURS` | ❌ | Default: 24 |
| `APP_HOST` | ❌ | Default: 127.0.0.1 |
| `APP_PORT` | ❌ | Default: 8000 |
| `LANGFUSE_SECRET_KEY` | ❌ | Langfuse observability (optional) |
| `LANGFUSE_PUBLIC_KEY` | ❌ | Langfuse observability (optional) |
| `LANGFUSE_HOST` | ❌ | Default: https://cloud.langfuse.com |

---

## 6. Troubleshooting

**Backend won't start:**
- Check that PostgreSQL is running: `pg_isready`
- Verify `.env` has `LLM_API_KEY` and `MODEL_ID` set
- Ensure port 8000 is free: `lsof -i :8000`

**Frontend can't connect:**
- Backend must be on port 8000 (CORS is configured for localhost:5173)
- Hard refresh browser (Ctrl+Shift+R) after backend restart
- Check browser DevTools Console for WebSocket errors

**"Service not responding" in chat:**
- Kill any zombie Python processes on port 8000
- Restart backend fresh
- Check that the LLM proxy endpoint is reachable

**Embedding model download slow:**
- First startup downloads all-MiniLM-L6-v2 (~80MB)
- Set `HF_TOKEN` env var for faster Hugging Face downloads
