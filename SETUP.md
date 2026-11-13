# LuaShield — Setup Guide

## Local Dev

```bash
npm install
cp .env.example .env
# edit .env — set SESSION_SECRET to any random 32+ char string

npx prisma migrate dev --name init
npx prisma db seed   # creates admin/admin123 account
npm run dev
# open http://localhost:3000
```

## Deploy to Railway

1. Push this folder to a GitHub repo
2. New Railway project → "Deploy from GitHub repo"
3. Add env vars:
   - `DATABASE_URL` = `file:./prisma/prod.db`  (or use Railway's Postgres addon — change schema provider to "postgresql")
   - `SESSION_SECRET` = any 40+ char random string
   - `NEXT_PUBLIC_APP_URL` = your Railway URL e.g. `https://luashield.up.railway.app`
4. Railway auto-detects Next.js and runs `npm run build` then `npm start`

## Deploy to Vercel

> Vercel's filesystem is read-only in production — use a real Postgres DB.
> Recommended: Vercel Postgres (free tier) or Supabase.

1. Change `prisma/schema.prisma` datasource provider to `"postgresql"`
2. Push to GitHub
3. Import to Vercel
4. Add env vars in Vercel dashboard:
   - `DATABASE_URL` = your Postgres connection string
   - `SESSION_SECRET` = random 40+ char string
   - `NEXT_PUBLIC_APP_URL` = your Vercel URL
5. After first deploy, run migrations:
   ```bash
   npx prisma migrate deploy
   ```

## First Login

- Username: `admin`
- Password: `admin123`
- **Change the password immediately** (Settings page — or run `npx prisma studio` and edit the hash)

## Usage Flow

1. **Scripts** tab → create a script, paste your Lua source
2. **Scripts** → click your script → **Generate Keys** → set expiry/limit/note
3. **Loader** tab → select your script → copy the generated loader snippet
4. Paste the loader at the top of your distributed script
5. Users run the script → key prompt appears → they enter their key → validated against your server
6. Manage keys any time: disable, reset HWID, delete

## Validate API (direct use)

```
GET /api/validate?key=LS-XXXX-XXXX-XXXX-XXXX&hwid=USERID&script=SCRIPTID
```

Returns:
- `200 OK` — valid, good to go
- `403 INVALID_KEY` — key doesn't exist
- `403 KEY_DISABLED` — key was manually disabled
- `403 KEY_EXPIRED` — past expiry date
- `403 HWID_MISMATCH` — running on wrong machine
- `403 USAGE_LIMIT_REACHED` — used too many times
- `403 WRONG_SCRIPT` — key belongs to different script
