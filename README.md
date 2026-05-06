# GCP Mobile File Manager (CloudBucket)

A production-oriented mobile file manager that lets users authenticate with Google, discover their Google Cloud Storage buckets, and securely manage files from a React Native app.
The platform combines a modern Expo mobile client with a FastAPI backend, Supabase authentication, and GCP Storage to deliver fast, secure, and scalable cloud file operations.

---

## 1) Project Overview

CloudBucket is a mobile-first application designed for users who need direct, reliable access to their own Google Cloud Storage (GCS) buckets.
It focuses on secure identity management, low-latency file operations, and clean architecture that separates authentication, API orchestration, and storage execution.

The system uses:
- **Supabase + Google OAuth** for user identity and session management.
- **FastAPI** for backend orchestration, token validation, bucket/file metadata operations, and presigned URL generation.
- **GCP Storage** as the final data layer where files are stored and served.
- **React Native (Expo) + NativeWind** for a cross-platform and maintainable mobile UI.

---

## 2) System Architecture

### High-Level Request Flow

1. User signs in with Google via Supabase on the mobile app.
2. Mobile app receives Supabase session tokens.
3. Mobile app calls FastAPI endpoints with Supabase JWT.
4. FastAPI validates JWT and retrieves user-linked cloud context.
5. FastAPI uses GCP credentials (service account or delegated access) to:
   - create/list/update/delete buckets,
   - list/create/update/delete file metadata operations,
   - create presigned upload/download URLs.
6. Mobile app uploads/downloads directly to/from GCP using presigned URLs.
7. Backend records metadata/audit events (optional) for observability and traceability.

### Architecture Diagram (Logical)

```text
[React Native App]
   |  (Google OAuth via Supabase)
   v
[Supabase Auth] -----> [Supabase DB (optional profile/audit)]
   |
   | (JWT)
   v
[FastAPI Backend]
   |  - Verify JWT
   |  - Bucket/File APIs
   |  - Presigned URL generation
   v
[Google Cloud Storage]
```

### Component Responsibility Table

| Component | Responsibility | Notes |
|---|---|---|
| Mobile App (Expo) | UI/UX, auth flow, file picker, upload/download actions | Uses NativeWind for scalable styling |
| Supabase Auth | Google OAuth, session/token issuance | Centralized identity provider |
| FastAPI API Layer | Authorization checks, business rules, GCP integration | Deploy on Vercel or VPS |
| GCP Storage | Durable file storage and retrieval | Accessed via secure signed URLs |

---

## 3) Key Features

- Google OAuth login powered by Supabase.
- Full bucket CRUD (create, list, update metadata, delete) per authenticated user policy.
- File listing with metadata (name, size, type, updated time).
- Direct file upload from device using **presigned URLs**.
- Direct file download/view using **presigned URLs**.
- File delete operations with server-side authorization checks.
- Extensible API design for future sharing, foldering, and tagging.

---

## 4) Folder Structure

```text
cloudbucket/
├─ frontend/
│  ├─ app/                         # Expo Router screens/routes (or /src/navigation if using React Navigation)
│  ├─ src/
│  │  ├─ components/               # Reusable UI components
│  │  │  ├─ common/
│  │  │  ├─ file/
│  │  │  └─ bucket/
│  │  ├─ screens/                  # Screen-level containers (if not fully route-based)
│  │  ├─ services/
│  │  │  ├─ api/                   # Axios/fetch clients + endpoint wrappers
│  │  │  ├─ auth/                  # Supabase auth helpers
│  │  │  └─ storage/               # Upload/download helpers
│  │  ├─ store/                    # Zustand/Redux state slices
│  │  ├─ hooks/                    # Custom hooks
│  │  ├─ utils/                    # Utility functions
│  │  ├─ types/                    # TypeScript types/interfaces
│  │  ├─ constants/                # App constants and config keys
│  │  └─ theme/                    # NativeWind theme tokens/extensions
│  ├─ assets/
│  ├─ .env.example
│  ├─ app.json
│  ├─ tailwind.config.js
│  ├─ babel.config.js
│  └─ package.json
│
├─ backend/
│  ├─ app/
│  │  ├─ main.py                   # FastAPI entrypoint
│  │  ├─ core/
│  │  │  ├─ config.py              # Environment and settings
│  │  │  ├─ security.py            # JWT verification and auth guards
│  │  │  └─ logging.py
│  │  ├─ api/
│  │  │  ├─ deps.py                # Shared API dependencies
│  │  │  └─ routes/
│  │  │     ├─ auth.py             # Auth/session helper endpoints
│  │  │     ├─ buckets.py          # Bucket listing and details
│  │  │     ├─ files.py            # File list/delete operations
│  │  │     └─ signed_urls.py      # Generate upload/download signed URLs
│  │  ├─ services/
│  │  │  ├─ gcp_storage.py         # GCS interactions
│  │  │  └─ supabase_client.py
│  │  ├─ schemas/                  # Pydantic models
│  │  └─ tests/
│  │     ├─ test_buckets.py
│  │     ├─ test_files.py
│  │     └─ test_signed_urls.py
│  ├─ requirements.txt
│  ├─ .env.example
│  ├─ vercel.json                  # Optional if deploying to Vercel
│  └─ Dockerfile                   # Optional for VPS/container deployment
│
├─ docs/
│  ├─ architecture.md
│  ├─ api-spec.md
│  └─ security.md
│
└─ README.md
```

