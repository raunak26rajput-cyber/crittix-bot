# Crittix Pterodactyl Bot

Telegram bot that gives users a Pterodactyl panel server (Node.js or Python)
after they earn slots through referrals. Users, referrals and created servers
are stored in local JSON files (`database/users.json`, `database/servers.json`).

## Hosting on Pterodactyl (Node.js server)

1. Create a server on your panel using the **Node.js egg** (import
   `EGGS/nodejs.json`, see below) — or any Node.js server with file access.
2. Upload the contents of this package to the server root (`/home/container`):
   `index.js`, `package.json`, `.env.example`, `EGGS/`, `README.md`.
   Do **not** upload `node_modules` — dependencies install automatically.
3. Create your config: rename `.env.example` to `.env` and fill it in
   (or set the same keys as server startup variables). See the list below.
4. Startup command (the egg's `CMD_RUN` variable):
   `npm start`
   The supplied Node.js egg runs `npm install` automatically before starting.
5. Start the server. On a correct config the console prints `Bot online: @...`.
   If a required variable is missing, the bot stops immediately and prints
   exactly which variable is missing.

The `database/` folder is created automatically on first start and must stay
on the server — it is your only database. Back it up regularly from the
panel's file manager. If a database file ever gets corrupted, the bot saves a
`.corrupt-<timestamp>.bak` copy next to it instead of silently wiping it.

## Environment variables

| Variable | Required | Notes |
|---|---|---|
| `BOT_TOKEN` | yes | New token from @BotFather |
| `PTERODACTYL_URL` | yes | e.g. `https://ptero.devine.name.ng` |
| `PTERODACTYL_API_KEY` | yes | Application API key (Admin → Application API) |
| `OWNER_ID` | no | Default `8033024853` |
| `ADMIN_IDS` | no | Comma-separated Telegram IDs, default = owner |
| `PTERODACTYL_LOCATION_ID` | no | Empty = auto-detect first location; or a location ID / short code |
| `PTERODACTYL_NODEJS_EGG_ID` | no | Empty = find egg by name `NODE JS` |
| `PTERODACTYL_PYTHON_EGG_ID` | no | Empty = find egg by name `Python Engine` |

Secrets are read only from the environment / `.env`. Nothing is hardcoded,
and tokens/keys are never printed in the logs.

## Importing the eggs (one time, on YOUR panel)

The bot creates servers with the eggs named **NODE JS** and **Python Engine**.
Import the supplied files once:

1. Panel → Admin → Nests → pick (or create) a nest.
2. **Import Egg** → upload `EGGS/nodejs.json` → confirm the name is `NODE JS`.
3. **Import Egg** → upload `EGGS/python.json` → confirm the name is `Python Engine`.
4. If your panel already has equivalent eggs under different names, either
   rename them to the names above, or set `PTERODACTYL_NODEJS_EGG_ID` /
   `PTERODACTYL_PYTHON_EGG_ID` to their panel Egg IDs.

Both supplied eggs are PTDL_v2 format (parkervcp yolks images:
`nodejs_22` default for Node, `python_3.12` default for Python) and were
validated as proper JSON.

## How panel creation works

- The bot finds the egg (by ID if configured, otherwise by exact name) and a
  location (configured value, otherwise the first one) through the
  Application API — with pagination, so large panels work too.
- It creates a fresh panel user + server. If server creation fails after the
  user was created, that just-created user is deleted again, so no orphan
  accounts are left behind. Existing users/servers are never touched.
- One creation per Telegram user at a time — double-tapping the egg button
  cannot create duplicates.
- The panel password the user typed is never shown back in chat, and the bot
  tries to delete the password message from the chat history.

## Notes

- Required channel joins: users must join @ZVXAY_ERA
  (owner/admins bypass). Change `REQUIRED_CHANNELS` in `index.js` if needed.
- Referral rule: 5 referrals = 1 server slot; every 5 more = another slot.
  Owner has unlimited slots. `/gift <telegram_id>` (admin) grants a slot.

## Admin Panel (owner + ADMIN_IDS)

Open the bot menu -> **Admin Panel** button (admins also have unlimited
server slots). 23 features, each with its own button:

Stats, All Users, User Lookup (by ID or @username), Top Referrers,
Gift Slot, Gift All +1, Add Points, Remove Points, Ban User, Unban User,
Banned List, All Servers, Broadcast, Set Panel Price (points per panel),
Set Referral Price (points earned per referral), Set First Panel Refs,
Set Menu Photo, Force Channels list, Admin List, Backup Database
(sends the JSON files), Export Users CSV, Delete User (with confirm),
Restart Bot (with confirm). 23 features in total.

Referral economy values are stored in `database/settings.json` and apply
immediately. Banned users are blocked from using the bot (admins exempt).
The classic commands `/stats`, `/gift <id>` and `/broadcast <msg>` still work.
