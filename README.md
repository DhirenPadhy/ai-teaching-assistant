TeachAI — AI Teaching Assistant
Faculty-facing dashboard to generate teaching materials (slides, quizzes, examples, infographics, summaries) with AI, manage a content library, track feedback, and chat with a pedagogical AI assistant.

Run & Operate
pnpm --filter @workspace/api-server run dev — run the API server (port 8080, proxied at /api)
pnpm --filter @workspace/ai-teaching-assistant run dev — run the frontend (Vite, proxied at /)
pnpm run typecheck — full typecheck across all packages
pnpm --filter @workspace/api-spec run codegen — regenerate API hooks and Zod schemas from OpenAPI spec
pnpm --filter @workspace/db run push — push DB schema changes (dev only)
Required env: DATABASE_URL — Postgres connection string; SESSION_SECRET — express session
Stack
pnpm workspaces, Node.js 24, TypeScript 5.9
Frontend: React 18 + Vite 7, Wouter (routing), TanStack Query, Framer Motion, Tailwind CSS v4, shadcn/ui
API: Express 5 (port 8080)
DB: PostgreSQL + Drizzle ORM
AI: OpenAI via Replit AI Integrations proxy (no user API key required)
Validation: Zod (zod/v4), drizzle-zod
API codegen: Orval (from OpenAPI spec in lib/api-spec/openapi.yaml)
Where things live
lib/api-spec/openapi.yaml — source-of-truth API contract
lib/api-spec/orval.config.ts — codegen config (indexFiles: false, no schemas key)
lib/db/src/schema/ — DB schemas: materials.ts, feedback.ts, conversations.ts, messages.ts
artifacts/api-server/src/routes/ — Express routers: materials/, feedback/, openai/
artifacts/ai-teaching-assistant/src/pages/ — React pages: dashboard, generate, library, feedback, assistant
artifacts/ai-teaching-assistant/src/index.css — color system (primary #381932, bg #FFF3E6)
Architecture decisions
Contract-first API: OpenAPI spec → Orval codegen → typed React Query hooks + Zod schemas. Never hand-write fetch calls.
SSE streaming for AI generation endpoints (/materials/generate, /feedback/analyze, /openai/conversations/:id/messages) — not wrapped in hooks, streamed via raw fetch + ReadableStream.
lib/api-zod/src/index.ts must stay as a single export * from "./generated/api" line — Orval regenerates it and stale exports cause TS2308 duplicate-export errors.
Sidebar uses dark plum (#381932) with cream text; main area uses cream background — two-tone branded layout.
All AI calls go through Replit AI Integrations proxy — no API key env var needed in code.
Product
Dashboard — stats overview (total materials, avg rating, subjects, trend), quick-generate shortcuts, recent materials list, feedback category breakdown
Generate — pick type (slide/quiz/example/infographic/summary), enter subject + topic, stream AI-generated content in real time
Library — browse/search/filter all materials by type, click to preview full content, delete materials
Feedback Hub — submit class feedback (subject, category, 1-5 rating, notes), AI-powered improvement analysis via streaming, feedback history, per-category bar charts
AI Assistant — multi-session chat with a pedagogical AI, streaming responses, session management
User preferences
Colors: primary #381932 (dark maroon), background #FFF3E6 (warm cream)
Font: Geometr231 BT (DM Sans used as fallback)
Clean, minimalist professional dashboard; no emojis in UI
Icons: lucide-react throughout
Gotchas
Orval codegen: indexFiles: false on zod output, no schemas key in config — prevents TS2308 duplicate exports
Run codegen after any OpenAPI spec change: pnpm --filter @workspace/api-spec run codegen
Do NOT call service ports directly — all traffic goes through shared proxy at localhost:80
SSE endpoints: do not use React Query hooks for them; use raw fetch + ReadableStream
Pointers
pnpm-workspace skill — workspace structure, TypeScript setup, codegen conventions
ai-integrations-openai skill — Replit AI proxy setup (no user API key needed)
