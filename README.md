# Secure File Sharing System

## Project summary

Secure File Sharing System is a local Flask and React application for uploading files, sharing them with registered users, and creating password-protected public links. The backend stores app records in SQLite and uploaded files on disk. The frontend is a Vite single-page app served by Flask after it is built.

## Tech stack

- Backend: Python, Flask, SQLite, bcrypt, PyJWT
- Frontend: React, Vite, React Router, Tailwind CSS v4, shadcn/ui Base UI components
- Storage: SQLite database in `instance/sfs.db` and uploaded files in `uploads/`

## Folder structure

```text
.
|-- app.py                    # Flask entry point
|-- backend/app/              # Backend config, database helpers, routes, services
|-- frontend/                 # Vite React application
|   |-- public/               # Static frontend assets
|   `-- src/                  # App shell, features, shared UI, styles
|-- instance/sfs.db           # Empty handover SQLite database
|-- uploads/                  # Runtime upload folder, kept empty in handover
|-- schema.sql                # SQLite schema used to recreate the database
|-- requirements.txt          # Python runtime dependencies
|-- .env.example              # Environment variable template
`-- README.md                 # Handover documentation
```

## Setup instructions

Prerequisites:

- Python 3.11+
- Node.js and npm

Install backend dependencies:

```powershell
python -m pip install -r requirements.txt
```

Install frontend dependencies:

```powershell
cd frontend
npm install
```

## Run and build commands

Run the backend API and built frontend:

```powershell
python app.py
```

Run the frontend dev server with API proxying to Flask:

```powershell
cd frontend
npm run dev
```

Build the frontend for Flask to serve:

```powershell
cd frontend
npm run build
```

Preview the built frontend with Vite:

```powershell
cd frontend
npm run preview
```

No lint or test commands are currently configured in the handover repository.

## Environment variables

The backend reads environment variables from the process environment. `.env.example` documents the expected keys, but `.env` is intentionally not kept in the repository.

| Variable | Default | Purpose |
| --- | --- | --- |
| `SECRET_KEY` | `dev-secret-change-me` | Flask app secret and fallback JWT signing key. Set this for any shared environment. |
| `JWT_SECRET_KEY` | `SECRET_KEY` value | JWT signing key. Set this for any shared environment. |
| `JWT_ALGORITHM` | `HS256` | JWT signing algorithm. |
| `JWT_EXPIRES_MINUTES` | `120` | Access token lifetime in minutes. |
| `SFS_DATABASE` | `instance/sfs.db` | SQLite database path. |
| `SFS_UPLOAD_FOLDER` | `uploads` | Folder used for uploaded files. |
| `FLASK_RUN_HOST` | `127.0.0.1` | Host used by `python app.py`. |
| `FLASK_RUN_PORT` | `5000` | Port used by `python app.py`. |
| `FLASK_DEBUG` | `0` | Set to `1` to enable Flask debug mode locally. |

## Database reset and setup

The database schema is defined in `schema.sql`. The app creates the database automatically when Flask starts.

Reset local app data to a clean empty state:

```powershell
Remove-Item -LiteralPath instance\sfs.db -Force -ErrorAction SilentlyContinue
Get-ChildItem -LiteralPath uploads -Force | Where-Object { $_.Name -ne '.gitkeep' } | Remove-Item -Recurse -Force
python -c "from backend.app import create_app; create_app()"
```

After reset, all application tables should exist with zero records and `uploads/` should contain only `.gitkeep`.

## Main features

- Account registration, sign-in, sign-out, and current-user JWT checks
- User-visible audit log for core auth, file, share, and link events
- Login history and suspicious-login recording after repeated failed attempts
- File upload with extension and size validation
- SHA-256 file integrity verification before download
- Owner downloads with download audit logging
- User-to-user sharing with password, optional expiry, and optional download limits
- Public link sharing with password, optional expiry, and optional download limits
- Dashboard summary with recent activity and charted upload/download history
- Light and dark theme support

## Important implementation notes

- Flask registers all API routes under `/api` and serves `frontend/dist` for the built SPA.
- Uploaded files are stored on disk; database rows store the original name, generated stored name, path, size, MIME type, and SHA-256 hash.
- Downloads fail if the stored file is missing or its hash no longer matches the database hash.
- Deleting an owned file removes its physical file and cascades related shares, links, and logs through SQLite foreign keys.
- Public links and user shares decrement remaining downloads only after a successful integrity check.
- The frontend stores the access token in `localStorage` as `sfs_access_token` and sends protected API requests with `Authorization: Bearer <token>`.

## Handover notes

- The repository has no Git metadata in this folder at handover time.
- Runtime data was cleared for handover: users, files, shares, links, login logs, download logs, uploaded files, and stale databases were removed.
- Tests and test-only dependencies were removed per cleanup scope.
- Generated folders are not kept: `frontend/node_modules`, `frontend/dist`, Python `__pycache__`, and pytest cache output.
- Keep `schema.sql`, `package-lock.json`, `.env.example`, and the source folders; they are required to recreate and maintain the app.