---

## 5) Tech Stack Details

| Layer | Technology | Why It Was Chosen |
|---|---|---|
| Mobile Frontend | React Native + Expo | Fast cross-platform delivery, OTA updates, strong ecosystem |
| Styling | NativeWind | Utility-first styling consistency with Tailwind-like DX |
| Backend API | FastAPI | High performance, async support, strong type validation with Pydantic |
| Auth + Data | Supabase | Managed auth flows (Google OAuth), simple JWT-based integration |
| Cloud Storage | Google Cloud Storage | Reliable object storage with signed URL support |
| Deployment | Vercel or VPS | Flexible choice between serverless simplicity and full control |

---

## 6) Implementation Roadmap

### Phase 1: Auth & Foundation Setup

- Initialize Expo app with TypeScript + NativeWind.
- Configure Supabase project and Google OAuth provider.
- Implement mobile sign-in/sign-out/session restore.
- Set up FastAPI skeleton with environment-driven config.
- Add JWT verification middleware/dependencies.

### Phase 2: Backend API for GCP Integration

- Configure GCP service account and storage client wrapper.
- Build bucket CRUD endpoints (`POST/GET/PATCH/DELETE /buckets`).
- Build file endpoints (`GET /buckets/{bucket}/files`, delete, and signed URL flows).
- Add secure authorization guard per user or tenant mapping.
- Implement structured error handling and logging.
- Add basic integration tests.

### Phase 3: Frontend UI with NativeWind

- Build bucket list screen and file list/detail screen.
- Add file picker integration (images/docs/media).
- Add loading, empty, and error states.
- Implement reusable components for list rows, buttons, and modals.
- Connect screens with backend API services.

### Phase 4: Upload/Download via Presigned URLs

- Add backend endpoints:
  - `POST /files/upload-url`
  - `POST /files/download-url`
- Use signed URLs in mobile app for direct transfer to GCP.
- Track progress and handle retry/cancel behavior.
- Add delete endpoint and confirmation UX.
- Harden with file type/size checks and URL expiration controls.

---

## 7) Security Best Practices

- Keep OAuth scopes minimal (principle of least privilege).
- Validate and verify all Supabase JWTs server-side before GCP operations.
- Never expose GCP service account keys in the mobile client.
- Use short-lived presigned URLs with strict object path constraints.
- Enforce content type and max file size rules before issuing upload URLs.
- Store secrets only in secure environment variables and secret managers.
- Add request logging/audit trails for sensitive operations (upload/delete).
- Rotate credentials periodically and monitor abnormal access patterns.

---

## 8) Setup Instructions

### Prerequisites

- Node.js + npm/yarn + Expo CLI
- Python 3.10+
- Supabase project with Google OAuth enabled
- GCP project + Storage buckets + service account

### Environment Variables

#### `frontend/.env`

```bash
EXPO_PUBLIC_SUPABASE_URL=your_supabase_project_url
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
EXPO_PUBLIC_API_BASE_URL=https://your-backend-domain.com
```

#### `backend/.env`

```bash
APP_ENV=development
APP_PORT=8000

SUPABASE_URL=your_supabase_project_url
SUPABASE_JWT_SECRET=your_supabase_jwt_secret

GCP_PROJECT_ID=your_gcp_project_id
GCP_DEFAULT_BUCKET=optional_default_bucket_name
GCP_SERVICE_ACCOUNT_JSON='{"type":"service_account",...}'

SIGNED_URL_EXPIRY_SECONDS=900
MAX_UPLOAD_SIZE_MB=50
ALLOWED_MIME_TYPES=image/jpeg,image/png,application/pdf
```

### Run Locally

#### Frontend

```bash
cd frontend
npm install
npx expo start
```

#### Backend

```bash
cd backend
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

---

## API Starter Endpoints (Suggested)

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Service health check |
| POST | `/buckets` | Create a new bucket (policy-controlled) |
| GET | `/buckets` | List user-accessible buckets |
| PATCH | `/buckets/{bucket}` | Update bucket metadata/config (safe subset) |
| DELETE | `/buckets/{bucket}` | Delete an empty bucket (or force policy) |
| GET | `/buckets/{bucket}/files` | List files in bucket |
| POST | `/files/upload-url` | Generate presigned upload URL |
| POST | `/files/download-url` | Generate presigned download URL |
| DELETE | `/buckets/{bucket}/files/{path}` | Delete a file |

---

## Future Enhancements

- Background uploads and resumable transfer support.
- Folder simulation, search, and filtering.
- Multi-account cloud support.
- In-app preview for common file types.
- Role-based sharing and collaboration controls.

