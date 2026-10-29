<p align="center">
  <img src="https://i.ibb.co.com/35BySv1C/image.png" alt="9Drive Cover" width="100%" />
</p>

# 9Drive — Multi-Cloud Virtual Storage Gateway

<p align="center">
  <a href="https://9drive.zenhosta.com"><img src="https://img.shields.io/badge/Live%20Demo-9drive.zenhosta.com-4F46E5?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Live Demo" /></a>
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge" alt="Apache 2.0 License" />
  <img src="https://img.shields.io/badge/Node.js-20%2B-339933?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js 20+" />
  <img src="https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React 18" />
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL" />
  <img src="https://img.shields.io/badge/Prisma-ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white" alt="Prisma" />
  <img src="https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
</p>

---

## 📌 Overview

**9Drive** is an open-source, high-performance storage gateway web application designed to merge multiple Google Drive accounts and S3-compatible cloud buckets into a single, unified virtual file dashboard. 

Instead of juggling storage limits across different accounts, 9Drive aggregates your total quota, organizes files in a virtual folder hierarchy, provides fast file previews, and intelligently streams file uploads directly to the connected storage provider that has available space—**without saving files to the host server disk**.

---

## ✨ Features

- 🌐 **Unified Multi-Account Gateway**: Connect multiple Google Drive accounts and S3-compatible object storage providers (AWS S3, Cloudflare R2, MinIO, Wasabi, Backblaze B2) into one aggregated pool.
- 🚀 **Zero-Disk Streaming Uploads**: Upload streams bypass local server storage and pipe straight to Google Drive or S3 endpoints with real-time transfer tracking.
- 🎯 **Smart Upload Routing**: Route files based on customizable policies:
  - **Most Available Space**: Dispatches file to the provider with greatest headroom.
  - **Round-Robin**: Balances storage distribution sequentially across accounts.
  - **Priority Order**: Fills primary accounts first before overflowing to secondary storage.
- 🔑 **External REST Upload API**: Programmatic file uploads (`POST /api/v1/uploads`) using cryptographically hashed, revocable API keys with rate-limiting and access audit logs.
- 📁 **Virtual Folder Tree**: Create, nest, rename, and reorganize folders without changing underlying cloud directory paths.
- 🔄 **Two-Way Synchronization**: Manually or automatically synchronize files created in the remote Google Drive `9drive` folder back into the MySQL catalog.
- 👁️ **File Previews & Sharing**: Built-in viewer for images, audio, video, PDFs, and code; generate time-limited or public shareable links.
- 🔒 **Enterprise-Grade Security**:
  - AES-256 encrypted storage for Google OAuth tokens and S3 credentials in the database.
  - Argon2/Bcrypt password hashing and short-lived JWT access tokens with rotating refresh tokens.
  - Optional Google reCAPTCHA v2/v3 support for registration.
- ⚙️ **In-App Management & Auto-Updates**: Configure Google OAuth directly from the Settings UI; trigger automated zero-downtime updates via `update.sh` (PM2 integration) with live deployment logs.
- 🐳 **Docker & Docker-Compose Ready**: Complete containerization for frontend, backend API, and MySQL.

---

## 📸 Screenshots & Preview

