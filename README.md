# Chatbot Platform — Backend

## Quickstart
```bash
cd chatbot-platform-backend
npm install
cp .env.example .env   # fill in DATABASE_URL, JWT_SECRET, OPENAI_API_KEY
npx prisma generate
npx prisma migrate dev --name init
npm run dev
```
Health check: http://localhost:3001/health
