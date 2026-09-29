# Telegram bot deployment

## Deploy on Railway

1. Push this project to a GitHub repository and create a Railway project from that repository.
2. Use Railway's default **Railpack** builder. The included `railway.json` sets the start command to `python bot.py` and health check path to `/`.
3. In **Settings → Networking**, generate a public domain. The app reads Railway's `RAILWAY_PUBLIC_DOMAIN` automatically and uses it to register the Telegram webhook; no `RENDER_EXTERNAL_URL` variable is needed on Railway.
4. Add these service variables under **Variables**:
   - `BOT_TOKEN`: the bot token from BotFather. If it was exposed anywhere, revoke it with BotFather and use the replacement.
   - `MAIN_OWNER_ID`: your Telegram numeric user ID.
   - `WEBHOOK_SECRET`: a random value, 1–256 characters, containing only letters, digits, `_` or `-`. Generate one locally with `openssl rand -hex 32`.
5. Deploy/redeploy. Railway injects `PORT`; the app binds to `0.0.0.0` on that port. The start command must be `python bot.py`—do not set it to `/bin/bash` or `/bin/bash ...`.
6. Stop any existing polling instance that uses the same bot token. Telegram does not allow polling and a webhook to be active at the same time.

Optional variables: `BOT_USERNAME`, `OWNER_NAME`, and `OWNER_LINK`.

### Persistence note

The bot stores its SQLite database and JSON data in the service working directory by default. A Railway redeploy/restart may lose data unless you attach a persistent volume and set `DATA_DIR` to the volume mount path. The service must have permission to write there.

## Deploy on Render

- Create a **Web Service** (not a Background Worker).
- Build command: `pip install -r requirements.txt`
- Start command: `python bot.py`
- Health check path: `/`
- Set `BOT_TOKEN`, `MAIN_OWNER_ID`, and `WEBHOOK_SECRET`; Render supplies `RENDER_EXTERNAL_URL` and `PORT`.
- Stop any existing polling instance using the same token before starting the webhook service.

## Security

Do not commit `.env`, bot tokens, webhook secrets, or runtime database files. Rotate the bot token immediately if it was shared publicly.

## Access control

Single-owner mode is enabled. The configured `MAIN_OWNER_ID` is the only owner; legacy extra-owner and admin IDs are discarded during data normalization, and there is no path to add another owner or administrator.
