# MASTER DEVELOPMENT PROMPT — BUS TRACK LIVE BUS TRACKER

## Project: BusTrack — Live Bus Tracker

I have an existing website project called **BusTrack — Live Bus Tracker**. I want you to inspect, debug, improve, and complete this existing project.

This is NOT a request to build a completely different website from scratch. Work on my existing codebase, preserve its existing functionality, and fix the problems described below.

**Most importantly: Do not remove, simplify, or change any of my existing requirements or intended workflows.** If an existing feature is partially implemented or broken, repair it rather than deleting it.

---

# 1. Understand the Existing Project Before Making Changes

My current technology stack:

- Frontend: React 18
- Routing: React Router 6
- Backend: Node.js + Express
- Database: MongoDB + Mongoose
- Authentication: JWT + bcryptjs
- Maps: Leaflet + React-Leaflet
- Map data: OpenStreetMap-compatible map tiles
- HTTP requests: Axios

Existing project structure:

```text
bus-tracker/
├── backend/
│   ├── models/
│   │   ├── User
│   │   ├── Stop
│   │   ├── BusRoute
│   │   └── Bus
│   ├── routes/
│   │   ├── auth
│   │   ├── stops
│   │   ├── busRoutes
│   │   └── buses
│   ├── middleware/
│   │   ├── JWT authentication
│   │   ├── Admin guards
│   │   └── Operator guards
│   ├── seed.js
│   └── server.js
└── frontend/
    └── src/
        ├── pages/
        │   ├── Home
        │   ├── LiveMap
        │   ├── RoutesList
        │   ├── RouteDetail
        │   ├── StopSearch
        │   ├── AdminPanel
        │   ├── Login
        │   └── Register
        ├── components/
        │   └── Layout
        ├── context/
        │   └── AuthContext
        └── utils/
            └── Axios API instance
```

Existing backend configuration:

```env

```

Existing demo data:
- 5 routes
- 15 stops
- 7 buses

Existing demo accounts mentioned in the README:

```text
demo user id ,passwords
```

These are existing README credentials, not guaranteed working credentials. Verify whether the accounts actually exist in the database and whether their passwords and roles are correct. Fix the seed and authentication process if necessary. Never bypass authentication just to make these credentials work.

Existing pages and intended functionality:

| Page | Function |
|---|---|
| Home | Live statistics, route search, active buses |
| Live Map | Live map with bus markers |
| Routes List | Browse and filter routes |
| Route Detail | Bus information, stops and ETA |
| Stop Search | Search stops and see associated routes |
| Admin Panel | Admin management |
| Login | Common login for all roles |
| Register | Account registration |

Existing API functionality should also be preserved wherever possible.

---

# 2. Overall Project Objective

BusTrack should be a functional, responsive, real-time bus tracking platform for government and private bus services.

Users should be able to:

- View available buses and routes.
- Search bus routes and stops.
- View bus timings.
- View current bus locations on a map.
- View estimated arrival times.
- See stops along a route.
- Save their home bus stand.
- Save their daily travelling route.
- Access personalized saved information after login.

Bus operators should be able to manage their buses and routes after admin approval.

Admins should control operator approvals and newly submitted buses/routes.

The website must have working frontend-backend integration, proper database operations, secure authentication, and a reliable live tracking system.

---

# 3. Responsive Website Design

One major problem is that the website does not display properly on mobile devices.

The current mobile appearance looks like a scaled-down desktop website rather than a properly responsive mobile layout.

### Problems to fix

- Text appears too large or zoomed.
- Buttons and navigation do not resize properly.
- Some elements do not fit the screen.
- The map and sidebar layout are unsuitable for mobile.
- Content may be clipped or require horizontal scrolling.
- Desktop layout is not adapting correctly.

### Expected design

**Desktop:**
- Use the existing desktop navigation.
- Keep the map and sidebar properly aligned.
- Maintain suitable content spacing and readable typography.

**Mobile:**
- Use a compact navigation layout.
- Make buttons and text appropriately sized.
- Make the map fit the available screen width.
- Stack, collapse, or adapt sidebars appropriately.
- Keep important actions easy to access.
- Avoid horizontal scrolling and unwanted zooming.

**Tablet:**
- Use an intermediate responsive layout.

Apply responsiveness to ALL pages, including Home, Live Map, Routes, Route Details, Stop Search, Login, Register, and Admin Panel.

Do not fix only the homepage.

Preserve the existing visual identity, theme, and useful design elements.

---

# 4. Fix the Map and Missing API Key Problem

The current map displays repeated text:

