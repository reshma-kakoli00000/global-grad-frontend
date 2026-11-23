# GlobalGrad Tracker — Frontend

The web client for **GlobalGrad Tracker**. GlobalGrad Tracker is a central, cloud-based workspace that helps students handle every stage of studying abroad, from finding universities to accepting an admission offer, all from one dashboard.

> Backend: see [`../global-grad-backend`](../global-grad-backend/readme.md)

---

## Overview

Students preparing to study abroad usually juggle spreadsheets, Google Drive folders, Gmail threads, and dozens of browser tabs. Deadlines get missed, document versions get mixed up, and eligibility is checked by hand. GlobalGrad Tracker brings all of this into one connected workspace:

- **One dashboard**: active applications, upcoming deadlines, and pending tasks in a single view.
- **End-to-end workflow**: from discovering universities to accepting an offer.
- **Cloud accessibility**: application data and documents are available from any device.
- **Built-in decision support**: an eligibility checker and program matching.

### User journey

```
1. Discovery → 2. Compare → 3. Save → 4. Outreach → 5. Prepare → 6. Check → 7. Submit → 8. Accept
   Explore     Programs     Build a    Professor    Document     Eligibility  Application  Track
   universities side-by-side shortlist  Tracker      Vault        Checker      Pipeline     offers
```

---

## Features

### Student workspace (`/dashboard`)

| Module | Route | Description |
|---|---|---|
| **Dashboard home** | `/dashboard` | Stats cards, application journey progress, critical deadlines, program matches, recent professor outreach. |
| **University & program explorer** | `/dashboard/programs` | Search and filter programs, featured and compact program cards, destination guides, market updates. |
| **Application pipeline** | `/dashboard/applications` | Kanban board tracking each application: *Research → Applied → Interview → Offer → Accepted*. |
| **Document vault** | `/dashboard/documents` | Organise SOPs, LORs, transcripts, and drafts, with storage usage and recent activity. |
| **Professor tracker** | `/dashboard/professors` | Professor list and profiles, research areas, fit score, communication timeline, internal notes. |
| **Scholarships** | `/dashboard/scholarships` | Funding overview, scholarship matches, active application tracker. |
| **Profile** | `/dashboard/profile` | Academic profile, credentials, test scores, research interests, experience, skills. |

### Admin console (`/admin`)

| Route | Description |
|---|---|
| `/admin/login` | Admin sign-in. |
| `/admin/dashboard` | Start university scrape jobs (Melbourne, Oxford, NUS) and watch background job status. |

### Public site

`/` (landing), `/features`, `/pricing`, `/about`, `/login`, `/signup`.

### Integration status

| Connected to the backend API | Using local mock data (`components/dashboard/*/data.ts`) |
|---|---|
| Sign up, log in, current user (`/users/me`) | Application pipeline, document vault, professor tracker, scholarships, programs |
| Admin: job list, scrape job submission | |

---

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    Next.js (App Router)                    │
│                                                            │
│  app/            pages & layouts (public, dashboard, admin)│
│     │                                                      │
│  components/     UI by feature (dashboard/*, features/,    │
│     │            pricing/, Navbar, Footer, JobForm, ...)   │
│     │                                                      │
│  providers/      AuthContext: user, token, role, login/    │
│     │            logout; persisted in localStorage         │
│     │                                                      │
│  services/       axios API clients (auth, jobs, scraping)  │
└─────┬──────────────────────────────────────────────────────┘
      │ HTTPS + JWT Bearer token
      ▼
  FastAPI backend (localhost:8000, or a public ngrok URL)
```

- **Rendering:** Next.js 16 App Router with the React Compiler enabled (`next.config.ts`).
- **State / auth:** `AuthProvider` (wrapped by `RootClientProvider`) stores the JWT and user in `localStorage`. Student and admin sessions are kept separately (`token` / `admin_token`).
- **API layer:** each domain has its own service module in `src/services/` built on `axios`. Requests include the `ngrok-skip-browser-warning` header so they work through ngrok.
- **Styling:** Tailwind CSS v4, `lucide-react` icons, responsive mobile-first layouts.

### Project structure

```
src/
├── app/
│   ├── page.tsx                 # landing page
│   ├── layout.tsx / RootClientProvider.tsx
│   ├── login/ signup/           # student auth
│   ├── features/ pricing/ about/
│   ├── dashboard/               # student workspace
│   │   ├── applications/ documents/ professors/
│   │   ├── programs/ scholarships/ profile/
│   │   └── page.tsx
│   └── admin/
│       ├── login/
│       └── dashboard/           # jobs + scraping
├── components/
│   ├── dashboard/               # per-module components + mock data
│   ├── features/ pricing/
│   └── Navbar, Footer, Layout, JobForm, JobTable
├── providers/authContext.tsx
├── services/                    # auth, jobs, scraping, programs, universities
└── types/
```

---

## Tech Stack

- **Framework:** Next.js 16 (App Router), React 19, TypeScript
- **Styling:** Tailwind CSS 4, lucide-react
- **HTTP:** axios
- **Auth:** JWT from the backend's OAuth2 password flow
- **Tooling:** ESLint (`eslint-config-next`), React Compiler

---

## How to Run

You need **Node.js 20+** and npm. To use login and the admin features, the backend must also be running.

```bash
npm install
npm run dev
```

Open **http://localhost:3000**.

### Point the frontend at your backend

The API base URL is currently set inside each service file:

- `src/services/auth/auth.tsx` → `baseURL`
- `src/services/jobs/jobs.tsx` → `API_BASE_URL`
- `src/services/scraping/scraping.tsx` → `API_BASE_URL`

They default to a public ngrok URL. For local development, switch them to the commented-out value:

```ts
const baseURL = "http://localhost:8000/";
```

To share the app, run `ngrok http 8000` against the backend and paste the generated URL into these files instead.

### Other scripts

```bash
npm run build   # production build
npm run start   # serve the production build
npm run lint    # lint
```

---

## Team — GradVault

| ID | Name |
|---|---|
| B200305027 | Reshma Kakoli |
| B200305048 | MD. Yusuf Hasan |

Submitted to **Dr. Md. Abu Layek**, Professor, Department of Computer Science & Engineering, Jagannath University.
