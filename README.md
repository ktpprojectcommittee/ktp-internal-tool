# KTP Internal Tool

An internal web app for KTP members at UChicago to share coursework, collect course feedback, and query uploaded materials using an AI chatbot.

---

## What It Does

**Course Dashboard**
The main page of the app. Displays all courses that KTP members have previously uploaded material to or left feedback on. Each course card shows aggregate feedback (ratings, difficulty, workload). From this dashboard, members can upload new files (PDFs, DOCX, TEX, etc.) tagged to a specific course, and browse or download files others have shared.

**Course Feedback**
Members can leave structured reviews on any course: overall rating, difficulty, estimated weekly workload, and free-text comments. Feedback is tied to a specific course and quarter, and surfaced on the dashboard so future members can make informed decisions when picking classes.

**RAG Chatbot**
A second page housing a retrieval-augmented generation (RAG) chatbot. Members can ask questions in natural language and the LLM pulls context from the uploaded coursework files to answer. Useful for quickly finding info across past assignments, lecture notes, and exams without manually digging through files.

---

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | Next.js 14 + TypeScript | App Router, SSR, great DX |
| Styling | Tailwind CSS | Utility-first, fast to build |
| Backend / DB / Auth / Storage | Supabase | All-in-one: PostgreSQL, file storage, auth, auto-generated APIs — no custom backend needed |
| Database | PostgreSQL (via Supabase) | Relational, supports full-text search, row-level security |
| File Storage | Supabase Storage | S3-compatible, CDN-backed, RLS policies for access control |
| Auth | Supabase Auth | Email/password with verification; can restrict to @uchicago.edu |
| RAG LLM | OpenAI API + pgvector | OpenAI for generation; pgvector (PostgreSQL extension) stores vector embeddings of uploaded files for semantic search/retrieval |
| Hosting | Vercel (frontend) + Supabase Cloud (backend) or subdomain on current KTP website (talk to Paulina) | Zero-config deploys, managed infra |

---

## Why Supabase?

We chose Supabase because it collapses the typical backend stack (database + auth + file storage + API layer) into a single managed platform. This means:

- No custom backend server to write or maintain
- Database APIs are auto-generated from our schema
- Row-Level Security (RLS) handles authorization at the database level — e.g., "only authenticated users can upload"
- Built-in file storage with CDN, so uploads/downloads are fast
- Runs on PostgreSQL, so we get full SQL power and can add **pgvector** for vector embeddings without adding another service

---

## How the RAG Chatbot Works

1. When a file is uploaded, it gets chunked into smaller passages and each passage is embedded into a vector using OpenAI's embedding model
2. Embeddings are stored in PostgreSQL via the **pgvector** extension
3. When a member asks a question, the query is embedded and a similarity search runs against the stored vectors to find the most relevant passages
4. Those passages are sent as context to the OpenAI chat model, which generates an answer grounded in the actual coursework material

---

## Project Structure (Planned)

```
ktp-internal-tool/
├── src/
│   ├── app/                    # Next.js pages (App Router)
│   │   ├── page.tsx            # Course dashboard
│   │   ├── chat/page.tsx       # RAG chatbot
│   │   └── ...
│   ├── components/             # Reusable UI components
│   └── lib/
│       ├── supabase.ts         # Supabase client setup
│       └── openai.ts           # OpenAI client setup
├── supabase/
│   └── migrations/             # DB schema migrations
├── .env.local                  # Environment variables (not committed)
└── README.md
```

---

## Getting Started

1. Clone the repo
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env.local` and fill in your Supabase and OpenAI keys
4. Run: `npm run dev`
5. Open `http://localhost:3000`
