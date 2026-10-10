---
name: open-lovable
description: Set up, run, and work on Firecrawl's open-lovable, an MIT-licensed Next.js app that chats with an LLM to build React apps, optionally cloning a website via Firecrawl scraping. Use when the user mentions open-lovable, a Lovable.dev clone, recreating a site as a React app, or asks how to configure or deploy it.
---

# open-lovable

Source: https://github.com/firecrawl/open-lovable (MIT, example app by the Firecrawl team).
Stack: Next.js 15.5 (App Router, Turbopack), React 19, Tailwind, Vercel AI SDK. Generated apps run in a sandbox (Vercel Sandbox by default, or E2B).

Before changing or explaining the code, read `review.md` in this skill directory for the security and quality findings from the last review.

## What it does

1. The user gives a prompt or a URL.
2. A URL is scraped through Firecrawl (`/api/scrape-url-enhanced`, `/api/scrape-screenshot`) to get markdown, HTML and a screenshot. Brand styles can be extracted with `/api/extract-brand-styles`.
3. An LLM streams a React/Vite app as file blocks (`/api/generate-ai-code-stream`).
4. The files are written into a sandbox and applied (`/api/apply-ai-code-stream`), packages are installed (`/api/install-packages-v2`, `/api/detect-and-install-packages`), and Vite is restarted (`/api/restart-vite`).
5. The sandbox dev server is previewed in the app (`/api/sandbox-status`, `/api/sandbox-logs`, `/api/check-vite-errors`).

## Setup

Requires Node 18+ and pnpm, npm or bun. The repo has both `pnpm-lock.yaml` and `package-lock.json`; pick one and stick with it.

```bash
git clone https://github.com/firecrawl/open-lovable.git
cd open-lovable
pnpm install
cp .env.example .env.local   # then edit; see below
pnpm dev                     # http://localhost:3000
```

### Environment (`.env.local`)

| Variable | Needed for |
|---|---|
| `FIRECRAWL_API_KEY` | Website scraping (required for URL cloning) |
| `AI_GATEWAY_API_KEY` | Vercel AI Gateway, preferred if set |
| `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY` / `GROQ_API_KEY` | Direct provider keys. At least one is needed. |
| `SANDBOX_PROVIDER` | `vercel` (default) or `e2b` |
| `VERCEL_OIDC_TOKEN` | Vercel Sandbox, from `vercel link` then `vercel env pull` |
| `VERCEL_TEAM_ID`, `VERCEL_PROJECT_ID`, `VERCEL_TOKEN` | Vercel Sandbox, PAT auth (alternative to OIDC) |
| `E2B_API_KEY` | E2B sandbox, when `SANDBOX_PROVIDER=e2b` |
| `MORPH_API_KEY` | Optional "fast apply" for edits |

Never commit `.env.local`. Never paste real keys into chat or into tracked files.

## Working on the code

- Entry points: `app/generation/page.tsx` (main builder UI), `app/api/*/route.ts` (server routes).
- Shared sandbox state is a global (`global.activeSandbox`) that routes such as `run-command` read. It is not per-user.
- Generated-code parsing is regex-based (`<file path="...">` style blocks). Changes to the prompt format must update the parsers in `apply-ai-code*` and `generate-ai-code-stream`.
- `packages/create-open-lovable` is a separate CLI scaffolder that runs `npm install` in the target directory.
- Known gaps: `package.json` defines `test:api`, `test:code` and `test:all`, but there is no `tests/` directory and no `test:integration` script. Don't claim the test suite passes; run `pnpm lint` and `pnpm build` instead.

## Deployment caution

The API routes have no authentication (there is no `middleware.ts`). Anyone who can reach the server can spend the operator's LLM and Firecrawl credits and can drive the sandbox. Run it on localhost or behind an auth layer (for example Vercel Deployment Protection or a reverse proxy with auth). Do not expose it publicly as-is.

## Commands

```bash
pnpm dev      # dev server with Turbopack
pnpm build    # production build; run before claiming a change works
pnpm start    # serve the production build
pnpm lint     # ESLint via next lint
```