```text
API KEY REQUIRED
carto.com/basemaps/api-key
```

The map appears partially loaded, with missing or restricted map tiles.

The current project uses Leaflet, and the README mentions CARTO Dark tiles with OpenStreetMap.

### Required actions

1. Inspect the existing Leaflet tile-layer configuration.
2. Identify which tile provider is currently being used.
3. Check whether the current provider requires an API key.
4. Fix the configuration properly.

If retaining CARTO:
- Configure the correct tile URL and required key.
- Store configuration appropriately.
- Follow the provider's current usage requirements.

If using standard OpenStreetMap tiles:
- Use a valid tile URL.
- Include proper attribution.
- Follow the tile provider's usage policy.

Do not blindly replace the existing map provider without checking the current implementation.

Do not introduce unnecessary paid services.

The final map must support:
- Bus markers.
- Bus movement.
- Route paths.
- Stop markers.
- Current bus locations.
- ETA-related information.
- Map zoom and navigation.

Do not break existing Leaflet functionality while fixing the tile problem.

---

# 5. Fix React, Frontend and Backend Errors

The browser console currently displays these warnings and errors:

```text
React Router Future Flag Warning:
v7_startTransition

React Router Future Flag Warning:
v7_relativeSplatPath

Error Component Stack:
AuthProvider (AuthContext.js)
App

Failed to load resource:
favicon.ico 404

POST http://localhost:3000/api/auth/login 401 (Unauthorized)

POST http://localhost:3000/api/auth/register 400 (Bad Request)
```

### Fix each issue

**A. React Router warnings**

- Inspect the installed React Router version.
- Check the current route configuration.
- Apply compatible future flags where appropriate.
- Verify that navigation and nested routes work correctly.
- Do not simply hide warnings.

**B. Favicon**

- Add a proper BusTrack favicon.
- Ensure the frontend references the correct favicon path.
- Remove the 404 error.

**C. Login 401 error**

Inspect:
- Whether the account exists.
- Whether the submitted email and password are correct.
- Whether passwords are hashed and compared correctly.
- Whether the request payload matches the backend.
- Whether JWT generation and validation work.
- Whether role information is correctly returned.

Do not bypass authentication or return fake success responses.

**D. Registration 400 error**

Inspect:
- Frontend registration fields.
- Backend validation.
- Required fields.
- MongoDB schema.
- Duplicate email handling.
- Password validation.
- API request payload.

Show meaningful validation messages to users instead of generic errors.

**E. Frontend/backend connection**

The README specifies:

```text
Backend: http://localhost:5001
Frontend: http://localhost:3000
```

However, the browser currently sends authentication requests to:

```text
http://localhost:3000/api/auth/login
http://localhost:3000/api/auth/register
```

Inspect the Axios base URL, frontend proxy configuration, and backend server configuration.

Ensure requests actually reach the backend running on port 5001, either through a correctly configured proxy or an appropriate API base URL.

Do not hardcode inconsistent API URLs across different components.

---

# 6. One Common Login Page for All Roles

I originally intended to have ONE common login page for all account types.

The current build appears to have removed the admin login option or functionality.

This must be corrected.

The common login system must support:

1. Normal users/passengers.
2. Bus operators.
3. Admins.

### Expected behaviour

- All account types use the same login page.
- Authentication identifies the account's actual role.
- After successful login, redirect the user to the appropriate dashboard.
- Passengers should access their personal features.
- Approved operators should access operator management features.
- Admins should access the admin dashboard.

Use proper role-based access control.

Do not create unnecessary separate login pages for each role.

Do not remove or disable admin functionality.

---

# 7. Public Access Without Mandatory Login

This is an important original requirement.

Visitors must NOT be forced to register or log in just to browse public transport information.

The following features must be publicly accessible:

- Available bus routes.
- Bus timings.
- Bus stops.
- Route details.
- Current bus locations.
- Bus information.
- ETA information.
- Route search.
- Stop search.
- Live map.

### Login should be required only for personal features such as:

- Saving a home bus stand.
- Saving a daily travelling route.
- Viewing and managing saved personal preferences.

If a visitor tries to save a personal preference, show an appropriate login/register prompt.

Do not put authentication barriers in front of publicly available transport information.

---

# 8. Admin Dashboard and Approval Workflow

The admin functionality must be fully restored and working.

I previously experienced problems where the admin credentials supplied by the website-building AI did not work.

Do not assume the admin account exists. Inspect the actual authentication and database implementation.

### Admin capabilities

The admin must be able to:

