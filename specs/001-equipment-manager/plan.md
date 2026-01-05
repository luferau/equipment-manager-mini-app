# Implementation Plan: Equipment Manager Mini App

**Branch**: `001-equipment-manager` | **Date**: 2026-01-05 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-equipment-manager/spec.md`

## Summary

Build a Telegram Mini App for managing laboratory equipment (measuring instruments, hardware, materials) with QR code scanning for instant equipment lookup, location tracking, file attachments, and complete audit trails. The system uses Vue 3 + Vuetify frontend served as a Telegram Mini App, with a Node.js/Express backend using SQLite for persistence.

## Technical Context

**Language/Version**: TypeScript 5.x (frontend), Node.js 20 LTS (backend)
**Primary Dependencies**: Vue 3, Vuetify 3, Pinia, Vite (frontend); Express.js, Prisma, node-telegram-bot-api, Zod (backend)
**Storage**: SQLite 3 (single-file database), local filesystem for attachments
**Testing**: Vitest (frontend), Jest or Vitest (backend)
**Target Platform**: Telegram Mini App (iOS/Android via Telegram client, WebApp API 6.9+)
**Project Type**: Web application (frontend + backend monorepo)
**Performance Goals**: QR scan to details <3s, search results <1s, 1000+ equipment items, 50 concurrent users
**Constraints**: 10MB max file upload, offline-capable with sync queue, mobile-first touch targets (44x44px)
**Scale/Scope**: Single organization, ~1000 equipment items, ~50 users, ~10 screens

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Evidence |
|-----------|--------|----------|
| I. Telegram Mini App Native | ✅ PASS | Using WebApp API 6.9+, native QR scanner, Telegram auth |
| II. Vue + Vuetify First | ✅ PASS | Vue 3 Composition API, Vuetify 3, Pinia |
| III. QR-Centric Workflow | ✅ PASS | Telegram native scanner, haptic feedback, printable QR codes |
| IV. Equipment Traceability | ✅ PASS | Immutable history, all actions tracked with user/timestamp |
| V. Mobile-First UX | ✅ PASS | 44px touch targets, thumb-reachable actions, skeleton screens |
| VI. Data Integrity & Audit Trail | ✅ PASS | Soft-delete only, checksums for files, audit log retention |
| VII. Node.js Backend & SQLite | ✅ PASS | Node.js 20, SQLite, Prisma ORM, Express.js |

**All gates passed. Proceeding to Phase 0.**

## Project Structure

### Documentation (this feature)

```text
specs/001-equipment-manager/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (OpenAPI spec)
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── models/           # Prisma schema and generated client
│   ├── services/         # Business logic (equipment, users, attachments)
│   ├── api/              # Express routes and middleware
│   ├── bot/              # Telegram bot commands and handlers
│   └── utils/            # Shared utilities (auth validation, file handling)
├── prisma/
│   └── schema.prisma     # Database schema
├── uploads/              # File attachment storage
└── tests/
    ├── integration/      # API endpoint tests
    └── unit/             # Service logic tests

frontend/
├── src/
│   ├── components/       # Reusable Vue components
│   │   ├── equipment/    # Equipment-related components
│   │   ├── common/       # Shared UI components
│   │   └── layout/       # App shell, navigation
│   ├── pages/            # Route-level views
│   │   ├── Home.vue
│   │   ├── Scan.vue
│   │   ├── Equipment.vue
│   │   ├── Search.vue
│   │   └── Admin.vue
│   ├── stores/           # Pinia stores
│   ├── services/         # API client and helpers
│   └── composables/      # Vue composables (useQRScanner, useAuth)
├── public/
└── tests/
    └── unit/             # Component tests
