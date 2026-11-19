# HANDOFF — read this first if a session resets

Spec: docs/MASTER_PROMPT.md (features) + docs/DESIGN_ENGINEERING_RULES.md (design/engineering rules).
Workflow: build in steps, each step ends with a full runnable zip. User says go / no go per step.

## Done
- Step 1: core fixes
  - API proxy to :5001 via CRA proxy, single axios instance (utils/api.js)
  - Login/register: normalised email, field-level validation messages, duplicate email message, safe 500s, JWT_SECRET required
  - Router v7 future flags set, SVG favicon, free OSM tiles (utils/mapConfig.js)
  - Public pages (no login): Home, Live map (Socket.io merge), Routes (+filter), Route detail (+ETA per stop), Stops
  - Common login page with role redirect (admin -> /admin). Admin page is READ-ONLY overview for now.
  - New design system (teal, IBM Plex Sans/Mono, Lucide icons, reduced motion), responsive layouts
  - Regex search input escaped. CORS restricted to CLIENT_URL. Simulator off by default.

## Not done yet (in order)
2. Public pages polish + responsive QA at real device sizes, seed check
3. Roles/approvals: operator `approved` status, pending bus/route submissions (`status: pending|approved|rejected`), admin approve/reject UI + backend guards, admin edit of buses/routes/stops, user list
4. Operator dashboard: add/update own buses & routes, ownership checks, real GPS (navigator.geolocation, ~10s PATCH /buses/:id/location, error states), Confirm Departure (departure timestamps, duplicate guard), validate lat/lng
5. Passenger: home stop / daily route (backend prefs endpoints already exist: GET/PATCH /api/auth/preferences), login prompt for visitors, favourites, crowd vote (POST /api/buses/:id/vote exists), live socket on route detail, offline mode (public/sw.js exists and is registered)
6. Forgot password with hashed expiring email OTP + rate limiting; /terms page + footer link
7. Final: responsive/security pass, EXPLANATION.md (workflow of each model), final README

## Known facts
- Never tested against a live MongoDB in the sandbox (download blocked). Frontend production build compiles. Backend files pass node --check.
- Operator accounts currently self-approve (no pending state) until step 3.
- Future (not to be built): operator mobile PWA improvements, push notifications.
