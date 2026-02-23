# 🧠 AI Learning Assistant

An AI-powered learning tool that processes YouTube videos and PDFs to generate flashcards, quizzes, and enables contextual chat using Retrieval-Augmented Generation (RAG).

---

## 🚀 Features

- 📺 **YouTube Processing** — Extracts and chunks transcripts from YouTube URLs
- 📄 **PDF Processing** — Extracts and embeds text from uploaded PDF files
- 🃏 **Flashcard Generation** — Auto-generates 10–15 structured flashcards from content
- 📝 **Quiz Generation** — Creates 5–10 multiple-choice questions with auto-evaluation
- 💬 **RAG Chat** — Contextual Q&A with streaming responses and chat history
- 📱 **Responsive UI** — Clean, mobile-friendly interface built with TailwindCSS

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14+ (App Router) |
| Backend | Python / FastAPI |
| Database | PostgreSQL / Supabase |
| Vector Store | Supabase pgvector / Pinecone |
| AI Model | OpenAI GPT-4.1 / Gemini |
| Embeddings | text-embedding-3-small / Gemini Embeddings |
| Styling | TailwindCSS |

---

## 📁 Project Structure

```
ai-learning-assistant/
├── frontend/                  # Next.js App
│   ├── app/
│   │   ├── page.tsx           # Home page
│   │   ├── chat/page.tsx      # Chat interface
│   │   ├── flashcards/page.tsx
│   │   └── quiz/page.tsx
│   ├── components/
│   ├── public/
│   └── package.json
│
├── backend/                   # FastAPI App
│   ├── main.py                # Entry point
│   ├── routers/
│   │   ├── video.py           # /process-video
│   │   ├── pdf.py             # /process-pdf
│   │   ├── flashcards.py      # /generate-flashcards
│   │   ├── quiz.py            # /generate-quiz
│   │   └── chat.py            # /chat
│   ├── services/
│   │   ├── embeddings.py
│   │   ├── rag.py
│   │   └── ai.py
│   ├── db/
│   │   └── supabase.py
│   ├── requirements.txt
│   └── .env.example
│
└── README.md
```

---

## ⚙️ Setup Instructions

### Prerequisites

- Node.js 18+
- Python 3.10+
- A [Supabase](https://supabase.com) account
- An [OpenAI](https://platform.openai.com) or [Google Gemini](https://aistudio.google.com) API key

---

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/ai-learning-assistant.git
cd ai-learning-assistant
```

---

### 2. Backend Setup

```bash
cd backend
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file in the `backend/` directory:

```env
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=your_supabase_project_url
SUPABASE_KEY=your_supabase_anon_key
PINECONE_API_KEY=your_pinecone_api_key       # Optional if using Pinecone
GEMINI_API_KEY=your_gemini_api_key           # Optional if using Gemini
```

Start the FastAPI server:

```bash
uvicorn main:app --reload --port 8000
```

---

### 3. Frontend Setup

```bash
cd frontend
npm install
```

Create a `.env.local` file in the `frontend/` directory:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

Start the development server:

```bash
npm run dev
```

The app will be available at `http://localhost:3000`.

---

### 4. Supabase / Database Setup

Run the following SQL in your Supabase SQL editor to enable pgvector and create the embeddings table:

```sql
-- Enable pgvector extension
create extension if not exists vector;

-- Create documents table
create table documents (
  id uuid primary key default gen_random_uuid(),
  source_id text,
  content text,
  embedding vector(1536),
  metadata jsonb,
  created_at timestamp default now()
);

-- Create index for similarity search
create index on documents using ivfflat (embedding vector_cosine_ops);
```

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/process-video` | Submit a YouTube URL for transcript extraction and embedding |
| `POST` | `/process-pdf` | Upload a PDF for text extraction and embedding |
| `POST` | `/generate-flashcards` | Generate 10–15 flashcards from processed content |
| `POST` | `/generate-quiz` | Generate 5–10 MCQs with auto-evaluation |
| `POST` | `/chat` | Send a message and receive a RAG-based streaming response |

### Example Request — Process Video

```json
POST /process-video
{
  "url": "https://www.youtube.com/watch?v=example"
}
```

### Example Request — Chat

```json
POST /chat
{
  "session_id": "abc123",
  "message": "What are the key concepts covered in this video?"
}
```

---

## 🏗️ Architecture Overview

```
User
 │
 ▼
Next.js Frontend
 │
 ▼
FastAPI Backend
 ├── YouTube Transcript API  ──► Chunking ──► Embeddings ──► Supabase pgvector
 ├── PDF Parser (PyMuPDF)    ──► Chunking ──► Embeddings ──► Supabase pgvector
 │
 ├── /generate-flashcards ──► Retrieve chunks ──► GPT-4.1 ──► JSON flashcards
 ├── /generate-quiz       ──► Retrieve chunks ──► GPT-4.1 ──► MCQ JSON
 └── /chat                ──► RAG retrieval   ──► GPT-4.1 ──► Streaming response
```

1. **Ingestion** — Content is extracted from YouTube or PDF, split into chunks (~500 tokens with overlap), and stored as vector embeddings in Supabase pgvector.
2. **Generation** — Flashcards and quiz questions are generated by sending retrieved chunks as context to the LLM with a structured output prompt.
3. **RAG Chat** — User messages are embedded, the top-k most relevant chunks are retrieved, and passed as context to the LLM for a grounded, streaming response.

---

## 🌐 Deployment

### Frontend (Vercel)

```bash
# Push your repo to GitHub, then connect it to Vercel
# Set environment variable in Vercel dashboard:
# NEXT_PUBLIC_API_URL = your deployed backend URL
```

### Backend (Railway / Render / Fly.io)

```bash
# Example for Railway:
railway init
railway up
```

Add your environment variables in the platform's dashboard.

---

## 📋 Environment Variables Summary

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes* | OpenAI API key |
| `GEMINI_API_KEY` | Yes* | Gemini API key (*one of the two) |
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_KEY` | Yes | Supabase anon/service key |
| `PINECONE_API_KEY` | No | Pinecone key (if not using pgvector) |
| `NEXT_PUBLIC_API_URL` | Yes | Backend URL for the frontend |

---

## 📦 Key Dependencies

### Backend
```
fastapi
uvicorn
openai
youtube-transcript-api
pymupdf
langchain
supabase
pgvector
python-dotenv
```

### Frontend
```
next
react
tailwindcss
axios
```

---

## 🎥 Demo

[Watch the demo video](#) — *Link to your 3–5 minute walkthrough*

[Live Demo on Vercel](#) — *Link to your deployed app*

---

## 📄 License

MIT License. See [LICENSE](./LICENSE) for details.