- Log in through the common login page.
- Access an admin dashboard.
- View registered users.
- View registered bus operators.
- Approve or reject bus operator registrations.
- View pending bus submissions.
- Approve or reject new bus submissions.
- View pending route submissions.
- Approve or reject new route submissions.
- View all registered buses.
- View bus details and associated route details.
- Update incorrect bus information.
- Update incorrect route information.
- Manage stop information.
- Correct errors in existing records.

### Approval rules

**Bus operator registration:**

1. Operator registers.
2. Account receives a pending approval status.
3. Admin reviews the registration.
4. Admin approves or rejects it.
5. Only approved operators can access operator management features.

**New bus or route submission:**

1. Approved operator submits a new bus or route.
2. Submission is stored with pending approval status.
3. Admin reviews the submission.
4. Admin approves or rejects it.
5. Only approved submissions become publicly visible.

**Existing record updates:**

- Operators can update their authorized existing buses and routes.
- These updates do NOT require admin approval.
- Admin can update any record to correct errors.

Make sure the approval workflow is implemented in both frontend and backend.

Hiding a button in the frontend is not sufficient security. Backend APIs must enforce admin permissions.

---

# 9. Bus Operator Dashboard and Management Features

Currently, the operator side does not provide the required options to add or update buses and routes.

Implement the missing functionality.

### Operator registration

- Operators can register using the common registration system.
- Their role must be stored correctly.
- Newly registered operators remain pending until admin approval.
- Pending operators must not access restricted management features.
- Approved operators can access their operator dashboard.

### Operator dashboard

Provide suitable options to:

- View assigned or managed buses.
- Add a new bus.
- Add a new route.
- Update existing buses.
- Update existing routes.
- View submission approval status.
- Start and stop bus tracking.
- Confirm bus departures from stops.

### Permissions

- Operators can manage only their own or authorized buses/routes.
- They cannot modify another operator's records.
- New buses and routes require admin approval.
- Updates to authorized existing records do not require admin approval.

Implement proper server-side authorization and ownership checks.

---

# 10. Live GPS Bus Tracking

The operator's device should provide the live GPS location of the bus.

### Expected workflow

1. An approved operator logs in.
2. The operator selects an authorized bus.
3. The operator starts a journey or tracking session.
4. The browser requests location permission.
5. If permission is granted, the browser obtains GPS coordinates.
6. The frontend periodically sends the current location to the backend.
7. The backend stores the latest location.
8. Public users can view the updated location on the Live Map.

Use the browser Geolocation API appropriately.

A reasonable initial update interval is approximately 10 seconds, subject to GPS availability and battery/network considerations.

Example API:

```text
PATCH /api/buses/:id/location
```

Example payload:

```json
{
  "lat": 12.3456,
  "lng": 76.5432,
  "speed": 35,
  "occupancy": 20,
  "stopIndex": 3
}
```

Adapt the payload to the actual data model if necessary.

### Required handling

- Location permission denied.
- GPS unavailable.
- Network interruption.
- Invalid GPS coordinates.
- Tracking stopped.
- Browser tab closed or refreshed.
- Unauthorized bus location updates.

Only an authorized operator should be able to update an assigned bus's location.

Provide clear tracking status to the operator.

Do not simulate successful GPS tracking when real location updates are failing.

---

# 11. Manual Stop Departure Confirmation

Along with automatic GPS tracking, I want the bus operator to manually confirm when the bus departs from each stop.

### Expected workflow

- Show the stops in the correct route order.
- Display the current stop.
- Provide a clear button such as "Confirm Departure".
- When the bus departs, the operator taps the button.
- Save the actual departure timestamp in the database.
- Update the current stop/departure status.
- Advance the tracking workflow to the next stop.
- Make the latest status available to passengers.
- Prevent accidental duplicate departure confirmations.

This manual confirmation must work alongside GPS tracking.

Do not replace GPS tracking with manual updates.

The system should distinguish between estimated arrival information and actual operator-confirmed departure information.

---

# 12. Terms and Conditions Page

A Terms and Conditions page is currently missing.

Create a dedicated page:

```text
/terms
```

Requirements:

- Accessible through a link in the website footer.
- Responsive on mobile, tablet and desktop.
- Consistent with the existing BusTrack theme.
- Easy to read and navigate.

Include relevant sections covering:

- General terms of service.
- User responsibilities.
- Passenger account responsibilities.
- Operator responsibilities.
- GPS and location permissions.
- Location and personal data usage.
- Account security.
- Bus location and ETA accuracy limitations.
- Service availability.
- Misuse and account suspension.
- Changes to terms.
- Contact information.

