# bus-tracking-project
BusTrack is a website for live bus tracking. Anyone can search routes and stops, see buses on a map and get arrival times. Operators share GPS and confirm stop departures, admins approve operators and submissions, and passengers save favourites. Built with React, Node and MongoDB.


# BusTrack — Project Explanation

*For explaining the project to your partner and mentor. Everything here matches the code in this zip.*

---

## 1. What BusTrack is, in one minute

BusTrack is a website where **anyone** can look up bus routes and stops, see where buses are right now on a map, and see how many minutes until the next bus reaches their stop. **Operators** (bus drivers or companies) use their phone's GPS to share their bus's position and confirm departures from each stop. An **admin** approves operators and anything new they submit. **Passengers** can sign in to save a home stop, a work stop and favourites, and to report how crowded a bus is.

Design principle: **public information needs no login**. Login is only for personal features and for managing buses.

**Problem it solves:** in many cities there is no reliable way to know when the next bus will arrive. BusTrack gives that, using only a browser: no app install, no paid map API.

---

## 2. Architecture

```
 Browser (React)                                  Server (Node + Express)                 Database
 ┌───────────────────────────┐   HTTPS /api/...   ┌──────────────────────────────┐        ┌─────────┐
 │ Pages: Home, Map, Routes, │ ─────────────────► │ routes/  (REST API)          │ ─────► │ MongoDB │
 │ Stops, My commute,        │ ◄───────────────── │ middleware/ (JWT, roles)     │ ◄───── │ 5 colls │
 │ Operator, Admin, Auth     │                    │ utils/ (ETA, mailer)         │        └─────────┘
 │                           │   WebSocket        │                              │
 │ Leaflet map (OSM tiles)   │ ◄═════════════════ │ Socket.io: pushes bus updates│
 │ Service worker (offline)  │                    └──────────────────────────────┘
 └───────────────────────────┘
```

- **REST API** for everything that is a request/response (lists, saving, login).
- **WebSocket (Socket.io)** for live changes, so the map and route pages update without refreshing.
- **MongoDB** stores five kinds of documents (the models below).
- **Leaflet + OpenStreetMap** draw the map. No API key, no cost.
- **Service worker** lets already-viewed routes and stops open offline.

### Roles

| Role | How they get it | What they can do |
|---|---|---|
| Visitor (no account) | n/a | Browse routes, stops, route details, ETAs, live map |
| Passenger | Register | Everything above + home/work stop, favourites, crowd reports |
| Operator | Register as operator, **then an admin must approve** | Manage **their own** buses and routes, run trips, share GPS, confirm departures |
| Admin | Created by the seed script only; **cannot** be created by registering | Approve operators, buses and routes; edit or delete any record; manage stops |

Role checks happen **on the server** (`middleware/auth.js`). Hiding a button in the page is never the only protection.

---

## 3. The five models and their workflows

### 3.1 User

**What it is:** an account.

| Field | Meaning |
|---|---|
| `name`, `email` (unique, lowercase) | Identity. Email is lowercased so `Meena@x.com` and `meena@x.com` are the same account |
| `password` | Stored only as a **bcrypt hash**, never as text |
| `role` | `passenger`, `operator` or `admin` |
| `approvalStatus` | `pending`, `approved`, `rejected`. Operators start `pending`; everyone else is `approved` |
| `homeStop`, `workStop` | References to Stop |
| `favoriteStops`, `favoriteRoutes` | Lists of references |

**Workflow**
1. **Register** (`POST /api/auth/register`): the server validates each field and returns a message per field. The client may only ask for passenger or operator; a request for "admin" is silently treated as passenger. Operators are saved as `pending`.
2. **Login** (`POST /api/auth/login`): same page for all roles. The server compares the password hash and returns a JWT (valid 7 days) plus `{name, email, role, approvalStatus}`. The page then sends admins to `/admin`, operators to `/operator`, passengers home.
3. **Staying signed in:** the token is kept in the browser. On every page load the app asks `/api/auth/me` for the current role and approval, so an operator approved since yesterday sees it immediately.
4. **Operator approval:** an admin approves or rejects (Admin → Approvals). Until approved, the operator sees a banner and the server refuses every management action.
5. **Preferences:** `GET/PATCH /api/auth/preferences` read and save home/work stops and favourites. The server checks that every ID is valid.
6. **Forgot password** (see 5.7).

### 3.2 Stop

**What it is:** a physical bus stop.

Fields: `name`, `code` (unique, optional), `location {lat, lng}`, `address`, `city`.

**Workflow:** created, edited and deleted **only by admins** (Admin → Stops). Everyone can read and search stops (`GET /api/stops?q=`). A stop is shared by many routes.

