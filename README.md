# 🪐 MISSION FORGE — Architect the Unknown

> **Where imagination becomes the next mission.**

MISSION FORGE is a full-stack web game where you become a **Mission Architect**: you pick a destination, build a spacecraft from modules, fly the mission through random emergencies and discoveries, and share your design with a community gallery. Your account, designs, scores and achievements are saved in a database, so everything is still there when you log out and back in.

---

## 1. For judges: what to look at (2-minute tour)

| Step | What to do | What it proves |
|---|---|---|
| 1 | Open the app → **Sign up** | Account creation, validation, secure password storage |
| 2 | Enter the **Mission Hub** and pick **Mission Forge** | Game UI, animations, sound (use the 🔊 / 🎵 buttons) |
| 3 | Choose a destination, **build a ship** (tap or drag modules, tap a placed module to rotate it) | Interactive builder with live health bars and the ORBIT AI advisor |
| 4 | Click **Save**, then **Launch** and make the decisions during the events | Dynamic consequence engine: outcomes depend on your ship and choices |
| 5 | On the report card click **Publish** | Data written to the database and shown in the public gallery |
| 6 | Click **👤** (top right of the game) → **Profile** | Stats, achievements, saved designs, mission history |
| 7 | **Log out**, then **log in again** | Everything is still there: the data is persisted |

An automated test checks this exact flow (see section 6).

---

## 2. Features

**Game**
- Cinematic intro, animated galaxy map and seven destinations, including **THE UNKNOWN**, a procedurally generated world with a different mystery each time
- Spacecraft builder with 16 modules (propulsion, power, science, shielding, rovers, drones, landers, sample return, and more). Drag and drop, tap to add, rotate, remove
- Live **Mission Health Score**: power, fuel, science, safety, cost, duration, risk
- **ORBIT**, the holographic AI companion that reacts to your design, warns about risks and celebrates your choices
- **Consequence Engine**: random events (solar storm, comms failure, blocked landing, fuel leak, mysterious signal) with several choices and risk-based outcomes; there is no single correct answer
- Launch countdown, journey timeline (Earth → Orbit → Deep Space → Destination → Exploration → Discovery), discoveries that depend on how you played
- Animated **Mission Report Card** with a title (Bold Explorer, Risk Taker, Science Pioneer, Precision Architect, Deep Space Visionary) and a share button
- Synthesized sound: ambient music, UI clicks, module snaps, engine ignition, countdown, radio, alarms, discovery and achievement jingles, with mute controls

**Game modes (Mission Hub)**
Mission Forge · Spacecraft Challenge (budget/module limits) · Mission Crisis (4 emergencies in a row) · Discovery Hunt (scan a planet for hidden clues) · Space Sandbox (no limits) · Daily Challenge (same mission for everyone, ranked) · Mission Gallery

**Full-stack features**
- Signup, login, logout, user profile
- Save designs privately, or **publish** them to the public gallery; unpublish or delete from your profile
- Missions and scores saved per user; mission count unlocks new modules (Fusion Core, AI Brain, Quantum Link)
- Achievements saved per user
- Gallery voting (one vote per user per design, toggleable)
- Real **daily leaderboard rank** computed from all players' scores that day

---

## 3. Architecture

```
┌──────────────────────────┐        HTTPS / JSON         ┌─────────────────────────┐        SQL        ┌──────────────┐
│  Browser                 │  ───────────────────────▶   │  Node.js + Express API  │  ──────────────▶  │  PostgreSQL  │
│  • React app (auth,      │  ◀───────────────────────   │  • validation (Zod)     │  ◀──────────────  │  users       │
│    profile)              │   httpOnly session cookie   │  • auth (bcrypt + JWT)  │                   │  designs     │
│  • Game (canvas, audio,  │                             │  • rate limiting        │                   │  missions    │
│    ORBIT) in an iframe   │                             │  • serves built React   │                   │  achievements│
└──────────────────────────┘                             └─────────────────────────┘                   │  votes       │
                                                                                                        └──────────────┘
```

In production a single Node process serves both the API and the built React app, so there is one thing to deploy and no cross-origin issues.

**Tech stack**

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, React Router |
| Game | Plain JavaScript, Canvas, Web Audio API (the original game, now API-backed) |
| Backend | Node.js, Express 4 |
| Database | PostgreSQL (`pg`) |
| Auth | bcrypt (cost 12), JSON Web Tokens in an httpOnly cookie |
| Validation / security | Zod, Helmet (CSP), express-rate-limit |
| Deployment | Dockerfile, docker-compose, Render blueprint |

---

## 4. Project structure

```
mission-forge/
├── server/
│   ├── index.js            Entry point (loads env, creates tables, starts server)
│   ├── app.js              Express app: security headers, routes, static files, error handler
│   ├── db.js               PostgreSQL connection + first-run gallery seed
│   ├── schema.sql          Database tables (created automatically)
│   ├── auth.js             JWT cookie helpers + requireAuth middleware
│   ├── validate.js         Zod schemas for every request body
│   ├── routes/
│   │   ├── auth.js         signup · login · logout · me
│   │   └── game.js         designs · missions · achievements · gallery · votes · profile · leaderboard
│   └── test/flow.test.js   End-to-end API test (18 checks)
├── client/
│   ├── index.html
│   ├── vite.config.js      Dev proxy: /api → Express
│   ├── public/game.html    The game (talks to the API)
│   └── src/
│       ├── main.jsx        Routing + auth state
│       ├── pages/          Auth (login/signup) · Play · Profile
│       ├── api.js          fetch wrapper
│       ├── AuthContext.js
│       └── Stars.jsx, styles.css
├── .env.example            Environment variable template
├── Dockerfile · docker-compose.yml · render.yaml
└── README.md
```