Do not copy unrelated generic terms.

Do not invent company addresses, legal registrations, or contact details.

Clearly distinguish actual implemented data practices from proposed ones. Do not claim legal compliance without verification.

---

# 13. Forgot Password Using Email OTP

The common login page currently does not have a Forgot Password option.

Add a secure OTP-based password reset system.

### Expected workflow

**Step 1: Forgot Password**

- Add a "Forgot Password?" link on the login page.
- User enters their registered email address.

**Step 2: Generate and send OTP**

- Backend generates a cryptographically secure OTP.
- Backend sends the OTP to the registered email.
- Never generate or expose the OTP in the frontend.

**Step 3: Verify OTP**

- User enters the received OTP.
- Frontend sends the OTP to the backend.
- Backend independently validates it.
- Only the backend can decide whether the OTP is correct.

**Step 4: Reset password**

- After successful verification, issue a short-lived, single-use reset authorization.
- User enters and confirms the new password.
- Backend hashes the new password using bcrypt.
- Update the database securely.
- Invalidate the OTP and reset authorization after successful use.

### Security requirements

- Generate OTP using a cryptographically secure random generator.
- Store only a secure hash of the OTP.
- Never expose OTP values or hashes in frontend code or API responses.
- Do not store OTPs in browser localStorage or sessionStorage.
- Do not log OTPs or email credentials.
- OTP should expire quickly (for example, after five minutes).
- Limit incorrect OTP attempts.
- Apply rate limiting to OTP generation and resend requests.
- Prevent OTP reuse.
- Prevent password reset authorization reuse.
- Use generic responses to avoid revealing whether an email is registered.
- Use HTTPS in production.
- Keep email credentials in backend environment variables.
- Apply the same secure password reset principles to relevant account roles.

Important clarification: OTPs should be hashed for secure storage, not stored as plaintext. Passwords should be hashed using bcrypt.

---

# 14. Existing Features Must Continue Working

Do not break the features that already exist.

Preserve and verify:

- Home page.
- Live Map.
- Route listing.
- Route filtering by type.
- Route details.
- Stop search.
- Bus information.
- ETA per stop.
- Admin functionality.
- Authentication.
- Registration.
- MongoDB data.
- Demo seed data.
- Leaflet map functionality.
- Existing useful components and design.

The following existing API routes should be preserved or replaced only when there is a clear technical reason and all related frontend functionality is updated:

```text
POST   /api/auth/register
POST   /api/auth/login

GET    /api/routes
GET    /api/routes/:id
GET    /api/routes/:id/eta/:stopId
POST   /api/routes
PUT    /api/routes/:id
DELETE /api/routes/:id

GET    /api/buses
PATCH  /api/buses/:id/location
POST   /api/buses
DELETE /api/buses/:id

GET    /api/stops
POST   /api/stops
```

Add the necessary new endpoints for operator approval, operator management, submission approval, stop departure confirmation, and password reset.

Keep existing API contracts compatible wherever practical.

---

# 15. UI/UX Quality Requirements

I do not want the website to look like a generic AI-generated or vibe-coded template.

Improve the existing interface while preserving its identity.

Avoid:

- Excessive gradients.
- Random decorative cards.
- Too many rounded containers.
- Unnecessary animations.
- Oversized headings.
- Huge empty spaces.
- Inconsistent spacing.
- Too many different font styles.
- Random icons and emojis.
- Fake statistics.
- Buttons that do nothing.
- Placeholder content presented as real information.
- Repetitive sections.
- Unnecessary dashboards and features.
- Desktop-only layouts.

Use:

- Consistent typography.
- Clear visual hierarchy.
- Purposeful spacing.
- Consistent colors.
- Proper alignment.
- Meaningful icons.
- Clear labels.
- Appropriate loading indicators.
- Useful empty states.
- Meaningful error messages.
- Accessible controls.
- Professional navigation.
- Responsive components.

Make the application feel like a real transport tracking product, not a generic template.

Do not redesign everything unnecessarily.

---

# 16. Technical and Security Requirements

Review the existing implementation for:

- Authentication and JWT validation.
- Password hashing.
- Role-based access control.
- Admin authorization.
- Operator authorization.
- Resource ownership.
- API input validation.
- MongoDB schema consistency.
- Error handling.
- Secure environment variables.
- CORS configuration.
- Axios configuration.
- Location update validation.
- OTP security.
- Email service configuration.
- Rate limiting.
- Duplicate data prevention.

Never:

