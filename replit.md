# Arabic–German Discord Assistant

This Node.js Discord bot translates Arabic messages into German and corrects German messages before replying with an Arabic translation.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required secret: `DISCORD_BOT_TOKEN` — Discord bot token

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/api-server/src/discord-bot/client.ts` — Discord.js client, intents, message filtering, and replies
- `artifacts/api-server/src/discord-bot/translation.ts` — keyless translation and local German correction rules
- `artifacts/api-server/src/lib/keep-alive.ts` — two-minute loop that pings the local health endpoint
- `artifacts/api-server/src/index.ts` — API server and Discord bot lifecycle
- `HOSTING.md` — Railway, Render, and external monitor setup

## Architecture decisions

- The Discord bot runs alongside the existing API server so the managed service stays observable through its health endpoint.
- The bot uses the Message Content Intent and ignores bot-authored messages before processing them.
- Translations use the keyless Google Translate-compatible endpoint through `@vitalets/google-translate-api`.
- German correction is intentionally local and conservative, covering punctuation, capitalization, spacing, and common mistakes.
- The Express health endpoint is pinged locally every two minutes; true 24/7 uptime requires an always-on hosting plan.

## Product

- Arabic messages remain visible as posted and receive a German translation reply.
- German messages receive a corrected German version and an Arabic translation.
- Messages in other languages, mixed-language messages, and bot messages are ignored.

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- The keep-alive loop and external monitor are not substitutes for always-on hosting when the bot must be continuously online.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