**Live Demo**: [https://9drive.zenhosta.com](https://9drive.zenhosta.com)

| Dashboard Overview | Shared File Preview |
| :---: | :---: |
| ![9Drive dashboard preview](https://i.ibb.co.com/HLjG3JRf/image.png) | ![9Drive shared file preview](https://i.ibb.co.com/QLpYGmx/image.png) |

---

## 🏗️ Architecture & Data Flow

```mermaid
flowchart TD
    Client["Web Browser / API Client"]
    Frontend["React + Vite Frontend (Port 5173 / Nginx)"]
    Backend["Express.js + TypeScript Backend (Port 4000)"]
    DB[("MySQL Database (Prisma ORM)")]
    GDrive["Google Drive API v3 (Folder: 9drive)"]
    S3Storage["S3-Compatible Storage (R2 / Wasabi / MinIO / S3)"]

    Client -->|Interacts with UI| Frontend
    Client -->|API Uploads & Auth| Backend
    Frontend -->|REST Requests & JWT| Backend
    Backend -->|Read/Write Metadata & Encrypted Tokens| DB
    Backend -->|Pipes File Stream Direct| GDrive
    Backend -->|Pipes File Stream Direct| S3Storage
```

### Upload Pipeline
1. Client sends multipart upload with file metadata.
2. Backend inspects storage policies (space remaining, active providers).
3. Selected cloud client opens a streaming write channel (`google.drive('v3').files.create` or `@aws-sdk/client-s3 Upload`).
4. Busboy streams chunks directly to the cloud pipe.
5. On completion, metadata record is committed to MySQL via Prisma.

---

## 📂 Project Structure

```txt
MY9Drive/
├── backend/                  # Node.js + Express + TypeScript API
│   ├── prisma/               # Database schema & migrations
│   │   ├── migrations/       # SQL migration history
│   │   └── schema.prisma     # Prisma Data Model (Users, Accounts, Files, Keys)
│   ├── src/
│   │   ├── app.ts            # Express application setup
│   │   ├── server.ts         # Server entry point
│   │   ├── config/           # App, OAuth, and database config
│   │   ├── middleware/       # Auth, error handling, rate limiting
│   │   ├── modules/          # Domain services (auth, files, storage, sync)
│   │   └── utils/            # Cryptography, tokens, streaming helpers
│   ├── Dockerfile            # Backend container definition
│   └── package.json
├── frontend/                 # React 18 + Vite + Tailwind CSS SPA
│   ├── src/
│   │   ├── App.tsx           # Route container & global providers
│   │   ├── components/       # UI components (UploadModal, FileGrid, QuotaBar)
│   │   ├── context/          # Auth & storage contexts
│   │   ├── pages/            # Dashboard, Explorer, Settings, Shared, Quota
│   │   └── lib/              # API clients & utility functions
│   ├── Dockerfile            # Multi-stage Nginx container build
│   └── vite.config.ts        # Vite configuration & proxy rules
├── docker-compose.yml        # Orchestration for frontend, backend, and MySQL
├── setup.sh                  # Interactive Linux/macOS setup script
├── setup.ps1                 # Interactive Windows PowerShell setup script
└── update.sh                 # Production PM2 deployment & self-update script
```

---

## 🚀 Quick Start (Automated Installation)

The setup script automates dependency installation, environment file generation (`backend/.env` & `frontend/.env`) with secure random secrets, and Prisma client generation.

### Linux / macOS

```bash
git clone https://github.com/ShihabulSakib/MY9Drive.git
cd MY9Drive
chmod +x setup.sh
./setup.sh
```

### Windows (PowerShell)

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\setup.ps1
```

Once initialized, start the services:

```bash
# Terminal 1 - Backend
cd backend
npm run prisma:migrate
npm run dev

# Terminal 2 - Frontend
cd frontend
npm run dev
```

Visit `http://localhost:5173` in your browser.

---

## 🐳 Docker Deployment

To spin up the entire application stack including MySQL using Docker Compose:

```bash
# 1. Copy sample environment files if not already created
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# 2. Launch all services
docker-compose up -d --build

# 3. Check container logs
docker-compose logs -f
```

Frontend will be available at `http://localhost:80` (or configured port) and backend API at `http://localhost:4000`.

---

## 🛠️ Manual Installation & Configuration

### 1. Prerequisites
- **Node.js**: v20.x or higher
- **npm**: v10.x or higher
- **MySQL**: v8.0 or higher
- **Google Cloud Console Account**: For OAuth Client credentials & Drive API.

### 2. Database Preparation
Create a dedicated database in MySQL:

```sql
CREATE DATABASE `9drive` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 3. Backend Environment Setup
Create `backend/.env`:

```env
DATABASE_URL="mysql://root:password@localhost:3306/9drive"
APP_PORT=4000
FRONTEND_URL="http://localhost:5173"

# Generate 32+ character random strings
JWT_ACCESS_SECRET="generate-a-strong-jwt-secret-key-32-chars-min"
TOKEN_ENCRYPTION_KEY="generate-a-32-byte-hex-or-string-key"

ACCESS_TOKEN_TTL_SECONDS=900
REFRESH_TOKEN_TTL_DAYS=30
MAX_UPLOAD_BYTES=5368709120

# Optional Google reCAPTCHA
RECAPTCHA_SECRET_KEY=""

# Optional Seed Credentials (can also be entered via Settings UI)
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GOOGLE_REDIRECT_URI="http://localhost:4000/connected-accounts/google/callback"
```

### 4. Frontend Environment Setup
Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:4000
VITE_RECAPTCHA_SITE_KEY=
```

### 5. Database Migrations
```bash
cd backend
npm install
npm run prisma:migrate
```

---

## 🔑 Google Cloud OAuth Setup

1. Open [Google Cloud Console](https://console.cloud.google.com/).
2. Create or select your project.
3. Enable **Google Drive API**:
   - Navigate to `APIs & Services` > `Library`.
   - Search for `Google Drive API` and click **Enable**.
4. Configure **OAuth Consent Screen**:
   - User Type: `External`.
   - Add scope: `.../auth/drive` or `.../auth/drive.file`.
   - In Testing mode, add your Google account email under **Test Users**.
5. Create **OAuth 2.0 Client IDs** (`Web application`):
   - **Authorized JavaScript origins**: `http://localhost:5173` (and production domain)
   - **Authorized redirect URIs**: `http://localhost:4000/connected-accounts/google/callback`
6. Enter credentials either in `backend/.env` or directly inside **Settings > Google Credentials** in the running 9Drive dashboard (credentials are encrypted into the database).

---

## 📡 REST API Reference

### Authentication
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/auth/register` | Register new user account |
| `POST` | `/auth/login` | Login with email and password |
| `GET` | `/auth/google/url` | Retrieve Google OAuth login redirect URL |
| `GET` | `/auth/google/callback`| OAuth callback handler |
| `POST` | `/auth/refresh` | Refresh expired access JWT |
| `POST` | `/auth/logout` | Revoke session refresh token |
| `GET` | `/auth/me` | Fetch authenticated user profile |

### Connected Storage Providers
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/connected-accounts` | List all connected Drive and S3 accounts |
| `GET` | `/connected-accounts/google/connect-url` | Generate Google Drive OAuth link |
| `POST` | `/connected-accounts/:id/sync-quota` | Synchronize quota usage for account |
| `DELETE` | `/connected-accounts/:id` | Disconnect storage account |

### Files & Folders
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/storage/summary` | Aggregate total storage, used, and free bytes |
| `GET` | `/folders` | List virtual directory tree |
| `POST` | `/folders` | Create a new virtual folder |
| `DELETE`| `/folders/:id` | Remove virtual folder |
| `GET` | `/files` | List files with optional search & folder filtering |
| `POST` | `/uploads` | Multipart file upload (zero-disk stream) |
| `GET` | `/files/:id/view-url` | Get preview stream URL |
| `GET` | `/files/:id/download` | Download file with original filename |
| `POST` | `/files/:id/share` | Generate public share token |
| `DELETE`| `/files/:id` | Delete file from catalog & remote storage |

---

## 🛡️ Security Highlights

- **Zero Local Footprint**: Uploaded files never touch the server filesystem; they are streamed dynamically to remote cloud storage.
- **Encrypted Secrets**: Sensitive OAuth refresh tokens and S3 keys are encrypted using AES-256-GCM before database insertion.
- **Hashed Session Tokens**: Session refresh tokens and public share tokens are stored as cryptographic hashes to safeguard against database leak exposure.
- **Sanitized Filenames & Inputs**: Path traversal mitigations and strict mime-type validation.

---

## 🔄 Production Maintenance & PM2 Updates

For self-hosted Linux VPS setups using PM2:

```bash
# Automated system update
./update.sh
```

Or trigger manual execution:
```bash
git pull origin main
cd backend && npm install && npx prisma migrate deploy && npm run build
cd ../frontend && npm install && npm run build
pm2 restart 9drive-backend
```

---

## 📈 Star History

[![Star History Chart](https://star-history.dera.page/svg?repos=zenhosta/9drive&type=Date)](https://star-history.dera.page/#zenhosta/9drive&type=Date)

---

## 📄 License

Copyright © 2026 Zenhosta.  
Licensed under the [Apache License, Version 2.0](LICENSE).
