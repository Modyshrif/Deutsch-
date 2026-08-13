# Hosting the Discord bot outside Replit

The bot is a long-running Node.js service. It needs:

- A process that stays running.
- An externally reachable HTTP port.
- The Discord bot token configured as a secret.
- Message Content Intent enabled in the Discord Developer Portal.

The service exposes a safe health endpoint at:

```text
/api/healthz
```

It returns `{"status":"ok"}` and does not expose secrets.

## Required environment variables

Set these in the host's environment-variable or secrets screen:

| Variable | Required | Description |
| --- | --- | --- |
| `DISCORD_BOT_TOKEN` | Yes | Discord bot token |
| `PORT` | No | Provided automatically by Railway/Render |
| `NODE_ENV` | No | Set to `production` |
| `LOG_LEVEL` | No | Defaults to `info` |

Do not set or upload `OPENAI_API_KEY`; the bot no longer uses OpenAI.

## Railway

1. Create a new Railway project from this repository.
2. Add `DISCORD_BOT_TOKEN` in the service Variables tab.
3. Railway will use `railway.json`:
   - Build: installs dependencies and builds the API service.
   - Start: runs the compiled Node.js service.
   - Health check: `/api/healthz`.
4. Copy the generated public domain for the monitoring URL below.

## Render

1. Create a new Web Service from this repository, or use the included `render.yaml` with a Blueprint.
2. Add `DISCORD_BOT_TOKEN` as a secret environment variable.
3. Use the included settings:
   - Build command: `corepack enable && pnpm install --frozen-lockfile && pnpm --filter @workspace/api-server run build`
   - Start command: `pnpm --filter @workspace/api-server run start`
   - Health check path: `/api/healthz`
4. Copy the generated public domain for the monitoring URL below.

Render's free web services can still spin down when idle. A paid or always-on service is the reliable option for a Discord gateway connection.

## External two-minute monitoring

The application also has an internal keep-alive loop, but an external monitor is more useful because it can wake a sleeping web service. Use a service such as UptimeRobot, Better Uptime, Cronitor, or Freshping:

1. Create an HTTP monitor or heartbeat.
2. Use the public URL:

   ```text
   https://YOUR_PUBLIC_DOMAIN/api/healthz
   ```

3. Set the interval to **2 minutes**.
4. Accept HTTP `200` as the healthy response.
5. Do not add the Discord token or any other secret to the URL or monitor headers.

External monitoring cannot guarantee uptime if the hosting provider blocks wake-ups on its free tier. For continuous Discord availability, use an always-on/Reserved VM plan.

## Local verification

```bash
pnpm install --frozen-lockfile
pnpm --filter @workspace/api-server run build
PORT=8080 pnpm --filter @workspace/api-server run start
curl http://localhost:8080/api/healthz
```
