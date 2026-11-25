# Schemz Frontend (`client`)

The React + Vite frontend for **Schemz**, a government scheme eligibility and application platform connecting citizens, scheme organizers, and administrators.

---

## 🛠️ Tech Stack

| Category | Technologies |
|----------|--------------|
| **Core** | React 19, Vite 7, React Router DOM 7 |
| **Styling & Motion** | Tailwind CSS 3, Framer Motion, Lucide React |
| **Forms & Utilities** | React Hook Form, `date-fns`, React Toastify |
| **Networking & Real-time** | Axios, Socket.io Client |
| **Linting** | ESLint 9 (`eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`) |

---

## ⚡ Quick Start

### 1. Install Dependencies

```bash
cd client
npm install
```

### 2. Configure Environment Variables

Create a `.env` file in `client/`:

```env
VITE_API_BASE_URL=http://localhost:5000/api
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id
```

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_API_BASE_URL` | Backend REST API base URL (also used to derive the Socket.io origin) | `http://localhost:5000/api` |
| `VITE_GOOGLE_CLIENT_ID` | Google OAuth 2.0 Web Client ID for Google Sign-In | — |

### 3. Run Development Server

```bash
npm run dev
```

The app will be available at `http://localhost:5173`.

---

## 📜 Available Scripts

| Script | Command | Description |
|--------|---------|-------------|
| `dev` | `npm run dev` | Start Vite development server with HMR |
| `build` | `npm run build` | Build production bundle into `dist/` |
| `preview` | `npm run preview` | Preview the production build locally |
| `lint` | `npm run lint` | Run ESLint across the project |

---

## 📁 Project Structure

```text
client/
├── public/                     # Static assets
├── src/
│   ├── assets/                 # Bundled assets
│   ├── components/             # Reusable UI & feature components
│   │   ├── ApplySchemeModal.jsx
│   │   ├── ConfirmationModal.jsx
│   │   ├── LandingComponents.jsx
│   │   ├── Logo.jsx
│   │   ├── Navbar.jsx
│   │   ├── ProtectedRoute.jsx
│   │   └── RuleBuilder.jsx
│   ├── context/
│   │   └── AuthContext.jsx     # Auth state, token persistence, role context
│   ├── pages/
│   │   ├── Landing.jsx
│   │   ├── Login.jsx
│   │   ├── Register.jsx
│   │   ├── user/               # Citizen pages (Dashboard, Profile, Schemes, Applications)
│   │   ├── organizer/          # Organizer pages (Create/Edit Scheme, Applications)
│   │   ├── admin/              # Admin pages (Pending/All Schemes & Applications)
│   │   └── shared/
│   │       └── SupportQueries.jsx  # Real-time Socket.io support chat
│   ├── services/
│   │   └── api.js              # Axios instance, JWT interceptor, API endpoints
│   ├── utils/
│   │   └── profileUtils.js     # Profile completion & helper utilities
│   ├── App.jsx                 # Route definitions & role-based guards
│   ├── main.jsx                # Application entry point
│   └── index.css               # Tailwind directives & global styles
├── Dockerfile                  # Container build & preview server (port 4173)
├── tailwind.config.js          # Custom gov/brand/accent palette & animations
├── vercel.json                 # SPA rewrite rules for Vercel
├── vite.config.js              # Vite configuration
└── package.json
```

---

## 🧭 Application Routes

Routes are guarded by `ProtectedRoute` based on the authenticated user's role (`user`, `organizer`, `admin`):

### Public
- `/` — Landing page
- `/login` — Email/password & Google OAuth login
- `/register` — User registration & Google OAuth sign-up

### Citizen (`user`)
- `/user/dashboard` — Citizen overview
- `/user/profile` — Demographics, bank/Aadhaar info & reusable profile documents
- `/user/matched-schemes` — Personalized scheme matches with eligibility scores
- `/user/all-schemes` — Browse all active schemes
- `/user/scheme/:id` — Scheme details & application modal
- `/user/my-applications` — Track submitted scheme applications
- `/user/apply-organizer` — Apply for organizer role
- `/user/support-queries` — Support queries

### Organizer (`organizer`)
- `/organizer/dashboard` — Organizer overview
- `/organizer/create-scheme` — Create scheme with visual `RuleBuilder`
- `/organizer/my-schemes` — Manage drafts, pending, and active schemes
- `/organizer/edit-scheme/:id` — Edit draft or rejected schemes
- `/organizer/scheme-applications/:schemeId` — Review citizen applications
- `/organizer/support-queries` — Real-time support chat with admins

### Admin (`admin`)
- `/admin/dashboard` — Admin overview
- `/admin/pending-schemes` — Approve/reject submitted schemes
- `/admin/all-schemes` — View and toggle scheme active status
- `/admin/pending-applications` — Approve/reject organizer role requests
- `/admin/all-applications` — View all organizer applications
- `/admin/support-queries` — Real-time support chat & query resolution

---

## 🔌 API & Real-Time Integration

- **REST API (`src/services/api.js`):** Attaches `Authorization: Bearer <token>` from `localStorage` on every request and redirects to `/login` on `401 Unauthorized` responses (excluding login/register endpoints).
- **Real-Time Chat (`src/pages/shared/SupportQueries.jsx`):** Connects via `socket.io-client` to the backend origin derived from `VITE_API_BASE_URL`, listening for `new-message` and `query-resolved` events.

---

## 🚀 Deployment

### Vercel
`vercel.json` is configured with SPA rewrites so all client-side routes resolve to `/index.html`:

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

### Docker
Build and run the containerized preview server (exposes port `4173`):

```bash
docker build -t schemz-client .
docker run -p 4173:4173 schemz-client
```