- Hardcode private credentials.
- Put JWT secrets in frontend code.
- Put email passwords in frontend code.
- Trust role information sent directly by the client.
- Allow an unapproved operator to manage buses.
- Allow an operator to modify another operator's records.
- Allow a pending bus/route submission to appear publicly.
- Fake GPS updates.
- Suppress errors instead of fixing them.

Use `.env.example` for documenting required environment variables without including real secrets.

---

# 17. Testing and Verification

Do not stop after making the UI look correct.

Test the complete workflows.

### Public user tests

- Open the website without logging in.
- View routes.
- View buses.
- Search stops.
- View route details.
- Open the live map.
- View bus locations.
- View ETA.
- Attempt to save a personal preference and verify login is requested.

### Authentication tests

- Valid passenger login.
- Invalid login.
- Passenger registration.
- Duplicate registration.
- Operator registration.
- Admin login.
- Forgot password request.
- OTP verification.
- Expired OTP.
- Incorrect OTP.
- Password reset.
- Login using the new password.

### Operator tests

- Pending operator cannot access management features.
- Admin can approve operator.
- Approved operator can access dashboard.
- Operator can add a bus.
- New bus remains pending until admin approval.
- Operator can add a route.
- New route remains pending until admin approval.
- Operator can update an authorized existing record without approval.
- Operator cannot update another operator's record.
- Operator can start and stop GPS tracking.
- Operator can confirm departure from stops.

### Admin tests

- Admin can log in.
- Admin can view pending operator registrations.
- Admin can approve/reject operators.
- Admin can approve/reject new buses.
- Admin can approve/reject new routes.
- Admin can view bus and route details.
- Admin can update incorrect records.
- Admin can manage stops.

### Map tests

- Map tiles load without API-key watermarks or errors.
- Bus markers appear correctly.
- Location updates reach the backend.
- Map reflects updated positions.
- Stop markers and route paths work.
- GPS failures are handled correctly.

### Responsive tests

Check all major pages at:

- Mobile screen sizes.
- Tablet screen sizes.
- Desktop screen sizes.

Verify that content is not clipped, text is readable, and there is no unwanted horizontal scrolling.

### Console and network tests

- No avoidable React warnings.
- No missing favicon.
- No failed API requests caused by incorrect configuration.
- No unhandled frontend errors.
- No backend crashes.
- No fake successful responses.

---

# 18. Implementation Instructions — Very Important

Follow this order:

**Phase 1 — Inspect**
- Read the complete existing project.
- Inspect frontend and backend.
- Understand existing database schemas and APIs.
- Identify which features are working, broken, or missing.
- Do not delete files or rewrite the entire project before understanding it.

**Phase 2 — Fix core errors**
- Fix frontend/backend connection.
- Fix login and registration.
- Fix routing warnings.
- Fix favicon.
- Fix map tile configuration.
- Fix responsive layouts.

**Phase 3 — Implement missing workflows**
- Common role-based login.
- Public access.
- Admin dashboard and approvals.
- Operator dashboard.
- Operator registration approval.
- Bus/route submission approval.
- GPS tracking.
- Manual stop departure confirmation.
- Forgot Password OTP.
- Terms and Conditions.

**Phase 4 — Verify**
- Test all roles.
- Test all workflows.
- Test backend APIs.
- Test database operations.
- Test security restrictions.
- Test responsive layouts.
- Fix remaining issues.

---

# 19. Final Instructions

1. Work with my existing codebase.
2. Preserve the current project structure where practical.
3. Preserve the existing useful design and theme.
4. Do not remove any original requirement.
5. Do not silently skip difficult features.
6. Do not replace real backend functionality with static dummy data.
7. Do not make buttons that only look functional.
8. Do not claim something works without testing it.
9. Do not introduce unnecessary paid APIs or services.
10. Do not create duplicate or conflicting implementations.
11. Do not change database structures without considering existing data and migrations.
12. Keep the frontend, backend, database and API behavior consistent.
13. If an external service needs credentials, clearly tell me which environment variables are required and how to configure them.
14. If a requirement cannot be implemented with the existing configuration, explain the limitation rather than pretending it is complete.

When finished, provide:

- A summary of all changes.
- A list of modified and newly created files.
- Any database schema changes.
- New environment variables required.
- Any installation commands.
- Exact steps to start the backend and frontend.
- Working test credentials, if demo accounts are configured.
- A list of features tested successfully.
- Any remaining issues that need my attention.

**Final goal:** Deliver a properly working, secure, responsive BusTrack application with public transport browsing, live GPS bus tracking, passenger features, operator management, admin approval workflows, and secure authentication — while retaining the existing project's intended functionality.
