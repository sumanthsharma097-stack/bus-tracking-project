# bus-tracking-project
BusTrack is a website for live bus tracking. Anyone can search routes and stops, see buses on a map and get arrival times. Operators share GPS and confirm stop departures, admins approve operators and submissions, and passengers save favourites. Built with React, Node and MongoDB.

# BusTrack — Live Bus Tracker

A website for government and private bus services: public route and stop search, live bus positions on a map, arrival estimates, operator trip tracking with real GPS, an admin approval workflow, and passenger favourites.

Stack: React 18 · React Router 6 · Node.js + Express · MongoDB + Mongoose · JWT + bcrypt · Socket.io · Leaflet + OpenStreetMap (free, no API key).

For a full walkthrough of the design and every model's workflow, read **EXPLANATION.md**.

## Run it
Requires Node 18+ and MongoDB running locally.

```bash
# 1. Backend
cd backend
npm install
cp .env.example .env          # Windows: copy .env.example .env
#   then set JWT_SECRET in .env. Generate one with:
#   node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
npm run seed                  # demo data and demo accounts
npm run dev                   

# 2. Frontend (new terminal)
cd frontend
npm install
npm start                     # http://localhost:3000
```
The frontend calls `/api/...`; the `proxy` setting in `frontend/package.json` forwards it to backend


All roles sign in on the same page. If login fails, run `npm run seed` again.

## Environment variables (`backend/.env`)
| Variable | Purpose |
|---|---|
| `PORT`, `MONGO_URI` | Server port and database |
| `JWT_SECRET` | **Required.** The server will not start without it |
| `CLIENT_URL` | Frontend address allowed by CORS |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `MAIL_FROM` | Email for password-reset codes |
| `TRUST_PROXY` | Set to 1 behind a reverse proxy so rate limits see real client addresses |
| `SIMULATE`, `SIM_TICK_MS` | Optional demo mode that moves running buses automatically. Leave `false` for real GPS |

Optional frontend variables: `REACT_APP_TILE_URL` (map tiles), `REACT_APP_API_URL`, `REACT_APP_SOCKET_URL`.

## Trying the features
- **Public:** open website. Routes, stops, route details, arrival times and the live map need no account.
- **Passenger:** sign in as Meena, star a route, set home and work stops, open *My commute*, report how full a bus is.
- **Operator GPS:** sign in as op1, *My buses → Trip controls → Start trip*, allow location. Browsers allow location only on **HTTPS or localhost**; for phone testing use an HTTPS tunnel. Keep the page open while driving.
- **Admin:** sign in as admin, open *Admin → Approvals*, approve op2 and the pending route and bus.
- **Demo without a bus:** set `SIMULATE=true` to see buses move on the map.
- **Password reset:** needs SMTP settings (Gmail: use an App Password). Without them no email is sent and the server console says so. Codes are never printed.
- **Offline:** use a production build (`npm run build`, serve the `build` folder). Open the site once online; viewed routes and stops then open offline.

## Tests
`cd backend && npm test` runs 9 security tests (password reset rules, login rate limit) using an in-memory stand-in for MongoDB.

## Project layout
```
backend/   server.js  seed.js  models/  routes/  middleware/  utils/  test/
frontend/  public/sw.js (offline)  src/{pages,components,context,utils}
docs/      the requirement documents this build follows
HANDOFF.md progress notes
```

## Before you publish
- Put your real contact details in `frontend/src/pages/Terms.js` (`CONTACT`) and have the terms reviewed.
- Serve over HTTPS, set a strong `JWT_SECRET`, change the demo passwords or delete the demo accounts.
- Use a hosted map-tile provider for heavy traffic (see the OpenStreetMap tile usage policy).

## Known limitations
See the limitations section of EXPLANATION.md. Main ones: GPS runs only while the operator's trip page is open; arrival times are straight-line estimates; the whole project was verified with a stand-in database in the build environment, so run through the checklist above on your own MongoDB.
