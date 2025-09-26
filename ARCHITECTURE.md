# Chatbot Platform — Architecture & Design (Minimal)

## Overview
A 2-service setup:
- **Backend**: Node.js (Express + TypeScript), JWT auth, Prisma ORM (PostgreSQL/Neon), LLM integration (OpenAI Responses API primary, OpenRouter fallback), file upload streaming to OpenAI Files API.
- **Frontend**: Next.js App Router + Tailwind + Zustand for simple auth state and pages (`/` login/register, `/chat`, `/upload`).

## Core Entities
- **User** (id, email, password hash)
- **Project** (belongs to user)
- **Prompt** (versioned; latest active prompt used as system prompt)
- **Chat** (array of message JSON entries; persisted per project)

## Key Flows
1. **Auth**: Email/password → bcrypt hash → JWT (`Authorization: Bearer <token>`). Middleware sets `req.userId`.
2. **Projects & Prompts**: CRUD; only owner can access. Only one active prompt at a time.
3. **Chat**: 
   - Build system prompt from active prompt (or default).
   - Call **OpenAI Responses API** (`/v1/responses`) with `gpt-4o-mini`. If unavailable, fallback to **OpenRouter** (`/api/v1/chat/completions`).
   - Persist user+assistant messages to `chats.messages` JSON array.
4. **File Uploads** (Good-to-have): `POST /api/projects/:id/files` with `multipart/form-data`. Proxies to **OpenAI Files API** with `purpose=assistants` and returns the file metadata.

## Non-Functional Considerations
- **Scalability**: 
  - Stateless API; horizontal scale behind load balancer. 
  - Postgres (Neon serverless) scales independently.
  - Move sessions/ratelimits to Redis if needed.
- **Security**:
  - Helmet + CORS; JWT with 7d expiry.
  - Hash passwords with bcrypt (cost=12).
  - Validate ownership on all project/file/chat actions.
  - Never log secrets; store envs in managed secret stores.
- **Extensibility**:
  - Add tables: analytics, usage events, roles (RBAC), webhooks.
  - Add providers by creating new adapters next to `chat.ts`.
- **Performance**:
  - Use streaming responses for lower latency (future improvement).
  - Cache system prompts per project.
- **Reliability**:
  - Graceful error handling; 4xx vs 5xx.
  - Prisma migrations; use pgbouncer/connection pooling in prod.

## Suggested Deployment
- **DB**: Neon serverless Postgres.
- **Backend**: Render/Railway/Fly. Set env: `DATABASE_URL`, `JWT_SECRET`, `OPENAI_API_KEY`, optionally `OPENROUTER_API_KEY`.
- **Frontend**: Vercel/Netlify. Set `NEXT_PUBLIC_API_URL` to your backend URL.

## API Summary
- `POST /api/auth/register { email, password, name? }`
- `POST /api/auth/login { email, password }`
- `GET /api/projects`
- `POST /api/projects { name, description? }`
- `GET /api/projects/:projectId`
- `POST /api/projects/:projectId/prompts { content }`
- `POST /api/chat { projectId, message }`
- `POST /api/projects/:projectId/files (multipart/form-data)`
