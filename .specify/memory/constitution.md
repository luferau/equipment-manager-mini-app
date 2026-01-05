<!--
  SYNC IMPACT REPORT
  ==================
  Version change: 1.0.0 → 1.1.0 (architecture update)

  Modified principles:
    - I. Telegram Mini App Native: Removed Cloud Storage as primary persistence
    - V. Offline-Resilient Design: Updated to use IndexedDB for local cache

  Added sections:
    - VIII. Node.js Backend & SQLite (new principle)

  Removed sections: None

  Technology Stack changes:
    - Added: Node.js runtime, Express.js, SQLite, node-telegram-bot-api, Prisma ORM
    - Changed: Primary Storage from "Telegram Cloud Storage" to "SQLite"
    - Changed: Deployment from "GitHub Pages" to "Local/Heroku"

  Templates requiring updates:
    ✅ .specify/templates/plan-template.md - No update needed (generic)
    ✅ .specify/templates/spec-template.md - No update needed (generic)
    ✅ .specify/templates/tasks-template.md - No update needed (generic)

  Follow-up TODOs: None
-->

# Equipment Manager Mini App Constitution

## Core Principles

### I. Telegram Mini App Native

All user interactions MUST occur within the Telegram Mini App context. The application:
- MUST use Telegram WebApp API (version 6.9+) for all platform features
- MUST integrate with Telegram's native QR scanner (no custom camera implementations)
- MUST utilize Telegram haptic feedback for scan confirmations
- MUST authenticate users via Telegram's built-in user identity (no separate auth system)
- MUST NOT require users to leave Telegram to complete any core workflow
- MAY use Telegram Cloud Storage for user preferences only (not shared data)

**Rationale**: Telegram Mini Apps provide built-in authentication and device access.
Shared data (equipment, history, files) requires a backend database accessible to all users.

### II. Vue + Vuetify First

The frontend architecture MUST follow Vue.js and Vuetify conventions:
- MUST use Vue 3 Composition API for all new components
- MUST use Vuetify 3 components for UI consistency and accessibility
- MUST structure components as single-file components (.vue)
- MUST use Pinia for state management when local component state is insufficient
- MUST follow Vuetify's Material Design guidelines for visual consistency
- MUST NOT introduce alternative UI frameworks or conflicting styling systems

**Rationale**: Consistency with the reference implementation (easy-qr-scan-bot) ensures
proven patterns for Telegram Mini App development and reduces integration risk.

### III. QR-Centric Workflow

QR codes are the primary equipment identification mechanism:
- Each equipment item MUST have a unique, system-generated identifier
- Identifiers MUST be encodable as both QR codes and human-readable alphanumeric strings
- QR scanning MUST use Telegram's native scanner (WebApp.showScanQrPopup)
- Scanning MUST trigger haptic feedback on success
- The system MUST support continuous scanning mode for bulk operations
- QR codes MUST be printable and include human-readable ID below the code
- Invalid or unknown QR codes MUST display clear error messages with recovery guidance

**Rationale**: QR codes enable instant equipment lookup without manual data entry.
Native Telegram scanning is more reliable than JS-based camera solutions.

### IV. Equipment Traceability

Every equipment state change MUST be tracked:
- Equipment location changes MUST record: who, when, from-location, to-location
- All equipment actions MUST be immutable once recorded (append-only history)
- Equipment MUST have defined states: Checked-Out, Available, Maintenance, Broken
- File attachments (documents, photos, videos) MUST be linked to equipment with timestamps
- Equipment types and categories MUST be hierarchical (type → unit)
- Room/location data MUST be normalized (room numbers as first-class entities)

**Rationale**: Equipment management requires complete audit trails for accountability.
Traceability answers "where is it?" and "who had it?" at any point in time.

### V. Mobile-First UX

The interface MUST prioritize mobile Telegram clients:
- Touch targets MUST be minimum 44x44px for accessibility
- Primary actions (scan, check-out, search) MUST be reachable with one thumb
- Forms MUST minimize typing; prefer selection, scanning, and defaults
- Loading states MUST show skeleton screens, not blank pages
- Error messages MUST be actionable ("Scan failed. Tap to retry.")
- The app MUST function on Telegram API version 6.9+ (Android and iOS)

