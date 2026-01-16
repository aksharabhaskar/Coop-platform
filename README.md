# Co-op Dashboard

## Overview

Co-op Dashboard is a full-stack overnight hackathon project that streamlines operations for cooperative credit societies. The platform offers a unified workspace for members, managers, and auditors to manage deposits, loans, and compliance while anchoring financial events to both a relational database and an Ethereum Sepolia smart contract for tamper-evident audit trails.

## Architecture

- **Frontend**: React + Vite single-page application with role-aware dashboards, Axios-based data layer, and real-time polling for session-aware updates.
- **Backend**: Go 1.24 service built on Echo, GORM, and a modular repository layer; issues signed session cookies and enforces rate limits, role-based access control, and OTP/Twilio integrations.
- **Data & Blockchain**: PostgreSQL (or SQLite) primary store, with mirrored transaction hashes recorded on Sepolia via the TransactionLedger contract and on-demand verification endpoints.
- **Documentation**: Swagger UI exposed at `/swagger` and synced OpenAPI description in `backend/src/docs` and `frontend/openapi.json`.

## Tech Stack

- Go 1.24, Echo, GORM, Swag, Twilio SDK
- React 19, Vite 7, Axios, Lucide React icons
- PostgreSQL or SQLite for development, Ethereum Sepolia for on-chain verification

## Backend Setup (`backend/`)

### Prerequisites

- Go 1.24.x toolchain (see `go.mod` for toolchain version)
- PostgreSQL 14+ **or** SQLite (for quick local runs)
- Optional: Twilio account for SMS OTP, Sepolia RPC provider (Alchemy, Infura, QuickNode) and funded wallet key for on-chain anchoring

### Environment Configuration

1. Copy the sample configuration:
   ```bash
   cd backend
   cp .env.example .env
   ```
2. Set the following values in `.env`:
   - `DATABASE_URL`: e.g. `postgresql://user:pass@localhost:5432/coop_dashboard?sslmode=disable` or `sqlite:///tmp/dev.db`
   - `TWILIO_*`: enable OTP SMS (leave blank to skip SMS delivery in dev)
   - `SEPOLIA_RPC_URL`, `CONTRACT_ADDRESS`, `PRIVATE_KEY`: required for blockchain writes; the service gracefully downgrades to local verification if absent
   - `PORT`: default `8080`

### Install and Run

```bash
cd backend
make install        # go mod download + tidy
make run            # go run src/main.go
```

The API listens on `http://localhost:8080`. Rate limits and session cookies require HTTPS in production; locally the Echo CORS policy already trusts Vite dev origins.

### Useful Commands

- `make dev`: hot-reload with Air (installs automatically on first run)
- `make test`: execute Go unit tests (none bundled yet, scaffold ready)
- `make swagger`: regenerate Swagger docs after handler changes

## Frontend Setup (`frontend/`)

### Prerequisites

- Node.js 20.x (Vite 7 requires Node 19 or later; prefer 20 LTS)
- pnpm, npm, or yarn (examples use npm)

### Environment Configuration

1. Copy the sample env file:
   ```bash
   cd frontend
   cp .env.example .env
   ```
2. For local API access, set `VITE_API_URL=http://localhost:8080/api/v1`.
3. The Vite dev server proxies `/api` to the hosted staging API as defined in `vite.config.js`; adjust or remove the proxy if you want all traffic to hit your local backend.

### Install and Run

```bash
cd frontend
npm install
npm run dev
```

The app starts on `http://localhost:5173` and automatically attaches session cookies (`withCredentials`) for authenticated calls.

### Build and Quality Checks

- `npm run build`: production build into `dist/`
- `npm run lint`: lint with the project ESLint ruleset

## API Notes

- Session auth is cookie-based; ensure `withCredentials=true` on client requests and serve over HTTPS in production.
- Role-specific endpoints live under `/api/v1` with manager/auditor middleware guards (see `backend/src/routes/routes.go`).
- Exports such as `GET /api/v1/audit/transactions/export` stream Excel files generated with `excelize`.
- Blockchain integrity endpoints (`/api/v1/audit/blockchain/status`) compare database records against Sepolia confirmations.

## Project Layout

```
backend/
  src/
    handlers/     # Auth, loans, deposits, audit modules
    middleware/   # Session auth, rate limiting
    repos/        # Data access layer and external integrations (Twilio SMS)
    blockchain/   # Sepolia client and verification helpers
    db/           # GORM models, migrations, blockchain initialization
frontend/
  src/
    assets/components/   # Role-specific dashboards & forms
    services/api.js      # Axios layer matching backend routes
    utils/asyncHandler.js
```

## Hackathon Credits

Built overnight by:

- Ranjith RD
- Akshara Bhaskar
- Gauri Girish Dhanakshirur
- Rishon Thomas Joby


