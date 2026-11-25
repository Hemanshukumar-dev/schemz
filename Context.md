# Schemz - Project Context & Architecture Reference

> **Last Updated:** 2026-02-28  
> **Status:** Active Development  
> **Purpose:** Rule-based government scheme eligibility & application platform for India

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Folder Structure](#folder-structure)
4. [Environment Variables](#environment-variables)
5. [Database Models](#database-models)
6. [Backend API Reference](#backend-api-reference)
7. [Authentication Flow](#authentication-flow)
8. [Services Layer](#services-layer)
9. [Eligibility Engine](#eligibility-engine)
10. [Frontend Architecture](#frontend-architecture)
11. [Client-Side Routing](#client-side-routing)
12. [Real-Time (Socket.io)](#real-time-socketio)
13. [File Uploads](#file-uploads)
14. [Running the Project](#running-the-project)
15. [Known Notes & Caveats](#known-notes--caveats)

---

## Project Overview

**Schemz** is a full-stack web application that connects Indian citizens with government welfare schemes. Three user roles exist:

| Role             | Purpose                                                                                    |
| ---------------- | ------------------------------------------------------------------------------------------ |
| `user` (Citizen) | Browse schemes, get AI-matched eligibility scores, apply to schemes, raise support queries |
| `organizer`      | Create & manage government schemes (AI-assisted), review citizen applications              |
| `admin`          | Approve/reject schemes and organizer applications, oversee all data                        |

Scheme eligibility is determined by a **rule-based scoring engine** (`eligibilityEngine.js`). Each scheme defines weighted rules (e.g., age ≤ 35, category == OBC) and users get a match percentage based on their profile.

---

## Tech Stack

### Backend (`/server`)

| Layer             | Technology                                                  |
| ----------------- | ----------------------------------------------------------- |
| Runtime           | Node.js (ESM modules)                                       |
| Framework         | Express 4                                                   |
| Database          | MongoDB via Mongoose 8                                      |
| Authentication    | JWT (jsonwebtoken) + Google OAuth 2.0 (google-auth-library) |
| Real-time         | Socket.io 4                                                 |
| File Uploads      | Multer (memory storage)                                     |
| Image Storage     | Cloudinary v2                                               |
| Document Storage  | Backblaze B2                                                |
| AI                | Google Gemini 2.5 Flash via `@ai-sdk/google` + `ai` SDK     |
| Email             | Nodemailer (SMTP/Gmail)                                     |
| Schema Validation | Zod (AI output validation only)                             |
| Password Hashing  | bcryptjs                                                    |
| Testing           | Jest + Supertest                                            |

### Frontend (`/client`)

| Layer          | Technology                                 |
| -------------- | ------------------------------------------ |
| Build Tool     | Vite 7                                     |
| Framework      | React 19                                   |
| Routing        | React Router DOM 7                         |
| HTTP Client    | Axios (with request/response interceptors) |
| State / Auth   | React Context API (`AuthContext`)          |
| Forms          | react-hook-form                            |
| Animations     | Framer Motion                              |
| Icons          | lucide-react                               |
| Notifications  | react-toastify                             |
| Real-time      | socket.io-client 4                         |
| Date Utilities | date-fns                                   |
| Styling        | Tailwind CSS 3 + Vanilla CSS               |

---

## Folder Structure

```
schemz/
├── client/                         # React frontend (Vite)
│   ├── public/
│   ├── src/
│   │   ├── App.jsx                 # Root component: router + role-based route guards
│   │   ├── main.jsx                # Entry point
│   │   ├── index.css               # Global CSS / Tailwind directives
│   │   ├── App.css
│   │   ├── context/
│   │   │   └── AuthContext.jsx     # Global auth state (user, login, logout, googleAuth, etc.)
│   │   ├── services/
│   │   │   └── api.js              # Axios instance + all API client functions (authAPI, schemeAPI, etc.)
│   │   ├── utils/
│   │   │   └── profileUtils.js     # Profile-related utility helpers
│   │   ├── components/
│   │   │   ├── ApplySchemeModal.jsx     # Multi-step modal for scheme application with doc uploads
│   │   │   ├── ConfirmationModal.jsx    # Generic confirmation dialog
│   │   │   ├── LandingComponents.jsx    # Landing page section components (hero, features, etc.)
│   │   │   ├── Logo.jsx                 # App logo component
│   │   │   ├── Navbar.jsx               # Role-aware top navigation
│   │   │   ├── ProtectedRoute.jsx       # Route guard (auth + role check)
│   │   │   └── RuleBuilder.jsx          # Visual UI for building scheme eligibility rules
│   │   └── pages/
│   │       ├── Landing.jsx              # Public landing page (redirects authenticated users)
│   │       ├── Login.jsx                # Login form (email/password + Google OAuth)
│   │       ├── Register.jsx             # Registration form
│   │       ├── user/
│   │       │   ├── Dashboard.jsx        # User dashboard: stats, quick links
│   │       │   ├── Profile.jsx          # Full profile editor with image + document uploads
│   │       │   ├── MatchedSchemes.jsx   # AI-matched scheme recommendations
│   │       │   ├── AllSchemes.jsx       # Browse all approved schemes
│   │       │   ├── SchemeDetail.jsx     # Detail view + apply button for a scheme
│   │       │   ├── MyApplications.jsx   # User's submitted scheme applications
│   │       │   └── ApplyOrganizer.jsx   # Form to apply for organizer role
│   │       ├── organizer/
│   │       │   ├── Dashboard.jsx        # Organizer dashboard: scheme stats
│   │       │   ├── CreateScheme.jsx     # Create scheme with AI-assist + RuleBuilder
│   │       │   ├── EditScheme.jsx       # Edit existing scheme draft
│   │       │   ├── MySchemes.jsx        # List organizer's own schemes
│   │       │   └── SchemeApplications.jsx  # View & approve/reject citizen applications
│   │       ├── admin/
│   │       │   ├── Dashboard.jsx        # Admin dashboard: platform-wide stats
│   │       │   ├── PendingSchemes.jsx   # Review pending scheme submissions
│   │       │   ├── AllSchemes.jsx       # View all schemes with status filter
│   │       │   ├── PendingApplications.jsx  # Review pending organizer applications
│   │       │   └── AllApplications.jsx  # View all organizer applications
│   │       └── shared/
│   │           └── SupportQueries.jsx   # Real-time support chat (used by all 3 roles)
│   ├── .env                         # VITE_API_BASE_URL
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── vercel.json                  # Vercel SPA rewrite config
│
├── server/                          # Node.js/Express backend
│   ├── server.js                    # Entry: Express setup, Socket.io, route mounting
│   ├── config/
│   │   └── db.js                    # Mongoose connection
│   ├── models/
│   │   ├── User.js                  # User schema (all roles, profile fields, OAuth)
│   │   ├── Scheme.js                # Scheme schema with embedded rule sub-documents
│   │   ├── SchemeApplication.js     # Citizen's application to a scheme
│   │   ├── OrganizerApplication.js  # User's application to become organizer
│   │   └── SupportQuery.js          # Support ticket with embedded messages
│   ├── controllers/
│   │   ├── authController.js        # register, login, getMe, updateProfile, uploadProfileImage, uploadProfileDocuments, googleLogin
│   │   ├── schemeController.js      # getAllSchemes, getScheme, getEligibleSchemes
│   │   ├── organizerController.js   # createScheme, getOrganizerSchemes, updateScheme, submitScheme, deleteScheme
│   │   ├── adminController.js       # Scheme approval + organizer application management
│   │   ├── applicationController.js # submitOrganizerApplication, getMyApplications, getApplicationStatus
│   │   ├── schemeApplicationController.js  # Full scheme application CRUD + approve/reject
│   │   ├── supportQueryController.js       # createQuery, getQueries, getQueryById, sendMessage, resolveQuery
│   │   └── aiController.js          # generateScheme (Gemini 2.5 Flash AI)
│   ├── routes/
│   │   ├── authRoutes.js            # /api/auth/*
│   │   ├── schemeRoutes.js          # /api/schemes/*
│   │   ├── organizerRoutes.js       # /api/organizer/*
│   │   ├── adminRoutes.js           # /api/admin/*
│   │   ├── applicationRoutes.js     # /api/application/*
│   │   ├── schemeApplicationRoutes.js  # /api/scheme-applications/*
│   │   ├── supportQueryRoutes.js    # /api/support-queries/*
│   │   └── aiRoutes.js              # /api/ai/*
│   ├── middleware/
│   │   ├── auth.js                  # protect (JWT verify), authorize (role check), generateToken
│   │   └── validateObjectId.js      # Validates MongoDB ObjectId in route params
│   ├── services/
│   │   ├── eligibilityEngine.js     # Core rule-evaluation engine (weighted match %)
│   │   ├── cloudinaryService.js     # uploadProfileImage (images → Cloudinary)
│   │   ├── backblazeService.js      # uploadDocument, getSignedUrl (PDFs/docs → B2)
│   │   └── emailService.js          # sendVerificationEmail, sendWelcomeEmail (Nodemailer)
│   ├── .env
│   └── package.json
│
└── tests/                           # Jest test suites (supertest)
```

---

## Environment Variables

### Server (`/server/.env`)

| Variable                       | Purpose                                   |
| ------------------------------ | ----------------------------------------- |
| `PORT`                         | Server port (default: `5000`)             |
| `NODE_ENV`                     | `development` or `production`             |
| `MONGODB_URI`                  | MongoDB connection string                 |
| `JWT_SECRET`                   | Secret key for signing JWTs               |
| `JWT_EXPIRE`                   | JWT expiry duration (e.g., `7d`)          |
| `FRONTEND_URL`                 | Frontend origin for CORS & email links    |
| `CLOUDINARY_CLOUD_NAME`        | Cloudinary cloud name                     |
| `CLOUDINARY_API_KEY`           | Cloudinary API key                        |
| `CLOUDINARY_API_SECRET`        | Cloudinary API secret                     |
| `B2_APPLICATION_KEY_ID`        | Backblaze B2 app key ID                   |
| `B2_APPLICATION_KEY`           | Backblaze B2 app key                      |
| `B2_BUCKET_NAME`               | B2 bucket name (e.g., `Beast77`)          |
| `B2_ENDPOINT`                  | B2 S3-compatible endpoint                 |
| `GOOGLE_CLIENT_ID`             | Google OAuth 2.0 client ID                |
| `GOOGLE_CLIENT_SECRET`         | Google OAuth 2.0 client secret            |
| `GOOGLE_CALLBACK_URL`          | Google OAuth redirect callback URL        |
| `GOOGLE_GENERATIVE_AI_API_KEY` | Gemini AI API key (from Google AI Studio) |
| `EMAIL_HOST`                   | SMTP host (default: `smtp.gmail.com`)     |
| `EMAIL_PORT`                   | SMTP port (default: `587`)                |
| `EMAIL_USER`                   | SMTP username                             |
| `EMAIL_PASS`                   | SMTP app password                         |
| `EMAIL_FROM`                   | Sender address for outgoing emails        |

### Client (`/client/.env`)

| Variable            | Purpose                                                     |
| ------------------- | ----------------------------------------------------------- |
| `VITE_API_BASE_URL` | Backend API base URL (default: `http://localhost:5000/api`) |

---

## Database Models

### `User`

Stores all users regardless of role (`user`, `organizer`, `admin`).

```
_id, name, email, password (select:false, optional for OAuth), googleId (sparse)
role: enum['user', 'organizer', 'admin']  default:'user'

// Profile / eligibility fields
age, dateOfBirth, phone, income, category (General/OBC/SC/ST/EWS)
education (Below 10th → Doctorate), gender (Male/Female/Other)
state, district, pinCode, maritalStatus, religion, disability (None/Physical/Visual/Hearing/Mental/Multiple)
occupation, profileImage

// Stored documents (reused across applications)
aadhaarNumber, bankDetails { accountNumber, ifscCode, bankName, branchName }
documents { aadhaarDocument, incomeCertificate, categoryCertificate }  // Backblaze B2 URLs

timestamps: createdAt, updatedAt
```

**Hooks:** `pre('save')` hashes `password` with bcrypt (salt 10). Skips if `googleId` is set.  
**Methods:** `matchPassword(entered)` — bcrypt compare (returns false for OAuth users).

---

### `Scheme`

Represents a government scheme created by an organizer.

```
_id, name, description, benefits, ministry
status: enum['draft', 'pending', 'approved', 'rejected']  default:'draft'
active: Boolean  // Admin toggles to open/close applications
rules: [RuleSchema]
createdBy: ObjectId → User
approvedBy: ObjectId → User
remarks: String  // Admin notes on rejection

timestamps: createdAt, updatedAt
Index: { status: 1, active: 1 }
```

**RuleSchema (embedded, no `_id`):**

```
field: enum['age', 'income', 'category', 'education', 'state', 'gender', 'marital_status', 'disability', 'occupation']
operator: enum['==', '!=', '<', '<=', '>', '>=', 'in', 'not in']
value: Mixed (String | Number | Boolean | Array)
weight: Number (1–100, default: 10)
```

---

### `SchemeApplication`

A citizen's application to a specific scheme.

```
_id
user: ObjectId → User
scheme: ObjectId → Scheme
status: enum['pending', 'approved', 'rejected']  default:'pending'

applicantDetails: {  // Snapshot of user profile at time of apply
  name, email, phone, dateOfBirth, age, gender, category, religion,
  maritalStatus, state, district, education, occupation, income, disability
}

applicationData: {
  purpose: String (required)
  bankDetails: { accountNumber, ifscCode, bankName, branchName }
  aadhaarNumber: String
  remarks: String
}

documents: {
  marksheet: String (Backblaze B2 URL — required)
  incomeCertificate: String (Backblaze B2 URL)
  categoryCertificate: String (Backblaze B2 URL)
  otherDocuments: [{ name, url }]  // up to 5 files
}

reviewedBy: ObjectId → User
reviewedAt: Date
rejectionReason: String

timestamps: createdAt, updatedAt
Indexes: { user:1, scheme:1 }, { scheme:1, status:1 }, { status:1 }
```

---

### `OrganizerApplication`

A `user`-role user applying to become an organizer.

```
_id
user: ObjectId → User
organization: String (required)
designation: String (required)
reason: String (required, minlength: 50)
contactNumber: String (required)
status: enum['pending', 'approved', 'rejected']  default:'pending'
reviewedBy: ObjectId → User
remarks: String
reviewedAt: Date

timestamps: createdAt, updatedAt
Index: { user:1, status:1 }
```

When admin approves, user's `role` is updated to `'organizer'`.

---

### `SupportQuery`

A real-time support ticket with threaded messaging.

```
_id
createdBy: ObjectId → User
subject: String (required)
status: enum['open', 'resolved']  default:'open'
messages: [MessageSchema]
resolvedAt: Date
resolvedBy: ObjectId → User

timestamps: createdAt, updatedAt
Indexes: { createdBy:1, status:1 }, { status:1 }
```

**MessageSchema:**

```
_id: ObjectId
sender: ObjectId → User
content: String (required)
createdAt: Date
```

---

## Backend API Reference

**Base URL:** `http://localhost:5000/api`  
**Auth Header:** `Authorization: Bearer <JWT>`

---

### Auth Routes — `/api/auth`

| Method | Path                        | Auth    | Description                                                                                  |
| ------ | --------------------------- | ------- | -------------------------------------------------------------------------------------------- |
| POST   | `/register`                 | Public  | Register with name, email, password, role                                                    |
| POST   | `/login`                    | Public  | Login with email + password; returns JWT                                                     |
| POST   | `/google`                   | Public  | Google One Tap OAuth login; verifies ID token via google-auth-library; creates user if new   |
| GET    | `/me`                       | Private | Get current user's full profile                                                              |
| PUT    | `/profile`                  | Private | Update user profile fields                                                                   |
| POST   | `/upload-profile-image`     | Private | Upload profile image → Cloudinary (max 5MB, images only)                                     |
| POST   | `/upload-profile-documents` | Private | Upload aadhaarDocument, incomeCertificate, categoryCertificate → Backblaze B2 (max 5MB each) |

---

### Scheme Routes — `/api/schemes`

| Method | Path     | Auth                | Description                                                                 |
| ------ | -------- | ------------------- | --------------------------------------------------------------------------- |
| GET    | `/`      | Public              | Get all active + approved schemes                                           |
| GET    | `/:id`   | Public              | Get single scheme by ID (validates ObjectId)                                |
| GET    | `/match` | Private (user only) | Run eligibility engine against user's profile; returns scored + sorted list |

---

### Organizer Routes — `/api/organizer` (organizer only)

| Method | Path                 | Description                            |
| ------ | -------------------- | -------------------------------------- |
| POST   | `/scheme`            | Create a new scheme draft              |
| GET    | `/schemes`           | Get own schemes                        |
| PUT    | `/scheme/:id`        | Update scheme (draft/rejected only)    |
| POST   | `/scheme/:id/submit` | Submit draft scheme for admin approval |
| DELETE | `/scheme/:id`        | Delete own scheme                      |

---

### Admin Routes — `/api/admin` (admin only)

| Method | Path                       | Description                                        |
| ------ | -------------------------- | -------------------------------------------------- |
| GET    | `/schemes/pending`         | Get all pending schemes                            |
| GET    | `/schemes`                 | Get all schemes (filterable by status)             |
| POST   | `/scheme/:id/approve`      | Approve a pending scheme                           |
| POST   | `/scheme/:id/reject`       | Reject a pending scheme (with remarks)             |
| PUT    | `/scheme/:id/toggle`       | Toggle scheme active/inactive                      |
| GET    | `/applications/pending`    | Get pending organizer applications                 |
| GET    | `/applications`            | Get all organizer applications                     |
| POST   | `/application/:id/approve` | Approve organizer application (upgrades user role) |
| POST   | `/application/:id/reject`  | Reject organizer application                       |

---

### Organizer Application Routes — `/api/application` (user only)

| Method | Path               | Description                              |
| ------ | ------------------ | ---------------------------------------- |
| POST   | `/organizer`       | Submit organizer application             |
| GET    | `/my-applications` | Get user's own organizer applications    |
| GET    | `/status`          | Get current organizer application status |

---

### Scheme Application Routes — `/api/scheme-applications`

| Method | Path                | Auth              | Description                                                                                     |
| ------ | ------------------- | ----------------- | ----------------------------------------------------------------------------------------------- |
| POST   | `/`                 | user              | Submit scheme application (multipart: marksheet required, max 5MB each, up to 5 otherDocuments) |
| GET    | `/my-applications`  | Any authenticated | Get current user's scheme applications                                                          |
| GET    | `/check/:schemeId`  | Any authenticated | Check if user has already applied to a scheme                                                   |
| GET    | `/scheme/:schemeId` | organizer, admin  | Get all applications for a specific scheme                                                      |
| GET    | `/`                 | admin             | Get all scheme applications (filterable by status/schemeId)                                     |
| PATCH  | `/:id/approve`      | organizer, admin  | Approve a scheme application                                                                    |
| PATCH  | `/:id/reject`       | organizer, admin  | Reject a scheme application (reason required)                                                   |

**File upload fields (multipart/form-data):**

- `marksheet` (1 file, required)
- `incomeCertificate` (1 file, optional — falls back to profile document)
- `categoryCertificate` (1 file, optional — falls back to profile document)
- `otherDocuments` (up to 5 files)

---

### Support Query Routes — `/api/support-queries` (user, organizer, admin)

| Method | Path            | Auth       | Description                                                |
| ------ | --------------- | ---------- | ---------------------------------------------------------- |
| POST   | `/`             | Any        | Create new support query                                   |
| GET    | `/`             | Any        | Get queries (role-filtered: users see own, admins see all) |
| GET    | `/:id`          | Any        | Get single query with messages                             |
| POST   | `/:id/messages` | Any        | Send a message in a query thread                           |
| PATCH  | `/:id/resolve`  | admin only | Mark query as resolved                                     |

---

### AI Routes — `/api/ai` (organizer only)

| Method | Path               | Description                                                                                                                                       |
| ------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| POST   | `/generate-scheme` | Generate scheme draft using Gemini 2.5 Flash. Body: `{ prompt: string }`. Returns structured `{ name, description, benefits, ministry, rules[] }` |

---

### Health Check

| Method | Path          | Auth   | Description                         |
| ------ | ------------- | ------ | ----------------------------------- |
| GET    | `/api/health` | Public | Returns server status and timestamp |

---

## Authentication Flow

### Email/Password

1. **Register:** POST `/api/auth/register` → creates user with hashed password → returns JWT + user data
2. **Login:** POST `/api/auth/login` → validates credentials → returns JWT + user data
3. **Persistence:** JWT stored in `localStorage` under key `"token"`. `AuthContext` loads it on mount by calling `/api/auth/me`
4. **Auto-logout:** Axios response interceptor removes token and redirects to `/login` on `401` (except auth endpoints)

### Google OAuth (One Tap)

1. Frontend receives signed ID token from Google One Tap
2. POST `/api/auth/google` with `{ credential: <idToken> }`
3. Backend verifies token using `google-auth-library` `OAuth2Client.verifyIdToken()`
4. Looks up user by email; creates new `user`-role account if not found (sets `googleId`, `profileImage`)
5. Returns own JWT + user data (same format as email/password)
6. Frontend calls `googleAuth(userData, token)` from `AuthContext` to set state

### Protected Routes (Client)

- `ProtectedRoute` component wraps all authenticated routes
- Checks `isAuthenticated` and `allowedRoles` from `AuthContext`
- Redirects to `/` if not authenticated; redirects to role-appropriate dashboard if wrong role

### JWT Middleware (Server)

- `protect`: Extracts Bearer token from `Authorization` header → `jwt.verify()` → attaches `req.user` (without password)
- `authorize(...roles)`: Checks `req.user.role` against allowed roles; returns 403 if not authorized
- `generateToken(id)`: Signs JWT with `JWT_SECRET`, expiry from `JWT_EXPIRE`

### Socket.io Authentication

- Socket.io middleware extracts token from `socket.handshake.auth.token` or `socket.handshake.query.token`
- Verifies JWT and attaches user to `socket.user`
- Connection rejected if token missing or invalid

---

## Services Layer

### `cloudinaryService.js`

- **`uploadProfileImage(buffer)`** — uploads to `profile-images/` folder, resource_type `image`
- **`uploadToCloudinary(buffer, folder, resourceType, publicId)`** — generic upload
- **`uploadDocument()` (deprecated)** — kept for backward compatibility; documents should use Backblaze B2

### `backblazeService.js`

- Connects via `backblaze-b2` npm package (S3-compatible)
- Authorizes lazily (promise-lock prevents race conditions)
- **`uploadDocument(buffer, folder, originalFilename)`** — uploads to B2, returns 7-day signed URL
- **`getSignedUrl(fileName, validDurationInSeconds)`** — generates signed download URL for existing files
- Folders used: `profile-documents/`, `marksheets/`, `certificates/`, `other-documents/`

### `emailService.js`

- **`sendVerificationEmail({ email, name, token })`** — sends HTML verification email with link
- **`sendWelcomeEmail({ email, name })`** — sends welcome email after verification (non-critical, doesn't throw)
- Config: SMTP via Nodemailer, IPv4-forced, 10s timeouts, STARTTLS on port 587

### `eligibilityEngine.js`

See [Eligibility Engine](#eligibility-engine) section.

---

## Eligibility Engine

**File:** `server/services/eligibilityEngine.js`  
**Exported as:** singleton `new EligibilityEngine()`

The engine evaluates user profiles against scheme rule sets and returns weighted match percentages.

### Field Name Mapping

Rules use `snake_case` (e.g., `marital_status`), User model uses `camelCase` (`maritalStatus`). Engine maps automatically via `RULE_TO_USER_FIELD`.

### Operators Supported

`==`, `!=`, `<`, `<=`, `>`, `>=`, `in`, `not in`

### Special Cases

| Case                                | Handling                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| `disability` as boolean in rule     | Converts to `true` if user disability != `'None'`            |
| `education: "PhD"`                  | Aliased to `"Doctorate"` (both in user value and rule value) |
| `in`/`not in` with mismatched types | Uses loose equality (`==`) for number/string coercion        |
| Missing user field                  | Rule evaluates as no-match (returns `false`)                 |

### Core Methods

- **`evaluateRule(userProfile, rule)`** → Boolean
- **`calculateMatch(userProfile, scheme)`** → 0–100 (weighted %: matched weight / total weight × 100)
- **`findEligibleSchemes(userProfile, schemes)`** → sorted array of `{ scheme, matchPercentage, matchedRules }`
- **`getMatchedRules(userProfile, scheme)`** → array of rules that passed

---

## Frontend Architecture

### `AuthContext` (`/client/src/context/AuthContext.jsx`)

Central auth state manager. Provides:

| Value                         | Type           | Description                            |
| ----------------------------- | -------------- | -------------------------------------- |
| `user`                        | Object \| null | Full user data                         |
| `loading`                     | Boolean        | True while fetching user on mount      |
| `login(credentials)`          | async fn       | Email/password login                   |
| `register(userData)`          | async fn       | Register + auto-login                  |
| `logout()`                    | fn             | Clears localStorage + state            |
| `googleAuth(userData, token)` | fn             | Set state after Google OAuth           |
| `updateUserProfile(data)`     | async fn       | Update profile via API                 |
| `refreshUser()`               | async fn       | Re-fetch user from `/me`               |
| `setUserFromResponse(user)`   | fn             | Directly update user from API response |
| `isAuthenticated`             | Boolean        | `!!user`                               |
| `isUser`                      | Boolean        | `user?.role === 'user'`                |
| `isOrganizer`                 | Boolean        | `user?.role === 'organizer'`           |
| `isAdmin`                     | Boolean        | `user?.role === 'admin'`               |

### `api.js` (`/client/src/services/api.js`)

Axios instance with:

- Base URL: `VITE_API_BASE_URL` or `http://localhost:5000/api`
- **Request interceptor:** Attaches `Authorization: Bearer <token>` from localStorage
- **Response interceptor:** On 401 (non-auth endpoints) → removes token → redirects to `/login`

Exported API groups:

- `authAPI` — register, login, getMe, updateProfile, uploadProfileImage, uploadProfileDocuments
- `schemeAPI` — getAllSchemes, getSchemeById, getMatchedSchemes
- `organizerAPI` — createScheme, getMySchemes, updateScheme, deleteScheme, submitScheme
- `applicationAPI` — submitApplication, getMyApplications, getApplicationStatus
- `adminAPI` — all admin scheme + organizer application management
- `aiAPI` — generateScheme
- `supportQueryAPI` — create, getAll, getById, sendMessage, resolve
- `schemeApplicationAPI` — submitApplication (multipart), getMyApplications, checkApplication, getSchemeApplications, getAllApplications, approveApplication, rejectApplication

### Key Components

| Component           | Purpose                                                             |
| ------------------- | ------------------------------------------------------------------- |
| `ProtectedRoute`    | Route guard: unauthenticated → `/`, wrong role → role dashboard     |
| `Navbar`            | Role-aware nav bar; hidden on `/`, `/login`, `/register`            |
| `RuleBuilder`       | Visual UI for creating/editing scheme eligibility rules (organizer) |
| `ApplySchemeModal`  | Multi-step scheme application form with file uploads                |
| `ConfirmationModal` | Reusable confirm/cancel dialog                                      |
| `LandingComponents` | Landing page sub-components (hero, features, CTA sections)          |
| `Logo`              | Schemz brand logo                                                   |

---

## Client-Side Routing

All routes are defined in `App.jsx`. The `Navbar` is hidden on `/`, `/login`, `/register`.

### Public Routes

| Path        | Component  | Description                                                     |
| ----------- | ---------- | --------------------------------------------------------------- |
| `/`         | `Landing`  | Landing page — redirects authenticated users to their dashboard |
| `/login`    | `Login`    | Login (email/password + Google One Tap)                         |
| `/register` | `Register` | Registration                                                    |

### User Routes (`allowedRoles: ['user']`)

| Path                    | Component        |
| ----------------------- | ---------------- |
| `/user/dashboard`       | `UserDashboard`  |
| `/user/profile`         | `Profile`        |
| `/user/matched-schemes` | `MatchedSchemes` |
| `/user/all-schemes`     | `AllSchemes`     |
| `/user/scheme/:id`      | `SchemeDetail`   |
| `/user/apply-organizer` | `ApplyOrganizer` |
| `/user/my-applications` | `MyApplications` |
| `/user/support-queries` | `SupportQueries` |

### Organizer Routes (`allowedRoles: ['organizer']`)

| Path                                       | Component            |
| ------------------------------------------ | -------------------- |
| `/organizer/dashboard`                     | `OrganizerDashboard` |
| `/organizer/create-scheme`                 | `CreateScheme`       |
| `/organizer/my-schemes`                    | `MySchemes`          |
| `/organizer/edit-scheme/:id`               | `EditScheme`         |
| `/organizer/scheme-applications/:schemeId` | `SchemeApplications` |
| `/organizer/support-queries`               | `SupportQueries`     |

### Admin Routes (`allowedRoles: ['admin']`)

| Path                          | Component             |
| ----------------------------- | --------------------- |
| `/admin/dashboard`            | `AdminDashboard`      |
| `/admin/pending-schemes`      | `PendingSchemes`      |
| `/admin/all-schemes`          | `AdminAllSchemes`     |
| `/admin/pending-applications` | `PendingApplications` |
| `/admin/all-applications`     | `AllApplications`     |
| `/admin/support-queries`      | `SupportQueries`      |

### Catch-All

`*` → `CatchAllRedirect` — authenticated users go to their role dashboard; unauthenticated → `/login`

---

## Real-Time (Socket.io)

**Server setup:** HTTP server wraps Express app; Socket.io attached with CORS config.

**Authentication:** Socket middleware verifies JWT from `socket.handshake.auth.token` or `socket.handshake.query.token`. Unauthorized connections are rejected.

**Events (server-side):**
| Event | Direction | Description |
|---|---|---|
| `join-query` | Client → Server | Join a Socket.io room `query-{queryId}` |
| `leave-query` | Client → Server | Leave a query room |

**Rooms:** Named `query-{queryId}`. Used by `SupportQueries.jsx` to receive real-time chat updates when a new message is sent in a support query.

**Access in controllers:** `req.app.get('io')` provides the io instance for emitting to rooms.

---

## File Uploads

### Profile Image

- **Route:** `POST /api/auth/upload-profile-image`
- **Storage:** Cloudinary (`profile-images/` folder)
- **Limits:** 5MB, images only (`image/*`)
- **Multer config:** Memory storage, single file field `image`

### Profile Documents (stored in user profile)

- **Route:** `POST /api/auth/upload-profile-documents`
- **Storage:** Backblaze B2 (`profile-documents/` folder)
- **Limits:** 5MB each, PDF/JPG/JPEG/PNG
- **Fields:** `aadhaarDocument`, `incomeCertificate`, `categoryCertificate`

### Scheme Application Documents

- **Route:** `POST /api/scheme-applications`
- **Storage:** Backblaze B2
- **Limits:** 5MB each, PDF/JPG/JPEG/PNG
- **Fields:**
  - `marksheet` (required) → `marksheets/` folder
  - `incomeCertificate` (optional, falls back to profile document) → `certificates/` folder
  - `categoryCertificate` (optional, falls back to profile document) → `certificates/` folder
  - `otherDocuments` (up to 5 files) → `other-documents/` folder
- **URLs stored:** Signed Backblaze B2 URLs (7-day expiry per upload)

---

## Running the Project

### Server

```bash
cd server
npm install
cp .env.example .env   # fill in your credentials
npm run dev            # nodemon server.js on port 5000
```

### Client

```bash
cd client
npm install
# Create .env with VITE_API_BASE_URL=http://localhost:5000/api
npm run dev            # Vite dev server on port 5173
```

### Tests

```bash
cd server
npm test               # Jest + Supertest (NODE_ENV=test)
```

---

## Known Notes & Caveats

1. **Google OAuth duplicate route:** `POST /api/auth/google` is registered twice in `authRoutes.js` (once at top, once at bottom). Functionally equivalent; the first registration handles requests.

2. **Backblaze B2 signed URL expiry:** Uploaded document URLs are signed at time of upload with a 7-day expiry. Stored URLs in MongoDB will become invalid after 7 days — re-signing logic (`getSignedUrl`) exists in `backblazeService.js` but is not wired to automatic refresh.

3. **Cloudinary document upload deprecated:** `cloudinaryService.uploadDocument()` still exists but logs a deprecation warning. All new document uploads (profile + application) use Backblaze B2.

4. **Email verification:** `sendVerificationEmail` is implemented but the full email verification flow (token generation, `/verify-email` endpoint, blocking unverified users) is **not yet implemented** in the backend controllers.

5. **AI output normalization:** The `aiController` post-processes Gemini output to fix common issues (`PhD` → `Doctorate`, stringified numbers → actual numbers for age/income).

6. **User profile completeness requirement:** When submitting a scheme application, the backend validates that `name, email, phone, age, gender, category, state, education` are all present in applicant details — prompting users to complete their profile first.

7. **Organizer scheme ownership:** Organizers can only view/manage schemes they created. Admin has full access to all schemes.
