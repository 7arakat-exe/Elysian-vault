# Elysian Vault

Elysian Vault is a secure, web-based document management and sharing platform built to address a gap in mainstream cloud storage: weak default authentication, uncontrolled link sharing, no tiered protection for highly sensitive files, and poor auditability. It gives users a server-controlled, encrypted environment for storing, sharing, and collaborating on documents — without sacrificing usability.

## Key Features

- **Encrypted file storage** — every uploaded file is encrypted at rest on the server using **AES-256-CTR**, with a unique IV generated per file.
- **Secure Vault** — a separate, PIN-protected high-security area for the most sensitive files, with optional **self-destruct timers** for automatic deletion.
- **Granular file sharing** — generate share links with configurable view/download permissions, expiration dates, optional recipient emails, and the ability to activate/deactivate or delete a share at any time.
- **Multi-Factor Authentication (MFA)** — TOTP-based MFA via authenticator apps, with support for org-wide enforcement.
- **Team collaboration** — create teams, invite members, assign roles (owner/admin/member), and manage shared team file spaces with configurable permissions.
- **File integrity verification** — SHA-256 hashing lets users confirm a stored file hasn't been tampered with.
- **Activity logging** — detailed, timestamped logs of user and admin actions (uploads, downloads, shares, vault access, settings changes) for auditability.
- **Admin dashboard** — user management, department management (with vault access/quota/MFA policy per department), system-wide settings, file oversight, and system-wide activity monitoring.

## Tech Stack

**Frontend**
- React + TypeScript, built with Vite
- Tailwind CSS for styling
- Framer Motion for animations
- React Router DOM for client-side routing

**Backend**
- Node.js + Express (REST API)
- PostgreSQL, accessed via the Sequelize ORM
- JSON Web Tokens (JWT) for session management
- bcrypt for password/PIN hashing
- Multer for file upload handling
- Nodemailer for transactional email (password resets, team invites)
- Joi for input validation

**Architecture**

Elysian Vault follows a standard client-server model: a React SPA communicates with a stateless Express REST API over HTTPS, with PostgreSQL as the persistence layer. Frontend and backend are deployed as independent services (originally on DigitalOcean App Platform — a static site for the frontend, a web service for the backend, and a managed PostgreSQL instance).

## Security Design

- **Encryption at rest:** File content is encrypted server-side with AES-256-CTR before being written to disk. A unique IV is generated per file, stored in the database, and prepended to the encrypted file for decryption.
- **Password & PIN hashing:** User passwords and vault PINs are hashed with bcrypt via Sequelize `beforeCreate` hooks — raw values are never persisted.
- **Vault access control:** Vault files require a separate 6-digit PIN (sent via a custom `X-Vault-PIN` header), checked against a stored bcrypt hash, independent of the main account login.
- **Transport security:** All client-server communication is intended to run over HTTPS.
- **Integrity checks:** A SHA-256 hash of each encrypted file is stored and can be recalculated on demand to detect tampering or corruption.

## Getting Started

### Prerequisites
- Node.js
- PostgreSQL

### Installation

```bash
# Clone the repository
git clone <repo-url>
cd elysian-vault

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### Environment Variables

Create a `.env` file in `backend/` with the following:

```env
PORT=5000
DATABASE_URL=postgres://user:password@localhost:5432/elysian_vault
TOKEN_SECRET=your_jwt_secret
EMAIL_SERVICE=your_email_service
EMAIL_USER=your_email_user
EMAIL_PASSWORD=your_email_password
EMAIL_FROM=noreply@example.com
FRONTEND_URL=http://localhost:5173
```

Create a `.env` file in `frontend/` with:

```env
VITE_API_BASE_URL=http://localhost:5000/api
```

### Running Locally

```bash
# Start the backend
cd backend
npm run dev

# Start the frontend (in a separate terminal)
cd frontend
npm run dev
```

The app will be available at `http://localhost:5173`, with the API running at `http://localhost:5000`.

## Project Structure

```
backend/
├── config/       # DB config, encryption logic, validation schemas
├── middleware/    # Auth checks, MFA enforcement, admin checks
├── models/        # Sequelize models + associations
├── routes/        # Express route handlers, grouped by resource
├── services/       # Activity logging, file expiration, etc.
└── uploads/        # Temp / encrypted / profile image storage

frontend/
├── components/     # Reusable UI components (Button, Modal, FileCard, etc.)
├── pages/          # Top-level views (Dashboard, Vault, Teams, Admin, etc.)
├── services/        # API communication layer (api.ts)
├── utils/           # Utility functions
└── types/           # Shared TypeScript types
```

## API Overview

The backend exposes a REST API under `/api`, organized by resource:

| Route | Purpose |
|---|---|
| `/api/auth` | Registration, login, MFA verification, password reset |
| `/api/profile` | View/update profile, change password |
| `/api/mfa` | MFA setup, verification, disable |
| `/api/files` | Upload, list, download, view, delete, integrity check |
| `/api/vault` | Secure vault operations (PIN-gated) |
| `/api/shares` | Create/manage file share links |
| `/api/share` | Public access to shared files via token (no auth required) |
| `/api/teams` | Team creation, membership, team file access, settings |
| `/api/notifications` | Team invitation notifications |
| `/api/admin` | User/file/department management, system settings, system-wide logs |
| `/api/activities` | View personal activity log |

## Testing

Testing covered integration testing of API endpoints (via Postman), frontend-backend interaction, and dedicated security testing — input validation, authentication/authorization flows (including MFA and vault PIN verification), and dependency vulnerability scanning. Detailed test cases spanned authentication, profile management, file management, the vault, team collaboration, sharing, and the admin panel.

## Deployment

Frontend and backend are deployed as separate services for independent scaling and deployment cycles:
- **Frontend:** Built with `vite build` and served as a static site.
- **Backend:** Deployed as a Node.js web service.
- **Database:** A managed PostgreSQL instance with SSL-enforced connections.

## Author

Built by Ziad Harakat as a final year development project, exploring server-side encryption, layered access control, and secure-by-design architecture for document management systems.