---

## 5. Install and run

**Requirements:** Node.js 18+ and PostgreSQL 13+ (or Docker).

### Option A: Docker (one command)
```bash
docker compose up --build
```
Open **http://localhost:3000**. The database is created automatically.

### Option B: Local
```bash
# 1. Create the database
createdb missionforge

# 2. Configure environment
cp .env.example .env
#    edit .env: set DATABASE_URL and JWT_SECRET (see table below)

# 3. Install, build the React client, start
npm install
npm run build
npm start
```
Open **http://localhost:3000**. Tables are created on first start, and the gallery is seeded with four sample designs.

**Development mode (hot reload):** run `npm run dev:server` in one terminal and `npm run dev:client` in another, then open http://localhost:5173.

### Environment variables (`.env`)

| Variable | Required | Meaning |
|---|---|---|
| `DATABASE_URL` | yes | PostgreSQL connection string, e.g. `postgres://user:pass@localhost:5432/missionforge` |
| `JWT_SECRET` | yes | Long random string (16+ characters). Generate: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"` |
| `DATABASE_SSL` | no | `true` for hosted databases that need SSL (Render, Neon, Supabase, Heroku) |
| `PORT` | no | Default `3000` |
| `NODE_ENV` | no | `production` enables Secure cookies |
| `JWT_DAYS` | no | Session length in days (default 7) |

---

## 6. Testing

```bash
npm test
```

The test starts the real app and walks through the complete user journey, checking every response:

**Sign Up → Login → Design → Play → Save → View Profile → Publish → Logout → Login Again → Data Persists**

It also checks that bad input is rejected, duplicate accounts are blocked, wrong passwords fail, anonymous users get `401`, invalid mission data is refused, votes work, and the gallery only shows published designs. With `DATABASE_URL` set it runs against your PostgreSQL; without it, it uses an in-memory Postgres emulator. Result: **18 / 18 checks pass.**

---

## 7. API reference

All endpoints are under `/api`, send and receive JSON, and (except signup/login/health) require the login cookie.

| Method & path | Purpose |
|---|---|
| `POST /auth/signup` | Create account (username 3-20 chars, valid email, password 8+ chars) |
| `POST /auth/login` | Log in with username or email |
| `POST /auth/logout` | Clear session |
| `GET /auth/me` | Current user |
| `GET /me/state` | Mission count + achievement keys (used by the game on load) |
| `GET /me/profile` | Stats, achievements, designs, recent missions |
| `POST /designs` | Save a design (private or published) |
| `PATCH /designs/:id` | Publish / unpublish (owner only) |
| `DELETE /designs/:id` | Delete (owner only) |
| `POST /missions` | Record a finished mission; returns daily rank for the Daily Challenge |
| `POST /achievements` | Unlock an achievement (idempotent) |
| `GET /gallery` | Published designs with vote counts and whether you voted |
| `POST /gallery/:id/vote` | Toggle your vote |
| `GET /leaderboard/daily` | Today's top scores |
| `GET /health` | Health check |

**Database tables:** `users`, `designs`, `missions`, `achievements`, `votes` (see `server/schema.sql`).

---

## 8. Security

- Passwords hashed with **bcrypt**; never stored or returned in plain text
- Session in an **httpOnly, SameSite=Lax** cookie (Secure in production), so scripts cannot read it
- Every request body validated server-side with **Zod**
- **Parameterized SQL** only (no SQL injection)
- Owners-only checks on edit/delete; votes require login
- **Rate limiting** on login/signup
- **Helmet** security headers and Content-Security-Policy
- CSRF mitigation: SameSite cookies + mutations must be `application/json`
- Errors are logged server-side; clients get generic messages in production

**Known limitation:** scores are calculated in the browser by the game, so a determined player could submit a fake score. The server only checks that values are in a valid range. A production competitive game would move scoring server-side.

---

## 9. Deployment

**Render (easiest):** push the folder to GitHub → Render dashboard → **New → Blueprint** → select the repo. `render.yaml` creates the web service, a PostgreSQL database and a random `JWT_SECRET`.

**Railway / Fly.io / Heroku:** add a PostgreSQL database, set `DATABASE_URL`, `DATABASE_SSL=true`, `JWT_SECRET`, `NODE_ENV=production`. Build command: `npm install && npm run build`. Start command: `npm start`.

**Any Docker host:** build the included `Dockerfile` and pass the environment variables above.

---

## 10. Troubleshooting

| Problem | Fix |
|---|---|
| `Startup failed: DATABASE_URL is not set` | Create `.env` from `.env.example` |
| `JWT_SECRET must be set` | Set a secret of 16+ characters |
| Blank page at `/` | Run `npm run build` first |
| No sound | Browsers start audio after the first click; also check the 🔊 / 🎵 buttons |
| Hosted DB connection error | Set `DATABASE_SSL=true` |

---

**YOU DIDN'T JUST BUILD A SPACECRAFT. YOU CREATED A POSSIBILITY.**
**MISSION FORGE — WHERE IMAGINATION BECOMES THE NEXT MISSION.**
