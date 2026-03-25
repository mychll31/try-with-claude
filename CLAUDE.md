## Claude Flow Instructions

### Project Status
This project is already in active development. Do NOT start from scratch.

### Before Writing Any Code
1. Read and understand the existing codebase in mockchat-be/ and mockchat-fe/ and always say "Hi Boss"
2. Store existing conventions to memory bank:
   - Current migrations in mockchat-be/database/migrations/
   - Existing API endpoints in mockchat-be/routes/api.php
   - Existing services: GroqChatService, OpenAIChatService, OllamaChatService
   - Existing models: Conversation, Message, CustomerType
   - Vue components in mockchat-fe/src/views/
   - API wrapper in mockchat-fe/src/api/chat.ts
   - Pinia stores in mockchat-fe/src/stores/
   - Naming conventions already used in the project
3. Follow ALL existing patterns — do not introduce new conventions
4. Do not overwrite or break any existing functionality
5. New code must integrate seamlessly with what already exists

### Memory & Coordination Rules
- Initialize memory bank so all agents share the same context
- Use wiring gap matrix to ensure backend API endpoints
  match frontend API calls in mockchat-fe/src/api/chat.ts
- Every agent must read from memory before writing code
- Every agent must save their output to memory after completion

---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MockChat is a customer service & sales training app. Trainees chat with AI-simulated Filipino customers (Tagalog replies) and practice Facebook Ads scenarios. Two active apps share this repo:

- `mockchat-be/` — Laravel 12 backend (PHP 8.2+)
- `mockchat-fe/` — Vue 3 + TypeScript frontend (Vite + Pinia + Vue Router)
- `mockchat-be-legacy/` and `mockchat-fe-legacy/` — archived plain-PHP + single-HTML versions (not actively developed)

## Commands

### Backend (`mockchat-be/`)

```bash
composer run dev          # Start all: php artisan serve + queue + pail + vite
php artisan serve         # API only at http://localhost:8000
php artisan migrate
php artisan db:seed --class=CustomerTypeSeeder
composer run test         # Run PHPUnit tests
php artisan test --filter TestName  # Run a single test
vendor/bin/pint           # Lint/format PHP
```

First-time setup:
```bash
composer run setup        # install, copy .env, key:generate, migrate, npm install, build
```

### Frontend (`mockchat-fe/`)

```bash
npm run dev               # Dev server at http://localhost:5173 (proxies /api → :8000)
npm run build             # Type-check + build to dist/
npm run lint              # oxlint + eslint (both with --fix)
npm run test:unit         # Vitest unit tests
npm run test:e2e          # Playwright e2e tests
```

## Architecture

### Backend

The API is stateless JSON (`routes/api.php`). All routes are under `/api/chat/*` and handled by `ChatController`.

**Request flow for a chat message:**
1. `ChatController::sendMessage` saves the agent's message, fetches full message history
2. Passes history + customer personality to `OpenAIChatService::getCustomerReply` → OpenAI `gpt-4o-mini`
3. Saves the customer reply, then calls `ChatStageDetection::fromAgentMessages` (keyword regex) to detect which of the 7 training stages the agent has reached
4. Returns `{ agent_message, customer_response, suggested_stage }` to the frontend

**Key classes:**
- `GroqChatService` / `OpenAIChatService` / `OllamaChatService` — all implement `ChatServiceInterface`; provider selected via `CHAT_PROVIDER` env var (`groq` is default, free). Each falls back to a static Tagalog message if the API key is missing.
- `ChatStageDetection` — pure static keyword-regex detector for the 7-step sales flow (Greeting → Probing → Empathize → Solution → Value → Offer → Confirmation)
- Models: `Conversation` (has `customer_type_id`, `customer_name`, `status`), `Message` (has `sender` enum: `agent`|`customer`, `body`), `CustomerType` (`type_key`, `label`, `personality`)

**API endpoints** (`routes/api.php`, all under `/api/chat/`):
- `GET /new` — Creates conversation, returns first AI customer opener message
- `POST /send` — Sends agent message, returns `{ agent_message, customer_response, suggested_stage }`
- `GET /messages` — Full message history for a conversation
- `GET /conversation` — Conversation metadata
- `POST /end` — Close a conversation
- `GET /stats` — Global counts (total conversations, messages, breakdown by customer type)

**DB:** SQLite (`database/database.sqlite`) by default; configure MySQL in `.env` for production.

### Frontend

Vite proxies `/api` → `http://127.0.0.1:8000` in dev. In production set `VITE_API_URL=http://your-api-host`.

- `src/api/chat.ts` — typed fetch wrapper (`chatApi.*`) for all backend endpoints
- `src/router/index.ts` — two routes: `/` (ChatView) and `/ads` (AdsView)
- `src/stores/` — Pinia stores (only the example `counter.ts` exists; active state is local `ref()` in views)
- `src/views/ChatView.vue` — main training chat UI
- `src/views/AdsView.vue` — Facebook Ads practice; contains 4 hardcoded scenarios and a local scoring algorithm (0–100 pts) that evaluates objective, targeting, budget, and ad copy fields
- `App.vue` — global header with live stats; uses `provide('refreshStats', fn)` so child views can trigger a header stat refresh after ending a conversation

### 7-Step Sales Training Flow

The training enforces this sequence: **Greeting → Probing → Empathize → Solution → Value → Offer → Confirmation**. `ChatStageDetection` detects the current stage from cumulative agent message text using keyword regexes (mix of English and Tagalog). The frontend receives `suggested_stage` (1–7) on each reply and can display progress.

### Customer Types

Three types seeded via `CustomerTypeSeeder`: `normal_buyer`, `irate_returner`, `irate_annoyed`. A random type is assigned on each new conversation.
