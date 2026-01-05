# Phase 0 Research: Equipment Manager Mini App

**Branch**: `001-equipment-manager` | **Date**: 2026-01-05 | **Plan**: [plan.md](./plan.md)

## Overview

This document captures technology decisions, rationale, and alternatives considered for the Equipment Manager Mini App implementation.

## Technology Decisions

### 1. Telegram Mini App Integration

**Decision**: Use Telegram WebApp API 6.9+ with native QR scanner

**Rationale**:
- Native `WebApp.showScanQrPopup()` provides reliable QR scanning without custom camera implementations
- Built-in haptic feedback via `WebApp.HapticFeedback.impactOccurred()`
- Telegram authentication via `initData` eliminates need for separate auth system
- Theme integration ensures visual consistency with Telegram client

**Alternatives Considered**:
- **Custom camera with html5-qrcode**: Rejected - less reliable, requires camera permissions handling, inconsistent across devices
- **External QR scanner app**: Rejected - breaks "never leave Telegram" principle

**Implementation Notes**:
```typescript
// QR Scanner integration
WebApp.showScanQrPopup({ text: 'Scan equipment QR code' }, (qrText) => {
  WebApp.HapticFeedback.impactOccurred('medium');
  // Navigate to equipment details
  router.push(`/equipment/${qrText}`);
  return true; // Close scanner
});
```

### 2. Frontend Framework

**Decision**: Vue 3 Composition API + Vuetify 3 + Pinia

**Rationale**:
- Reference project (easy-qr-scan-bot) uses this stack - proven patterns for Telegram Mini Apps
- Vuetify 3 provides Material Design components with built-in accessibility
- Pinia is the Vue 3 recommended state management solution
- Vite provides fast development experience with HMR

**Alternatives Considered**:
- **React + MUI**: Rejected - no reference implementation, different ecosystem
- **Vanilla Vue without Vuetify**: Rejected - would require building UI components from scratch
- **Vuex for state**: Rejected - Pinia is the modern replacement with better TypeScript support

**Key Dependencies**:
```json
{
  "vue": "^3.4.x",
  "vuetify": "^3.5.x",
  "pinia": "^2.1.x",
  "vite": "^5.x",
  "@anthropic-ai/telegram-miniapp": "latest"
}
```

### 3. Backend Runtime & Framework

**Decision**: Node.js 20 LTS + Express.js

**Rationale**:
- Node.js 20 LTS provides stability and long-term support (April 2026)
- Express.js is minimal, well-documented, and widely understood
- JavaScript/TypeScript across the stack reduces context switching
- Heroku and local deployment both support Node.js natively

**Alternatives Considered**:
- **Fastify**: Rejected - additional learning curve, Express is sufficient for this scale
- **NestJS**: Rejected - over-engineered for ~10 endpoints
- **Deno**: Rejected - less deployment options, newer ecosystem

### 4. Database

**Decision**: SQLite 3 with Prisma ORM

**Rationale**:
- SQLite requires zero configuration - single file, no separate server process
- Sufficient for expected scale (1,000 items, 50 concurrent users)
- Easy backup (copy the .db file)
- Prisma provides type-safe queries and automatic migrations
- Local-first development matches deployment target

**Alternatives Considered**:
- **PostgreSQL**: Rejected - requires separate server, overkill for this scale
- **MongoDB**: Rejected - schema flexibility not needed, relational model fits better
- **Better-sqlite3 without ORM**: Rejected - Prisma provides migrations and type safety

**Performance Estimates**:
| Operation | Expected Latency |
|-----------|-----------------|
| Equipment lookup by ID | <10ms |
| Full-text search (1000 items) | <100ms |
| History query (100 records) | <50ms |
| Concurrent writes | SQLite WAL mode supports ~50 concurrent users |

### 5. File Storage

**Decision**: Local filesystem with metadata in SQLite

**Rationale**:
- Simplest approach for single-server deployment
- Files stored in `uploads/` directory with UUID filenames
- Metadata (original name, MIME type, size, uploader, equipment link) in SQLite
- Easy backup alongside database file

**Alternatives Considered**:
- **Telegram Bot API file storage**: Rejected - 20MB limit, files expire after 1 hour for bots
- **S3-compatible storage**: Rejected - external dependency, additional cost
- **Base64 in database**: Rejected - bloats database, poor performance

**File Organization**:
```text
uploads/
├── {year}/
│   └── {month}/
│       └── {uuid}.{ext}
```

### 6. Telegram Bot Integration

**Decision**: node-telegram-bot-api for bot commands, WebApp for Mini App

**Rationale**:
- node-telegram-bot-api is mature and well-maintained
- Bot handles: `/start` command, sending Mini App button, webhook registration
- All user interactions happen in Mini App, not bot commands
- Bot token validates `initData` server-side

**Implementation Pattern**:
```typescript
// Bot sends Mini App button
bot.onText(/\/start/, (msg) => {
  bot.sendMessage(msg.chat.id, 'Open Equipment Manager', {
    reply_markup: {
      inline_keyboard: [[{
        text: 'Open App',
        web_app: { url: process.env.WEBAPP_URL }
      }]]
    }
  });
});
```

### 7. QR Code Generation

**Decision**: `qrcode` npm package for server-side generation

**Rationale**:
- Generate QR codes as PNG/SVG on demand
- Include human-readable equipment ID below QR code
- Support printable label format (A4 sheet with multiple codes)

**Alternatives Considered**:
- **Client-side generation**: Rejected - server-side ensures consistency
- **Third-party QR API**: Rejected - external dependency, potential costs

