# Chatbot Platform (Minimal)

Monorepo:
- `chatbot-platform-backend` — Node/Express/Prisma/JWT + OpenAI Responses API (fallback: OpenRouter)
- `chatbot-platform-frontend` — Next.js/Tailwind/Zustand

## Run Locally
1. **Backend**
```bash
cd chatbot-platform-backend
npm install
cp .env.example .env   # fill DATABASE_URL (Neon), JWT_SECRET (32+ chars), OPENAI_API_KEY, optional OPENROUTER_API_KEY
npx prisma generate
npx prisma migrate dev --name init
npm run dev
```
Health check: http://localhost:3001/health

2. **Frontend**
```bash
cd ../chatbot-platform-frontend
npm install
echo "NEXT_PUBLIC_API_URL=http://localhost:3001" > .env.local
npm run dev
```
- Login/Register at: `http://localhost:3000`
- Chat UI: `http://localhost:3000/chat`
- File Upload: `http://localhost:3000/upload`

## Deploy (Quick)
- **Neon**: create DB, get `DATABASE_URL`.
- **Backend (Render/Railway/Fly.io)**:
  - Build command: `npm i && npx prisma generate && npm run build`
  - Start command: `node dist/index.js`
  - Envs: `DATABASE_URL`, `JWT_SECRET`, `OPENAI_API_KEY`, optional `OPENROUTER_API_KEY`, `USE_OPENAI_RESPONSES=true`
- **Frontend (Vercel)**:
  - Env: `NEXT_PUBLIC_API_URL=https://<your-backend-host>`
  - Build: default Next.js build

## Notes
- Responses API is primary; if not configured, OpenRouter used as fallback.
- File uploads go to OpenAI Files API with `purpose=assistants` and echo metadata.
- Extend easily with streaming, analytics, roles, and more providers.
