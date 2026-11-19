# BusTrack — Live Bus Tracker (build step 1)

React 18 + Node/Express + MongoDB. Free OpenStreetMap tiles, no API keys.

## Run it
Requires Node 18+ and MongoDB running locally.

```bash
# 1. Backend
cd backend
npm install
cp .env.example .env          # then set JWT_SECRET (see comment inside)
npm run seed                  # demo data + demo accounts
npm run dev                   # http://localhost:5001

# 2. Frontend (new terminal)
cd frontend
npm install
npm start                     # http://localhost:3000
```
The frontend calls `/api/...`; the `proxy` setting in `frontend/package.json` forwards it to port 5001.
Windows: if `cp` fails use `copy .env.example .env`.


All three sign in on the same page. If login fails, run `npm run seed` again: it recreates the accounts with hashed passwords.

## Environment variables (backend/.env)
`PORT`, `MONGO_URI`, `JWT_SECRET` (required, server refuses to start without it), `CLIENT_URL`, `SIMULATE` (demo movement, default false), `SIM_TICK_MS`.
Optional frontend: `REACT_APP_TILE_URL`, `REACT_APP_API_URL`, `REACT_APP_SOCKET_URL`.

## Docs
`docs/MASTER_PROMPT.md` and `docs/DESIGN_ENGINEERING_RULES.md` are the requirements this project is built against. `HANDOFF.md` tracks progress.