```

**Structure Decision**: Web application structure with `backend/` and `frontend/` directories. The backend serves both the API and the built frontend as static files. Single deployment unit to Heroku or local server.

## Deployment Architecture

**Option A: Backend Serves Frontend (Selected)**

```text
┌─────────────────────────────────────────────┐
│  Local Machine / Heroku / VPS               │
│                                             │
│  Node.js Server (Express)                   │
│  ├── /api/*           → API endpoints       │
│  ├── /*               → Serve Vue frontend  │
│  ├── SQLite database  → ./data/dev.db       │
│  └── File storage     → ./uploads/          │
│                                             │
│  Exposed via:                               │
│  - Development: ngrok HTTPS tunnel          │
│  - Production: Domain with HTTPS            │
└─────────────────────────────────────────────┘
         ▲
         │ HTTPS (Telegram requires secure connection)
         │
    Telegram Mini App
    (Mobile client → WebApp API)
```

**Why Not GitHub Pages?**

GitHub Pages only serves static files and cannot:
- Run Node.js server for API endpoints
- Execute server-side Telegram initData validation
- Access SQLite database
- Handle multi-user shared state

Unlike the [easy-qr-scan-bot](https://github.com/MBoretto/easy-qr-scan-bot) reference project (which uses Telegram Cloud Storage for per-user data), this app requires a backend database for shared equipment tracking across all users.

## Complexity Tracking

> No constitution violations. All design choices align with established principles.

| Aspect | Decision | Rationale |
|--------|----------|-----------|
| Monorepo | frontend + backend in one repo | Simpler deployment, shared types possible |
| File storage | Local filesystem | Constitution mandates local storage, sufficient for scale |
| Auth | Telegram initData only | No separate auth system per constitution |

## Phase 0: Research (Complete)

See [research.md](./research.md) for detailed technology decisions:

- **Telegram Integration**: WebApp API 6.9+ with native QR scanner
- **Frontend**: Vue 3 + Vuetify 3 + Pinia + Vite
- **Backend**: Node.js 20 + Express.js + Prisma + SQLite
- **File Storage**: Local filesystem with SHA-256 checksums
- **Authentication**: Telegram initData validation (HMAC-SHA256)
- **Offline Support**: IndexedDB cache with sync queue

## Phase 1: Design Artifacts (Complete)

### Data Model

See [data-model.md](./data-model.md) for complete Prisma schema:

| Entity | Purpose |
|--------|---------|
| User | Telegram-authenticated users with roles |
| Equipment | Trackable items with UID, status, location |
| EquipmentType | Equipment categories (Oscilloscope, Cable, etc.) |
| Location | Rooms/areas where equipment can be |
| EquipmentAction | Immutable audit trail of all actions |
| Attachment | File metadata with checksums |

**Key Design Patterns**:
- Soft-delete for equipment (`isDeleted` flag)
- Separate home vs. current location
- BigInt for Telegram user IDs
- UUID-based file storage with year/month organization

### API Contracts

See [contracts/openapi.yaml](./contracts/openapi.yaml) for complete OpenAPI 3.0 specification:

| Endpoint Group | Routes |
|----------------|--------|
| Auth | POST /api/auth/validate |
| Equipment | GET/POST /api/equipment, GET/PATCH/DELETE /api/equipment/:id |
| Equipment Actions | POST /api/equipment/:id/checkout, /return |
| History | GET /api/equipment/:id/history |
| Files | GET/POST /api/equipment/:id/files, GET /api/files/:id |
| Locations | GET/POST /api/locations, PATCH /api/locations/:id |
| Types | GET/POST /api/types, PATCH /api/types/:id |
| Users | GET/POST /api/users, GET/PATCH /api/users/:id |

**Security**: All endpoints require `X-Telegram-Init-Data` header validation.

### Developer Setup

See [quickstart.md](./quickstart.md) for:
- Prerequisites and installation steps
- Environment configuration
- Database setup with Prisma
- Telegram bot configuration
- Development server startup
- Deployment instructions (local and Heroku)

## Phase 2: Tasks (Pending)

Run `/speckit.tasks` to generate actionable implementation tasks based on:
- User stories from [spec.md](./spec.md)
- Data model from [data-model.md](./data-model.md)
- API contracts from [contracts/openapi.yaml](./contracts/openapi.yaml)

Tasks will be organized by priority (P1-P6) matching user story priorities.