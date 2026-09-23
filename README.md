# ChatBot ➟ Full-Stack AI Chat Platform

> A production-ready, full-stack AI chat platform powered by **Google Gemini**, with authenticated chat history, streaming responses, context-aware memory, and a polished ChatGPT-like UI.

![Project Screenshot](./frontend/public/readme_file_img.png)

[![Stack](https://img.shields.io/badge/Stack-React%20%7C%20Node.js%20%7C%20MongoDB%20%7C%20Redis%20%7C%20Gemini-blue?style=flat-square)](#tech-stack)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-3178C6?style=flat-square&logo=typescript&logoColor=white)](#tech-stack)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](#quick-start-with-docker-recommended)


---

## Table of Contents

- [About The Project](#about-the-project)
- [Live Demo](#live-demo)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Environment Variables](#environment-variables)
- [Quick Start with Docker (Recommended)](#quick-start-with-docker-recommended)
- [Manual Setup](#manual-setup)
- [Security & Rate Limiting](#security--rate-limiting)
- [Contributing](#contributing)

---

## About The Project

ChatBot is a full-stack ChatGPT-style platform powered by Google Gemini, offering secure, real-time AI conversations with persistent, user-owned chat history.
Users sign in via JWT or Google OAuth to get streaming, context-aware answers and can search, rename, and manage all past chats anytime.
Built with React, Express, MongoDB, Redis, and TypeScript, it pairs a polished responsive UI with a robust, rate-limited API.
Dockerized for one-command setup and ready for Vercel, it is a production-ready base for learning, self-hosting AI product.

---

## Live Demo

**Try it live ➜ [https://chatbot-client-theta.vercel.app](https://chatbot-client-theta.vercel.app/)**

---

## Features

### 🔐 Authentication & User

- **Email + Password** auth with `bcryptjs` hashing and **JWT** (`access_token` + `refresh_token`)
- **Google OAuth 2.0** login via `@react-oauth/google` + `google-auth-library`
- Protected routes — unauthenticated users are redirected to `/login`
- **Refresh-token flow** (`POST /api/v1/auth/refresh`) and logout with token invalidation
- User profile management: get/update name & `customInstructions`, delete account

### 🤖 AI Chat ➝ Gemini Streaming

- Real-time **SSE streaming** (`POST /api/v1/messages/stream` + `/stream/edit`)
- **Server-Sent Events** consumed via `fetch` + `ReadableStream` on the client — token-by-token rendering
- **Abort / Stop generation** support via `AbortController`
- **Edit & regenerate** — edit a prior user message and re-stream the assistant reply (`removeMessagesAfter` logic)
- **File / image attachments** support (inlineData) forwarded to Gemini
- **Function / tool calling** architecture (`src/lib/ai/tools`) — extensible for RAG or external APIs

### 🧠 Context Memory & Token Management

- `tiktoken`-based **TokenCounter** (`src/lib/memory/TokenCounter.ts:1`)
- **ContextCompressor** (`src/lib/memory/ContextCompressor.ts:1`) — prunes trivial messages (`ok`, `thanks`, etc.) and compresses conversation to fit token budget
- **ContextBudget** guard to avoid exceeding model limits
- Chat-level `summary` / `summaryUpdatedAt` fields for long-history summarization

### 💬 Chat Management

- Create, fetch, rename (`PATCH /api/v1/chats/:id`), delete single or **delete all** chats
- **Sidebar** with collapsible layout (desktop) + drawer (mobile) — `src/components/Sidebar.tsx:1` and `src/pages/Home.tsx:1`
- **Search chats** by name (in-memory filter) with modal overlay
- Auto-naming: first user prompt becomes chat title
- Per-chat message history persisted in MongoDB (`Chat` model)

### 🌙 Dark Mode

- Light Mode is enabled by default
- Toggle between Light Mode and Dark Mode


### 🛡️ Infra & DevOps

- **Docker Compose** with 5 services: `mongo` (7), `mongo-express`, `redis` (7-alpine), `backend`, `frontend`
- **Redis-backed rate limiting** (`express-rate-limit` + `rate-limit-redis`)
- **Vercel-ready** (`vercel.json` in both apps) + `Dockerfile` / `Dockerfile.prod` for each service
- **TypeScript** strict mode in both frontend and backend

---

## Tech Stack

### Frontend — `frontend/`

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 19.1 | UI library |
| TypeScript | 5.8 | Type safety |
| Vite | 7.1 | Build & dev server |
| React Router | 7.9 | SPA routing |
| Redux Toolkit + redux-persist | 2.9 / 6.0 | State + persistence |
| Tailwind CSS + @tailwindcss/vite | 4.1 | Styling |
| shadcn/ui + Radix UI | — | Accessible components |
| Axios | 1.12 | HTTP client |
| @react-oauth/google | 0.13 | Google OAuth |
| React Hook Form + Zod + @hookform/resolvers | 7.63 / 4.1 | Forms & validation |
| React Markdown + remark-gfm + Prism.js / highlight.js | 10.1 / 11.11 | Markdown & code highlight |
| React Hot Toast | 2.6 | Notifications |
| lucide-react + react-icons | 0.544 / 5.7 | Icons |

### Backend — `backend/`

| Technology | Version | Purpose |
|------------|---------|---------|
| Node.js | 20 (Alpine) | Runtime |
| TypeScript | 5.8 | Type safety |
| Express | 5.1 | Web framework |
| Mongoose + MongoDB | 8.18 / 7 | ODM & Database |
| ioredis + Redis | 5.11 / 7-alpine | Rate-limit store |
| @google/generative-ai | 0.24 | Gemini SDK |
| google-auth-library | 10.9 | Google token verify |
| jsonwebtoken | 9.0 | JWT |
| bcryptjs | 3.0 | Password hashing |
| express-rate-limit + rate-limit-redis | 8.5 / 6.0 | Throttling |
| tiktoken | 1.0 | Token counting |
| pdf-parse | 1.1 | PDF ingestion |
| morgan, cors, dotenv | — | Middleware & config |
| swagger-ui-express + yamljs | 5.0 / 0.3 | API docs (`swagger.yaml:1`) |

### Infra

| Tool | Purpose |
|------|---------|
| Docker + Docker Compose | Local orchestration (mongo, redis, mongo-express, api, web) |
| Vercel | Frontend + serverless backend deploys |
| MongoDB Atlas | Alternative to local Mongo |
| Upstash / managed Redis | Alternative to local Redis |

---

## Project Structure

```
chatbot/
├── docker-compose.yml          # mongo, mongo-express, redis, backend, frontend
├── .env                        # root compose vars (ROOT_USERNAME, MONGODB_URL, REDIS_PASSWORD)
│
├── backend/
│   ├── src/
│   │   ├── api/v1/
│   │   │   ├── auth/controllers/   # register, login, googleLogin, logout, refreshToken
│   │   │   ├── chat/controllers/   # create, findAll, findSingle, update, remove*
│   │   │   ├── message/controllers/# streamCreate, streamEdit
│   │   │   └── user/controllers/   # getUser, updateMe, updateName, deleteAccount
│   │   ├── config/gemini.ts        # GoogleGenerativeAI singleton
│   │   ├── db/                     # connectDB (mongoose)
│   │   ├── lib/
│   │   │   ├── ai/                 # provider, gemini-provider, tools
│   │   │   ├── auth/               # register/login/refresh/google logic
│   │   │   ├── chat/               # chat service
│   │   │   ├── memory/             # TokenCounter, ContextBudget, ContextCompressor
│   │   │   ├── token/              # JWT helpers
│   │   │   └── user/               # user service
│   │   ├── middlewares/            # authenticate, authenticateRefresh, rateLimiter, index
│   │   ├── models/                 # Chat.ts, User.ts
│   │   ├── redis/                  # client, connectRedis
│   │   ├── routes/index.ts         # all /api/v1/* bindings
│   │   ├── utils/                  # error, hashing, query, types
│   │   ├── validators/             # auth.ts, chat.ts (Zod)
│   │   ├── types/common.ts
│   │   ├── app.ts                  # Express app factory
│   │   └── index.ts                # entry — connectDB/Redis, listen or serverless export
│   ├── swagger.yaml                # OpenAPI 3.0 spec
│   ├── Dockerfile / Dockerfile.prod
│   ├── vercel.json                 # @vercel/node runtime
│   ├── tsconfig.json
│   └── package.json
│
└── frontend/
    ├── src/
    │   ├── api/axiosInstance.ts    # axios baseURL = VITE_BASE_URL
    │   ├── assets/ + public/       # vite.png, readme_file_img.png
    │   ├── components/
    │   │   ├── ui/                 # shadcn primitives
    │   │   ├── sidebar/            # sidebar subcomponents
    │   │   ├── Sidebar.tsx
    │   │   ├── ContentArea.tsx     # message list + ChatInput orchestration
    │   │   ├── ChatInput.tsx       # textarea, file attach, send/stop
    │   │   ├── Message.tsx         # markdown + code highlight + edit
    │   │   ├── ThemeProvider.tsx
    │   │   └── ...
    │   ├── features/
    │   │   ├── auth/authSlice.ts
    │   │   ├── chat/chatSlice.ts   # createMessageStream, editMessageStream (SSE)
    │   │   └── user/userSlice.ts
    │   ├── hooks/useAppStore.ts
    │   ├── lib/utils.ts
    │   ├── pages/                  # Home, Login, Register, Loading, Upgrade
    │   ├── store/store.ts          # persistReducer (auth, chat, user)
    │   ├── types/ / validator/
    │   ├── App.tsx                 # Routes: /, /login, /register, /upgrade
    │   └── main.tsx                # GoogleOAuthProvider + Redux Provider
    ├── components.json             # shadcn config
    ├── vite.config.js              # alias @ → src, tailwind plugin
    ├── index.html
    ├── nginx.conf / Dockerfile*
    ├── vercel.json
    ├── tsconfig.json
    └── package.json
```

---

## API Endpoints

### Auth — public + private

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/auth/register` | — | Register (`name`, `email`, `password`) → `access_token` + `refresh_token` |
| `POST` | `/api/v1/auth/login` | — | Login with email/password |
| `POST` | `/api/v1/auth/google` | — | Google One-Tap / OAuth (`credential` ID token) |
| `POST` | `/api/v1/auth/logout` | Bearer | Invalidate session |
| `POST` | `/api/v1/auth/refresh` | Refresh token | Issue new access + refresh tokens |

### Chat — private (Bearer)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/chats?search=&sort_by=&sort_type=` | List current user chats (search by name, sort) |
| `POST` | `/api/v1/chats` | Create new chat (`{}` → returns `id`, `name: "New Chat"`) |
| `DELETE` | `/api/v1/chats` | Delete **all** chats for user |
| `GET` | `/api/v1/chats/:id` | Fetch single chat with `messages[]` |
| `PATCH` | `/api/v1/chats/:id` | Rename chat (`{ name }`) |
| `DELETE` | `/api/v1/chats/:id` | Delete single chat |

### Message — private (Bearer) — **SSE streaming**

| Method | Endpoint | Body | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/messages/stream` | `{ chatId, prompt, files? }` | Stream assistant reply as `data: { chunk }` + `data: { done, message }` |
| `POST` | `/api/v1/messages/stream/edit` | `{ chatId, messageId, prompt }` | Edit user message & re-stream assistant response |

> Both message endpoints are rate-limited by `aiLimiter` (10 req/min per IP) — `src/middlewares/rateLimiter.ts:35`.

### User — private (Bearer)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/user` | Get profile |
| `PATCH` | `/api/v1/user` | Update `customInstructions` |
| `PATCH` | `/api/v1/user/name` | Update `name` |
| `DELETE` | `/api/v1/user` | Delete account & all chats |

Health: `GET /health` and `GET /` — see `src/app.ts:12`.

Interactive docs: import `backend/swagger.yaml` into Swagger UI or serve via `swagger-ui-express` if mounted.

---

## Environment Variables

> Never commit real secrets. Use `.env.example` as a template and keep `.env` gitignored.

### Root `/.env` (used by `docker-compose.yml`)

| Variable | Example | Purpose |
|----------|---------|---------|
| `ROOT_USERNAME` | `testuser` | Mongo init root user |
| `ROOT_PASSWORD` | `testpassword` | Mongo init root password |
| `DATABASE_NAME` | `Chatbot` | DB name |
| `MONGODB_URL` | `mongodb://testuser:testpassword@mongo:27017/Chatbot?authSource=admin` | Compose-internal Mongo URI |
| `REDIS_PASSWORD` | `defaultpassword` | Redis `requirepass` |

### `backend/.env`

| Variable | Required | Example / Notes |
|----------|----------|-----------------|
| `PORT` | No | `3000` |
| `MONGODB_URL` | **Yes** | Atlas: `mongodb+srv://user:pass@cluster.mongodb.net/Chatbot` · Local compose: `mongodb://testuser:testpassword@mongo:27017/Chatbot?authSource=admin` |
| `ACCESS_TOKEN_SECRET` | **Yes** | Random 32+ char string (JWT HMAC) |
| `GEMINI_API_KEY` | **Yes** | From Google AI Studio (`AQ....` or `AIza...`) |
| `GOOGLE_CLIENT_ID` | **Yes** | OAuth 2.0 Client ID (`xxx.apps.googleusercontent.com`) |
| `REDIS_URL` | Prod | `redis://default:pass@host:17076` (Upstash) — if unset, falls back to `REDIS_HOST`/`REDIS_PORT`/`REDIS_PASSWORD` |
| `REDIS_HOST` | Local | `redis` (compose service name) |
| `REDIS_PORT` | Local | `6379` |
| `REDIS_PASSWORD` | Local | `defaultpassword` |
| `VERCEL` | Auto | Set by Vercel — triggers serverless mode (`src/index.ts:16`) |

### `frontend/.env`

| Variable | Required | Example |
|----------|----------|---------|
| `VITE_BASE_URL` | **Yes** | `http://localhost:3000/api/v1` (local) or deployed API URL |
| `VITE_GOOGLE_CLIENT_ID` | **Yes** | Same as backend `GOOGLE_CLIENT_ID` |

---

## Quick Start with Docker (Recommended)

**Prerequisites:** Docker Desktop (or Docker Engine + Compose plugin), a Gemini API key, and a Google OAuth Client ID.

```bash
# 1. Clone
git clone https://github.com/Mohosin999/chatbot-full-stack-mern-app.git
cd chatbot-full-stack-mern-app

# 2. Create root .env (for compose)
ROOT_USERNAME=testuser
ROOT_PASSWORD=testpassword
DATABASE_NAME=Chatbot
MONGODB_URL=mongodb://testuser:testpassword@mongo:27017/Chatbot?authSource=admin
REDIS_PASSWORD=defaultpassword
EOF

# 3. Backend env (backend/.env )
PORT=3000
MONGODB_URL=mongodb://testuser:testpassword@mongo:27017/Chatbot?authSource=admin
ACCESS_TOKEN_SECRET=replace-with-strong-random-secret-32chars
GEMINI_API_KEY=your_gemini_api_key_here
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=defaultpassword
EOF

# 4. Frontend env (frontend/.env)
VITE_BASE_URL=http://localhost:3000/api/v1
VITE_GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
EOF

# 5. Build & run (all 5 services)
docker compose up --build

# Or detached
docker compose up --build -d
docker compose logs -f backend
```

**URLs after `up`:**

| Service | URL |
|---------|-----|
| Frontend (Vite) | http://localhost:5173 |
| Backend API | http://localhost:3000 · Health: http://localhost:3000/health |
| Mongo Express | http://localhost:8081 (basic auth `admin` / `admin123`) |
| MongoDB | `localhost:27017` |
| Redis | `localhost:6379` |

**Useful commands:**

```bash
docker compose ps
docker compose down          # stop
docker compose down -v       # stop + delete volumes (wipes DB/Redis)
docker compose up --build backend   # rebuild only backend
```

---

## Manual Setup

**Prerequisites:** Node.js 18+ (20 recommended), MongoDB 7+ (local or Atlas), Redis 7+ (local or Upstash), Gemini API key, Google OAuth Client ID.

### 1. Clone & install

```bash
git clone https://github.com/Mohosin999/chatbot-full-stack-mern-app.git
cd chatbot-full-stack-mern-app

# Backend
cd backend
npm ci

# Frontend (in a second terminal)
cd ../frontend
npm ci
```

### 2. Configure env

Create `backend/.env` and `frontend/.env` as shown in [Environment Variables](#environment-variables) — for manual run use an Atlas URI or local `mongodb://127.0.0.1:27017/Chatbot`, and a local Redis or `REDIS_URL`.

### 3. Run

```bash
# Terminal 1 — backend (tsx watch)
cd backend
npm run dev
# → http://localhost:3000  (see src/index.ts:17)

# Terminal 2 — frontend (vite)
cd frontend
npm run dev
# → http://localhost:5173
```

### 4. Build for production

```bash
# Backend
cd backend
npm run build   # tsc → dist/
npm start       # node dist/index.js

# Frontend
cd frontend
npm run build   # vite build → dist/
npm run preview # vite preview
```

---

## Security & Rate Limiting

- **Passwords** hashed with `bcryptjs` (10 rounds) — `src/utils/hashing.ts`
- **JWT** — `ACCESS_TOKEN_SECRET` signs short-lived access tokens; refresh tokens are persisted/validated via `lib/token`
- **Auth middleware** — `authenticate.ts` (Bearer) and `authenticateRefresh.ts` guard private routes
- **CORS** enabled (configure origin for prod), **Morgan** request logging, **trust proxy** for rate-limit behind proxies (`src/app.ts:7`)
- **Redis rate limiters** (`src/middlewares/rateLimiter.ts:1`):

| Limiter | Window | Max | Scope |
|---------|--------|-----|-------|
| `generalLimiter` | 15 min | 100 | All requests |
| `authLimiter` | 15 min | 5 | `/auth/*` |
| `aiLimiter` | 1 min | 10 | `/messages/stream*` |

All limiters use `rate-limit-redis` with `ioredis` and degrade gracefully if Redis is unavailable.

---

## Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feat/your-feature`)
3. Commit (`git commit -m "feat: add your feature"`)
4. Push (`git push origin feat/your-feature`)
5. Open a Pull Request

Please run `npm run lint` (frontend) and `npm run build` (backend) before submitting.

---

## Author

**Mohosin Hasan Akash** — [Portfolio](https://mohosin-hasan-akash.vercel.app/) · mohosin.hasan.akash@gmail.com

If this project helped you, consider giving it a ⭐ on [GitHub](https://github.com/Mohosin999/chatbot-full-stack-app).