**Rationale**: Equipment scanning happens on mobile devices in the field.
Desktop support is secondary to mobile usability.

### VI. Data Integrity & Audit Trail

All data operations MUST maintain integrity and auditability:
- Equipment IDs MUST be immutable once generated
- Deletion MUST be soft-delete with retention (never hard-delete equipment records)
- All user actions MUST include: user_id, timestamp, action_type, before/after state
- File attachments MUST be stored with checksums for integrity verification
- Data exports MUST include complete history, not just current state
- The system MUST support administrative queries for compliance audits

**Rationale**: Equipment tracking systems are often subject to audits.
Complete, tamper-evident history is essential for compliance and dispute resolution.

### VII. Node.js Backend & SQLite

All shared data MUST be stored in a centralized backend:
- MUST use Node.js as the server runtime
- MUST use SQLite as the database (single-file, zero-configuration)
- MUST use Prisma ORM for type-safe database access and migrations
- MUST use Express.js for REST API endpoints
- MUST use node-telegram-bot-api for Telegram Bot API integration
- Files MUST be stored on the local filesystem with metadata in SQLite
- The backend MUST be deployable to local machine or Heroku
- All API endpoints MUST validate Telegram initData before processing

**Rationale**: SQLite is the simplest database solution - no separate server process,
single file for all data, easy backup (copy the file), and sufficient for typical
equipment inventory scale (thousands of items, dozens of users).

## Technology Stack

The following technology choices are non-negotiable for this project:

### Frontend (Mini App)

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Framework | Vue 3 | Reference project compatibility, reactive UI |
| UI Components | Vuetify 3 | Material Design, accessibility, mobile-first |
| State Management | Pinia | Vue 3 recommended, devtools support |
| Build Tool | Vite | Fast HMR, modern ES modules |
| Platform API | Telegram WebApp SDK | Native Mini App integration |
| Code Quality | ESLint + Prettier | Consistent code style |

### Backend (Bot + API)

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Runtime | Node.js 20 LTS | Stable, long-term support |
| Framework | Express.js | Minimal, well-documented |
| Database | SQLite 3 | Zero-config, single-file, portable |
| ORM | Prisma | Type-safe queries, easy migrations |
| Bot Library | node-telegram-bot-api | Mature, well-maintained |
| File Storage | Local filesystem | Simple, no external dependencies |
| Validation | Zod | Runtime type validation |

### Deployment

| Environment | Platform | Notes |
|-------------|----------|-------|
| Development | Local machine | SQLite file in project directory |
| Production | Heroku or local server | Heroku free tier or any Node.js host |
| Frontend | Served by backend | Single deployment unit |

## Security & Compliance

Security requirements for equipment tracking systems:

- User identity MUST come exclusively from Telegram (initData validation)
- All API requests MUST validate Telegram initData signature server-side
- File uploads MUST validate MIME types before storage
- File uploads MUST be size-limited (configurable, default 10MB)
- PII (personally identifiable information) MUST be minimized; use Telegram user IDs
- Equipment location data MUST NOT expose precise GPS coordinates (room numbers only)
- Admin actions MUST require elevated Telegram user permissions (bot admin list)
- Audit logs MUST be retained for minimum 2 years

## Governance

This Constitution is the authoritative source for architectural and development decisions.

**Amendment Process**:
1. Propose changes via documented rationale in a pull request
2. Changes require approval from project maintainer(s)
3. Breaking changes (principle removal/redefinition) require migration plan
4. Version increment follows semantic versioning (see below)

**Versioning Policy**:
- MAJOR: Backward-incompatible principle changes or removals
- MINOR: New principles, sections, or materially expanded guidance
- PATCH: Clarifications, wording improvements, typo fixes

**Compliance Review**:
- All pull requests MUST verify alignment with these principles
- Code reviews MUST check Constitution compliance before approval
- Complexity beyond these principles MUST be justified in PR description

**Runtime Guidance**:
For day-to-day development patterns, refer to project documentation and
the Telegram Mini Apps official documentation at https://core.telegram.org/bots/webapps

**Version**: 1.1.0 | **Ratified**: 2026-01-05 | **Last Amended**: 2026-01-05
