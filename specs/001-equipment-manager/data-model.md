# Data Model: Equipment Manager Mini App

**Branch**: `001-equipment-manager` | **Date**: 2026-01-05 | **Plan**: [plan.md](./plan.md)

## Overview

This document defines the database schema using Prisma ORM for SQLite. The model supports all functional requirements from [spec.md](./spec.md).

## Entity Relationship Diagram

```text
┌─────────────────┐       ┌─────────────────┐
│      User       │       │   EquipmentType │
├─────────────────┤       ├─────────────────┤
│ id (PK)         │       │ id (PK)         │
│ telegramId (UK) │       │ name            │
│ username        │       │ description     │
│ firstName       │       │ isActive        │
│ lastName        │       │ createdAt       │
│ role            │       │ updatedAt       │
│ isActive        │       └────────┬────────┘
│ createdAt       │                │
│ updatedAt       │                │ 1:N
└────────┬────────┘                │
         │                ┌────────▼────────┐
         │                │    Equipment    │
         │ 1:N            ├─────────────────┤
         │ (holder)       │ id (PK)         │
         │                │ uid (UK)        │◄──────── QR Code encodes this
         │                │ name            │
         └───────────────►│ description     │
                          │ typeId (FK)     │───────► EquipmentType
         ┌───────────────►│ homeLocationId  │───────► Location
         │                │ currentLocId    │───────► Location
         │                │ currentHolderId │───────► User
         │ 1:N            │ status          │
         │ (location)     │ isDeleted       │
         │                │ createdAt       │
┌────────┴────────┐       │ updatedAt       │
│    Location     │       │ createdById     │───────► User
├─────────────────┤       └────────┬────────┘
│ id (PK)         │                │
│ name            │                │ 1:N
│ description     │                │
│ isActive        │       ┌────────▼────────┐
│ createdAt       │       │ EquipmentAction │
│ updatedAt       │       ├─────────────────┤
└─────────────────┘       │ id (PK)         │
                          │ equipmentId (FK)│───────► Equipment
                          │ userId (FK)     │───────► User
                          │ actionType      │
                          │ fromLocationId  │───────► Location (nullable)
                          │ toLocationId    │───────► Location (nullable)
                          │ fromStatus      │
                          │ toStatus        │
                          │ notes           │
                          │ createdAt       │
                          └─────────────────┘

┌─────────────────┐
│   Attachment    │
├─────────────────┤
│ id (PK)         │
│ equipmentId (FK)│───────► Equipment
│ uploadedById(FK)│───────► User
│ filename        │
│ originalName    │
│ mimeType        │
│ size            │
│ checksum        │
│ createdAt       │
└─────────────────┘
```

## Prisma Schema

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

// ============================================
// User Management
// ============================================

enum UserRole {
  USER
  ADMIN
}

model User {
  id         Int      @id @default(autoincrement())
  telegramId BigInt   @unique
  username   String?  // Telegram username (optional)
  firstName  String
  lastName   String?
  role       UserRole @default(USER)
  isActive   Boolean  @default(true)
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  // Relations
  heldEquipment    Equipment[]       @relation("CurrentHolder")
  createdEquipment Equipment[]       @relation("CreatedBy")
  actions          EquipmentAction[]
  attachments      Attachment[]

  @@index([telegramId])
  @@index([isActive])
}

// ============================================
// Master Data
// ============================================

model EquipmentType {
  id          Int      @id @default(autoincrement())
  name        String   @unique
  description String?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  equipment Equipment[]

  @@index([isActive])
}

model Location {
  id          Int      @id @default(autoincrement())
  name        String   @unique // Room number or area name
  description String?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  homeEquipment    Equipment[]       @relation("HomeLocation")
  currentEquipment Equipment[]       @relation("CurrentLocation")
  actionsFrom      EquipmentAction[] @relation("FromLocation")
  actionsTo        EquipmentAction[] @relation("ToLocation")

  @@index([isActive])
}

// ============================================
// Equipment Core
// ============================================

enum EquipmentStatus {
  AVAILABLE
  CHECKED_OUT
  MAINTENANCE
  BROKEN
}

model Equipment {
  id              Int             @id @default(autoincrement())
  uid             String          @unique // Human-readable ID, e.g., "EQ-001-ABC"
  name            String
  description     String?
  status          EquipmentStatus @default(AVAILABLE)
  isDeleted       Boolean         @default(false) // Soft delete
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  // Foreign Keys
  typeId          Int
  homeLocationId  Int?
  currentLocationId Int?
  currentHolderId Int?
  createdById     Int

  // Relations
  type            EquipmentType    @relation(fields: [typeId], references: [id])
  homeLocation    Location?        @relation("HomeLocation", fields: [homeLocationId], references: [id])
  currentLocation Location?        @relation("CurrentLocation", fields: [currentLocationId], references: [id])
  currentHolder   User?            @relation("CurrentHolder", fields: [currentHolderId], references: [id])
  createdBy       User             @relation("CreatedBy", fields: [createdById], references: [id])
  actions         EquipmentAction[]
  attachments     Attachment[]

  @@index([uid])
  @@index([status])
  @@index([isDeleted])
  @@index([typeId])
  @@index([currentLocationId])
  @@index([name]) // For search
}

// ============================================
// Audit Trail
// ============================================

enum ActionType {
  CREATED
  UPDATED
  CHECKED_OUT
  RETURNED
  STATUS_CHANGED
  LOCATION_CHANGED
  DELETED
  FILE_ATTACHED
}