**Output Formats**:
- PNG for screen display and single-label printing
- SVG for high-resolution printing
- PDF for batch printing (multiple labels per page)

### 8. Authentication & Authorization

**Decision**: Telegram initData validation only

**Rationale**:
- Constitution mandates Telegram-only authentication
- `initData` contains user ID, username, and signature
- Server validates signature using bot token (HMAC-SHA256)
- User roles stored in database, keyed by Telegram user ID

**Security Flow**:
1. Mini App sends `initData` with every API request (header)
2. Server validates HMAC signature against bot token
3. Server extracts user ID, looks up role in database
4. Role determines access (User vs Admin)

**Implementation**:
```typescript
// Middleware for initData validation
function validateInitData(initData: string, botToken: string): TelegramUser | null {
  // Parse initData
  // Validate hash using HMAC-SHA256
  // Return user data if valid
}
```

### 9. Offline Support

**Decision**: IndexedDB cache with sync queue

**Rationale**:
- Connectivity can be unreliable in lab/warehouse environments
- Critical actions (checkout, return) queue locally when offline
- Read operations serve from cache first, then refresh
- Sync queue processes when connectivity returns

**Implementation Pattern**:
- Pinia store persisted to IndexedDB via `pinia-plugin-persistedstate`
- Sync queue stored separately, processed in order
- Conflict resolution: server wins, user notified

**Cached Data**:
- Equipment list (refreshed on app open)
- Location/type lookup tables
- Pending actions queue

### 10. API Design

**Decision**: RESTful JSON API with Zod validation

**Rationale**:
- REST is simple and well-understood
- Zod provides runtime type validation matching TypeScript types
- OpenAPI spec generated from Zod schemas for documentation

**Endpoint Structure**:
```text
POST   /api/auth/validate          - Validate initData, return user profile
GET    /api/equipment              - List equipment (with search/filter)
GET    /api/equipment/:id          - Get equipment details
POST   /api/equipment              - Create equipment
PATCH  /api/equipment/:id          - Update equipment
DELETE /api/equipment/:id          - Soft-delete equipment (admin only)
GET    /api/equipment/:id/history  - Get equipment action history
POST   /api/equipment/:id/checkout - Check out equipment
POST   /api/equipment/:id/return   - Return equipment
POST   /api/equipment/:id/files    - Upload file attachment
GET    /api/equipment/:id/files    - List file attachments
GET    /api/files/:id              - Download file
GET    /api/locations              - List locations
POST   /api/locations              - Create location (admin)
GET    /api/types                  - List equipment types
POST   /api/types                  - Create equipment type (admin)
GET    /api/users                  - List users (admin)
POST   /api/users                  - Create user (admin)
PATCH  /api/users/:id              - Update user role (admin)
```

## Open Questions Resolved

| Question | Resolution |
|----------|------------|
| How to handle shared data across users? | SQLite backend, not Telegram Cloud Storage |
| File size limits? | 10MB per file (constitution requirement) |
| Equipment deletion? | Admin-only soft-delete (spec clarification) |
| Equipment editing? | Any user can edit, changes tracked (spec clarification) |

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| SQLite concurrent write contention | WAL mode, optimistic locking |
| File storage disk space | Monitor usage, configurable limits |
| Telegram API rate limits | Cache aggressively, batch operations |
| initData expiration | Re-validate on session start, handle gracefully |

## Deployment Strategy

### Selected: Backend Serves Frontend (Unified Deployment)

**Architecture**:
```javascript
// backend/src/server.js
const express = require('express');
const path = require('path');
const app = express();

// API routes
app.use('/api', apiRouter);

// Serve static frontend build
app.use(express.static(path.join(__dirname, '../../frontend/dist')));

// SPA fallback - all non-API routes serve index.html
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../../frontend/dist/index.html'));
});
```

**Build Process**:
```bash
# Build frontend
cd frontend && npm run build  # Output: frontend/dist/

# Start unified server
cd backend && npm start        # Serves both API and frontend
```

**Why This Approach**:
- ✅ Single deployment target (one server, one port)
- ✅ No CORS configuration needed
- ✅ Simplified Telegram initData validation (same origin)
- ✅ Works on local machine, Heroku, or any Node.js host
- ✅ Easy backup (copy entire directory with SQLite + uploads)

**Rejected Alternative: Separate Frontend (GitHub Pages)**:
- ❌ GitHub Pages cannot run Node.js backend
- ❌ Requires CORS setup for API calls
- ❌ Splits deployment into two services
- ❌ easy-qr-scan-bot uses this because it has **no backend** (uses Telegram Cloud Storage)

### Development vs. Production

| Environment | Frontend | Backend | Database |
|-------------|----------|---------|----------|
| **Development** | Vite dev server (localhost:5173) | Express (localhost:3000) | SQLite (./dev.db) |
| **Production** | Express serves `dist/` | Express (port 3000) | SQLite (./data/prod.db) |

**HTTPS Requirement**: Telegram Mini Apps require HTTPS. Use:
- Development: ngrok tunnel (`ngrok http 3000`)
- Production: Domain with SSL or Heroku (auto-HTTPS)

## Next Steps

1. **Phase 1**: ✅ Created detailed data model (Prisma schema)
2. **Phase 1**: ✅ Defined API contracts (OpenAPI spec)
3. **Phase 1**: ✅ Written quickstart.md for development setup
4. **Phase 2**: Generate implementation tasks with `/speckit.tasks`