### 3.3 BusRoute

**What it is:** a numbered service such as "500A", defined by an **ordered list of stops**. The order is the direction of travel, so *the position of a stop in the list is what makes ETA and "next stop" possible*.

Fields: `routeNumber` (unique), `name`, `type` (government/private), `operator` (company name), `stops` (ordered), `fare`, `firstBus`, `lastBus`, `frequency` (timetable info), `color`, `isActive`, `approvalStatus`, `createdBy`.

**Workflow**
1. **Creation.** Admin: saved as `approved` immediately. Approved operator: saved as `pending` with `createdBy` set to them. A pending route is **invisible to the public**.
2. **Review.** Admin → Approvals → Approve or Reject. Approved routes appear in lists, search and the map.
3. **Update.** The owner (or an admin) can edit an existing route, including stop order, **without** a new approval. Another operator cannot (403).
4. **Public reading.** `GET /api/routes` (filter by type or search), `GET /api/routes/:id` (route + its running buses), `GET /api/routes/:id/eta/:stopId`.

### 3.4 Bus

**What it is:** one vehicle on one route, with its live state.

| Field | Meaning |
|---|---|
| `busNumber` (unique), `route`, `operator` | Which vehicle, which route, **who owns it** |
| `location {lat,lng}`, `speed`, `lastUpdated` | Latest position only (no history is stored) |
| `status` | `idle`, `active` (trip running), `breakdown`, `completed` |
| `stopIndex`, `currentStop`, `nextStop` | Where on the route it is, by stop order |
| `occupancy` | `empty`, `moderate`, `crowded`, `full` |
| `departures[]` | Operator-confirmed: `{stopIndex, stop, departedAt}` per stop left |
| `tripStartedAt` | When the current trip began |
| `crowdVotes[]` | Passenger reports (see 5.5) |
| `approvalStatus`, `createdBy` | Approval, like routes |

**Lifecycle of a bus**

```
 operator adds bus ──► pending ──admin approves──► approved, status = idle
                                   │
                     operator: Start trip
                                   ▼
        status = active, stopIndex = 0, departures = []
                │   every ~10 s: GPS position ──► location updated ──► pushed live
                │   at each stop: Confirm departure ──► departures[i] saved, next stop moves on
                ▼
       operator: End trip ──► status = completed ──► removed from public "running" lists
```

Only the **owning operator or an admin** can start/stop trips, send positions, confirm departures or edit the bus. The bus must be `approved`.

### 3.5 PasswordReset

**What it is:** a short-lived record for one password-reset attempt. It stores **only hashes**: the OTP (bcrypt) and the reset token (sha256). Details in 5.7.

### How the models connect

```
User ──owns──► Bus ──runs on──► BusRoute ──has ordered──► Stop
 │                                                         ▲
 └── homeStop / workStop / favourites ─────────────────────┘
 └── favoriteRoutes ──► BusRoute
```

---

## 4. The six improvements and where they live

| # | Improvement | Where in code |
|---|---|---|
| 1 | **WebSocket live updates** instead of polling | `server.js` (rooms `route-<id>`), `utils/socket.js`, `RouteDetail.js`, `LiveMap.js` |
| 2 | **GPS-driven movement** (real operator GPS; optional simulator for demos) | `TripControl.js`, `PATCH /api/buses/:id/location`, `utils/simulator.js` (`SIMULATE=true`) |
| 3 | **Distance-based ETA** with rush-hour factor | `utils/haversine.js`, `GET /api/routes/:id/eta/:stopId` |
| 4 | **Favourites + My commute** | `PrefsContext.js`, `MyCommute.js`, `/api/auth/preferences` |
| 5 | **Crowd reports** | `POST /api/buses/:id/vote`, vote chips on `RouteDetail.js` |
| 6 | **Offline mode** | `public/sw.js`, offline banner in `Layout.js` |