model EquipmentAction {
  id             Int             @id @default(autoincrement())
  actionType     ActionType
  notes          String?         // Optional notes/reason for action
  createdAt      DateTime        @default(now())

  // Before/After State
  fromStatus     EquipmentStatus?
  toStatus       EquipmentStatus?
  fromLocationId Int?
  toLocationId   Int?

  // JSON field for storing changed field values on UPDATE
  changes        String?         // JSON: { "field": { "from": "old", "to": "new" } }

  // Foreign Keys
  equipmentId    Int
  userId         Int

  // Relations
  equipment      Equipment       @relation(fields: [equipmentId], references: [id])
  user           User            @relation(fields: [userId], references: [id])
  fromLocation   Location?       @relation("FromLocation", fields: [fromLocationId], references: [id])
  toLocation     Location?       @relation("ToLocation", fields: [toLocationId], references: [id])

  @@index([equipmentId])
  @@index([userId])
  @@index([createdAt])
  @@index([actionType])
}

// ============================================
// File Attachments
// ============================================

model Attachment {
  id           Int      @id @default(autoincrement())
  filename     String   // UUID-based filename on disk
  originalName String   // Original uploaded filename
  mimeType     String
  size         Int      // File size in bytes
  checksum     String   // SHA-256 hash for integrity
  createdAt    DateTime @default(now())

  // Foreign Keys
  equipmentId  Int
  uploadedById Int

  // Relations
  equipment    Equipment @relation(fields: [equipmentId], references: [id])
  uploadedBy   User      @relation(fields: [uploadedById], references: [id])

  @@index([equipmentId])
  @@index([uploadedById])
}
```

## Equipment UID Generation

Equipment identifiers follow the pattern: `EQ-{NNNN}-{XXX}`
- `EQ` - Fixed prefix
- `NNNN` - Zero-padded sequential number (0001-9999)
- `XXX` - Random alphanumeric suffix for uniqueness

**Generation Logic**:
```typescript
function generateEquipmentUid(sequence: number): string {
  const suffix = Math.random().toString(36).substring(2, 5).toUpperCase();
  const padded = sequence.toString().padStart(4, '0');
  return `EQ-${padded}-${suffix}`;
}
```

**Examples**: `EQ-0001-A7K`, `EQ-0042-B3M`, `EQ-1000-XYZ`

## Key Design Decisions

### 1. Soft Delete Pattern

Equipment records use `isDeleted` boolean instead of physical deletion:
- Preserves complete audit trail
- Historical actions remain queryable
- Recovery is possible if needed
- All list queries filter by `isDeleted = false`

### 2. Separate Home vs Current Location

- `homeLocationId`: Where equipment belongs when not in use
- `currentLocationId`: Where equipment currently is
- When returned, `currentLocationId` is set to `homeLocationId`
- Equipment without `homeLocationId` displays "No home location set"

### 3. Immutable Action History

`EquipmentAction` records are append-only:
- Never updated or deleted
- Complete trail of who did what, when
- `changes` JSON field captures field-level diffs for UPDATE actions

### 4. File Storage Strategy

Attachments metadata in SQLite, actual files on filesystem:
- `filename`: UUID-based path (e.g., `2026/01/550e8400-e29b-41d4-a716-446655440000.pdf`)
- `checksum`: SHA-256 hash for integrity verification
- Files organized by year/month to avoid large directories

### 5. BigInt for Telegram IDs

Telegram user IDs can exceed JavaScript's safe integer limit (2^53), so:
- `telegramId` is `BigInt` in Prisma
- Serialized as string in JSON responses
- Frontend handles as string

## Indexes Rationale

| Index | Purpose |
|-------|---------|
| `User.telegramId` | Fast user lookup during auth validation |
| `Equipment.uid` | QR code lookup (primary user flow) |
| `Equipment.name` | Full-text search on equipment name |
| `Equipment.status` | Filter by availability |
| `Equipment.isDeleted` | Filter active equipment |
| `EquipmentAction.equipmentId` | Load history for equipment detail view |
| `EquipmentAction.createdAt` | Sort history chronologically |

## Migration Strategy

Prisma handles migrations automatically:
```bash
# Generate migration from schema changes
npx prisma migrate dev --name init

# Apply migrations in production
npx prisma migrate deploy

# Generate TypeScript client
npx prisma generate
```

## Seed Data

Initial seed for development/testing:
```typescript
// prisma/seed.ts
const types = [
  { name: 'Oscilloscope', description: 'Signal measurement device' },
  { name: 'Network Analyzer', description: 'Network parameter measurement' },
  { name: 'Wattmeter', description: 'Power measurement device' },
  { name: 'Cable', description: 'Connection cables and adapters' },
  { name: 'Other', description: 'Miscellaneous equipment' },
];

const locations = [
  { name: 'Lab 101', description: 'Main laboratory' },
  { name: 'Lab 102', description: 'Secondary laboratory' },
  { name: 'Storage A', description: 'Primary storage room' },
  { name: 'Storage B', description: 'Secondary storage room' },
];
```

## Validation Rules

| Field | Validation |
|-------|------------|
| `Equipment.name` | Required, 1-200 characters |
| `Equipment.description` | Optional, max 2000 characters |
| `Location.name` | Required, unique, 1-100 characters |
| `EquipmentType.name` | Required, unique, 1-100 characters |
| `Attachment.size` | Max 10MB (10,485,760 bytes) |
| `Attachment.mimeType` | Allowlist: image/*, video/*, application/pdf, etc. |