(Future item #7 is in section 9.)

---

## 5. End-to-end workflows

### 5.1 Passenger checks when the bus comes
1. Opens the site (no login). Home shows running buses, route and stop counts from the database.
2. Opens a route, taps their stop.
3. Browser calls `GET /api/routes/:id/eta/:stopId`.
4. Server finds buses on that route that have **not passed** the stop and computes, for each:
   - distance = straight-line (haversine) from the bus's live position to its next stop + the sum of stop-to-stop distances up to the chosen stop;
   - speed = 25 km/h × rush-hour factor (0.6 at 8–10am and 5–8pm, 1.2 at night 10pm–5am, otherwise 1.0);
   - ETA minutes = distance ÷ speed (at least 1).
5. The page lists buses by soonest, with distance, stops away and crowding. While the page is open, the WebSocket keeps it updating (ETA refreshed at most every 5 seconds).

### 5.2 Operator runs a trip (the real-GPS path)
1. Operator signs in (must be approved) → **My buses** → **Trip controls**.
2. **Start trip** → `POST /api/buses/:id/trip/start`. Bus becomes `active` at stop 0.
3. Browser asks for location permission (`navigator.geolocation.watchPosition`). Every 10 seconds the latest fix is sent with `PATCH /api/buses/:id/location`.
4. The server checks: signed in, owns the bus, bus approved, coordinates valid and in range, speed sane. It saves the position and **emits it to everyone watching** that route (and to the global map).
5. Passengers' maps and route pages update immediately.
6. Failures are **shown, never faked**: permission denied, no GPS signal, device can't locate, insecure (non-HTTPS) page, server unreachable (with a retry count), or the server refusing (stops sharing).
7. **End trip** → status `completed`; the bus disappears from public lists.

### 5.3 Manual departure confirmation (works alongside GPS)
- The trip page lists stops in order; the one that is due has a **Confirm departure** button.
- `POST /api/buses/:id/depart {stopIndex}`. The server only accepts it if `stopIndex` equals the number of departures already recorded, and it records it with **one atomic database update**. So a double-tap, a slow connection or a stale screen cannot record the same departure twice or skip a stop; the server answers 409.
- The actual time is saved. Passengers see "Left Town Hall at 10:42 (**confirmed**)", clearly separate from estimated arrival times.

### 5.4 Approval workflow
```
Operator registers ─► pending ─► Admin approves ─► can manage buses
                              └► Admin rejects  ─► blocked (banner explains)

Approved operator adds bus/route ─► pending (hidden from public) ─► Admin approves ─► public
Approved operator edits own existing bus/route ─► takes effect immediately (no approval)
Admin edits anything ─► immediately
```
Admin → Approvals shows the three queues with a count badge. Seed data includes a pending operator, route and bus so you can demo it.

### 5.5 Crowd reports
- Signed-in passenger taps how full a bus is (4 levels). Visitors get a "sign in" prompt.
- Server accepts it only for a running, approved bus. One vote per user per bus (a new one replaces theirs); the last 50 are kept; only votes from the **last 30 minutes** count; the level with the most votes wins; if levels tie, the latest voter's choice wins when it is one of the tied levels. The result is saved as the bus's occupancy and pushed live.
- The operator can also set occupancy from the trip page.

### 5.6 Favourites and My commute
- Star on routes and stops; "Set as home / work" on stops. Visitors get a sign-in dialog and return to where they were after signing in.
- **My commute** shows: next buses at the home stop and work stop (ETA of every route serving the stop), routes that go **from home to work** in the right direction (home appears before work in the stop order), and favourites.

### 5.7 Forgot password (email code)
1. **Request:** user enters email. The server answers the *same message* whether or not the email exists. If it exists, a 6-digit code is generated with a cryptographically secure generator and emailed. Only a **bcrypt hash** is stored.
2. **Verify:** code expires in **5 minutes**, **5 wrong tries** allowed (counted atomically), works **once**. A correct code is exchanged for a random reset token (stored hashed, valid **10 minutes**).
3. **Reset:** the token plus the new password. The token is found-and-deleted in one step so it is **single use**; the password is hashed with bcrypt.
4. Protections: rate limits on each step, a 60-second resend wait, nothing stored in the browser, the code never logged or returned in a response, mail credentials only in `.env`. Works for all roles.

### 5.8 Offline mode
A service worker saves the app files and **only public, non-live data** (routes and stops). Offline, previously viewed routes and stops still open and a banner explains that live positions and ETAs are unavailable. Accounts, preferences, admin/operator data, live buses and ETAs are deliberately **never** cached.

---

## 6. API summary

| Area | Endpoints |
|---|---|
| Auth | `POST /api/auth/register`, `/login`; `GET /me`; `GET/PATCH /preferences`; `POST /forgot-password`, `/verify-otp`, `/reset-password` |
| Routes | `GET /api/routes`, `/:id`, `/:id/eta/:stopId`, `/mine` (operator); `POST /`, `PUT /:id` (operator/admin, ownership checked); `DELETE /:id` (admin) |
| Buses | `GET /api/buses`, `/:id`, `/mine`; `POST /`, `PUT /:id`; `PATCH /:id/location`; `POST /:id/trip/start`, `/trip/stop`, `/depart`, `/vote`; `DELETE /:id` (admin) |
| Stops | `GET /api/stops`, `/:id`; `POST`, `PUT /:id`, `DELETE /:id` (admin) |
| Admin | `GET /api/admin/pending`, `/users`, `/routes`, `/buses`; `PATCH /users/:id/approval`, `/routes/:id/approval`, `/buses/:id/approval` |
| Live | Socket.io: client sends `watchRoute` / `unwatchRoute`; server emits `busUpdate` (that route) and `globalBusUpdate` (map) |

---

## 7. Security measures

- Passwords: bcrypt hashes. Tokens: JWT signed with a secret that must be set (the server refuses to start without it).
- Role and ownership checks on the server for every protected endpoint; admins cannot self-register; role is never trusted from the client.
- Pending and rejected buses/routes are filtered out of every public endpoint.
- Rate limits: login, registration, and each password-reset step. Resend cooldown on codes.
- Generic responses for forgotten passwords (no account discovery); OTP and reset token stored hashed, expiring, single use.
- Input validation on server (and friendly messages in the form); coordinates, speed and IDs validated; search text escaped before use in queries.
- CORS limited to the frontend address; basic security headers; secrets only in `.env`.
- Offline cache excludes all private data.

---

## 8. How this was tested, honestly

**Done:**
- 9 automated backend tests (`npm test`): password-reset rules (generic replies, hashes only, single use, attempt cap, expiry, cooldown, rate limits) and login rate limit.
- Permission rules (pending/rejected operators blocked; owner vs other operator vs admin) unit-tested.
- Every page, for every kind of user, checked in a real headless browser at phone (360 px), tablet (820 px) and laptop (1280 px) widths: 48 combinations, **no sideways scrolling, no JavaScript errors**.
- Browser flow tests: login/register errors, approvals, bus editing, GPS sharing with permission granted and denied, network loss, departure confirmation and duplicate rejection, favourites, visitor sign-in prompts, live push updates, voting, forgot-password, offline reload.
- The production frontend build compiles without warnings.

**Limits you should state to your mentor:**
- The build environment had **no MongoDB**, so those browser tests ran against a stand-in API with the same response shapes. Run the README checklist on a real MongoDB to confirm the database side (seeding, saving, the atomic departure guard).
- Real email sending was not tested (needs your SMTP settings).
- Real phone GPS was tested only with simulated browser coordinates.
- Map tiles could not load in the test environment (blocked network); they load normally on a normal connection.

---

## 9. Limitations and future work

**Current limitations**
- GPS shares only while the operator's trip page is **open** in the browser; closing the tab stops updates (the public page then shows "updated N min ago").
- ETA uses straight-line distance and an average speed, not real roads or live traffic.
- Only the latest position is stored, so there is no trip history or analytics.
- Reset does not sign out sessions already issued (tokens last 7 days).
- Phone GPS testing requires HTTPS.

**Future work (not built)**
- **#7: Operator phone app/PWA** — an installable operator view with a screen wake-lock and background-friendly tracking so GPS keeps sending when the phone locks; automatic stop detection when the bus enters a stop's radius.
- Push notifications ("your bus is 5 minutes away").
- Road-based ETA using a routing service and traffic data.
- Position history for reports and timetable accuracy; admin analytics.
- Session revocation/refresh tokens; account deletion and data export; email verification.
- End-to-end tests against a real database in CI.

---

## 10. Demo script (5 minutes)

1. **As a visitor:** Home → search "500A" → open route → tap a stop → show ETA and crowding → Live map.
2. **Show the login prompt:** tap the star → "Sign in to continue".
3. **As Meena (passenger):** set home/work stops → My commute → report crowding.
4. **As admin:** Admin → Approvals → approve operator `op2`, the pending bus and route (count badge drops). Show Users and Buses tabs.
5. **As op1 (operator):** My buses → Trip controls → Start trip (allow location) → Confirm departure → show passengers' page updating live in a second window.
6. **Forgot password** (if email is configured) and the **Terms** page.
7. Turn the network off: previously viewed routes still open with the offline banner.

---

## 11. Questions your mentor might ask

- **Why WebSockets instead of refreshing?** Updates arrive instantly and the server isn't hit by every client polling every few seconds.
- **Why is the stop order stored in the route?** It defines direction, "next stop", stops-away and the home→work check.
- **Why two kinds of arrival information?** Estimates are calculated; confirmed departures are facts entered by the operator. They are shown separately so passengers aren't misled.
- **Why approvals?** Anyone can register as an operator; approval stops fake buses and routes appearing publicly.
- **Can an operator edit someone else's bus?** No: the server compares the bus owner with the signed-in user and returns 403.
- **What stops a double departure?** The server accepts only the next expected stop and applies it in one atomic update.
- **Is the OTP stored?** No, only a hash; it expires in 5 minutes, allows 5 tries and works once.
- **What if GPS fails?** The operator sees the exact reason; the system never invents positions.
