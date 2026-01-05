# Tasks: Equipment Manager Mini App

**Input**: Design documents from `/specs/001-equipment-manager/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/openapi.yaml, quickstart.md

**Tests**: Not explicitly requested - test tasks are NOT included. Add tests if needed during implementation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Web app structure**: `backend/` and `frontend/` at repository root
- Backend: `backend/src/`, `backend/prisma/`, `backend/tests/`
- Frontend: `frontend/src/`, `frontend/tests/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per plan.md (backend/, frontend/ directories)
- [ ] T002 [P] Initialize backend Node.js project with TypeScript in backend/package.json
- [ ] T003 [P] Initialize frontend Vue 3 + Vite project with TypeScript in frontend/package.json
- [ ] T004 [P] Install backend dependencies: express, prisma, node-telegram-bot-api, zod, cors, multer, qrcode in backend/
- [ ] T005 [P] Install frontend dependencies: vue, vuetify, pinia, vue-router, axios in frontend/
- [ ] T006 [P] Configure ESLint + Prettier for backend in backend/.eslintrc.js
- [ ] T007 [P] Configure ESLint + Prettier for frontend in frontend/.eslintrc.js
- [ ] T008 Create backend environment configuration in backend/.env.example
- [ ] T009 Create frontend environment configuration in frontend/.env.example

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

### Database Schema & ORM

- [ ] T010 Create Prisma schema with all models in backend/prisma/schema.prisma (User, Equipment, EquipmentType, Location, EquipmentAction, Attachment)
- [ ] T011 Generate initial migration and Prisma client (npx prisma migrate dev --name init)
- [ ] T012 Create database seed script with sample types/locations in backend/prisma/seed.ts

### Authentication & Authorization

- [ ] T013 Implement Telegram initData validation utility in backend/src/utils/telegram-auth.ts
- [ ] T014 Create auth middleware for API routes in backend/src/api/middleware/auth.middleware.ts
- [ ] T015 Implement role-based access control helper in backend/src/utils/rbac.ts

### API Infrastructure

- [ ] T016 Setup Express server with CORS, JSON parsing in backend/src/server.ts
- [ ] T017 Create API router structure in backend/src/api/routes/index.ts
- [ ] T018 Implement error handling middleware in backend/src/api/middleware/error.middleware.ts
- [ ] T019 Create Zod validation schemas for common types in backend/src/schemas/common.ts
- [ ] T020 Setup static file serving for frontend build in backend/src/server.ts

### Telegram Bot

- [ ] T021 Implement Telegram bot with /start command in backend/src/bot/bot.ts
- [ ] T022 Configure bot webhook/polling and Mini App button in backend/src/bot/commands.ts

### Frontend Infrastructure

- [ ] T023 Configure Vue Router with route structure in frontend/src/router/index.ts
- [ ] T024 Setup Vuetify 3 with theme configuration in frontend/src/plugins/vuetify.ts
- [ ] T025 Create Pinia store structure in frontend/src/stores/index.ts
- [ ] T026 Create API client with Telegram initData header in frontend/src/services/api.ts
- [ ] T027 Create auth composable using Telegram WebApp in frontend/src/composables/useAuth.ts
- [ ] T028 Create app shell layout with navigation in frontend/src/components/layout/AppShell.vue

**Checkpoint**: Foundation ready - user story implementation can now begin

---

## Phase 3: User Story 1 - Scan, View, and Update Equipment (Priority: P1)

**Goal**: Users can scan QR codes to view equipment details, see history, view/add file attachments

**Independent Test**: Print a test QR code, scan in Telegram, verify equipment details display, attach a file

### Backend Implementation for US1

- [ ] T029 [P] [US1] Create Equipment service with getById, getByUid in backend/src/services/equipment.service.ts
- [ ] T030 [P] [US1] Create EquipmentAction service with getByEquipmentId in backend/src/services/action.service.ts
- [ ] T031 [P] [US1] Create Attachment service with getByEquipmentId, create in backend/src/services/attachment.service.ts
- [ ] T032 [US1] Create file upload utility with checksum generation in backend/src/utils/file-upload.ts
- [ ] T033 [US1] Implement GET /api/equipment/:id endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T034 [US1] Implement GET /api/equipment/:id/history endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T035 [US1] Implement GET /api/equipment/:id/files endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T036 [US1] Implement POST /api/equipment/:id/files (multipart) in backend/src/api/routes/equipment.routes.ts
- [ ] T037 [US1] Implement GET /api/files/:id for file download in backend/src/api/routes/files.routes.ts
- [ ] T038 [US1] Create Zod schemas for equipment endpoints in backend/src/schemas/equipment.schema.ts

### Frontend Implementation for US1

- [ ] T039 [US1] Create QR scanner composable using WebApp.showScanQrPopup in frontend/src/composables/useQRScanner.ts
- [ ] T040 [US1] Create equipment store with current equipment state in frontend/src/stores/equipment.store.ts
- [ ] T041 [US1] Create Scan page with scanner trigger in frontend/src/pages/Scan.vue
- [ ] T042 [US1] Create Equipment Details page with tabs (Info, History, Documents) in frontend/src/pages/EquipmentDetails.vue
- [ ] T043 [P] [US1] Create EquipmentInfo component in frontend/src/components/equipment/EquipmentInfo.vue
- [ ] T044 [P] [US1] Create EquipmentHistory component in frontend/src/components/equipment/EquipmentHistory.vue
- [ ] T045 [P] [US1] Create EquipmentDocuments component in frontend/src/components/equipment/EquipmentDocuments.vue
- [ ] T046 [US1] Create FileUpload component with camera capture in frontend/src/components/common/FileUpload.vue
- [ ] T047 [US1] Create EquipmentNotFound error component in frontend/src/components/equipment/EquipmentNotFound.vue
- [ ] T048 [US1] Add haptic feedback on successful scan in frontend/src/composables/useQRScanner.ts

**Checkpoint**: User Story 1 complete - QR scanning, viewing, and file attachment works independently

---

## Phase 4: User Story 2 - Check Out Equipment (Priority: P2)

**Goal**: Users can check out equipment to a destination location and return it to home location

**Independent Test**: View equipment, tap "Take Equipment", select destination, confirm checkout. Then tap "Return".

### Backend Implementation for US2

- [ ] T049 [US2] Extend Equipment service with checkout, return methods in backend/src/services/equipment.service.ts
- [ ] T050 [US2] Implement POST /api/equipment/:id/checkout endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T051 [US2] Implement POST /api/equipment/:id/return endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T052 [US2] Create Zod schemas for checkout/return in backend/src/schemas/equipment.schema.ts
- [ ] T053 [US2] Implement optimistic locking for concurrent checkout in backend/src/services/equipment.service.ts

### Frontend Implementation for US2

- [ ] T054 [P] [US2] Create locations store with location list in frontend/src/stores/locations.store.ts
- [ ] T055 [US2] Implement GET /api/locations call in frontend/src/services/api.ts
- [ ] T056 [US2] Create CheckoutDialog component with location selector in frontend/src/components/equipment/CheckoutDialog.vue
- [ ] T057 [US2] Create ReturnDialog component with confirmation in frontend/src/components/equipment/ReturnDialog.vue
- [ ] T058 [US2] Add checkout/return actions to EquipmentDetails page in frontend/src/pages/EquipmentDetails.vue
- [ ] T059 [US2] Show current holder info when equipment is checked out in frontend/src/components/equipment/EquipmentInfo.vue

**Checkpoint**: User Story 2 complete - checkout and return works independently

---

## Phase 5: User Story 3 - Search and Browse Equipment (Priority: P3)

**Goal**: Users can search equipment by name/type/location and browse by room or category

**Independent Test**: Enter search term, verify filtered results. Browse by room, verify equipment counts.

### Backend Implementation for US3

- [ ] T060 [US3] Extend Equipment service with search, filter, pagination in backend/src/services/equipment.service.ts
- [ ] T061 [US3] Implement GET /api/equipment with search/filter params in backend/src/api/routes/equipment.routes.ts
- [ ] T062 [US3] Implement GET /api/locations with equipment counts in backend/src/api/routes/locations.routes.ts
- [ ] T063 [US3] Implement GET /api/types with equipment counts in backend/src/api/routes/types.routes.ts

### Frontend Implementation for US3

- [ ] T064 [P] [US3] Create types store with equipment type list in frontend/src/stores/types.store.ts
- [ ] T065 [US3] Create Home page with search and browse options in frontend/src/pages/Home.vue
- [ ] T066 [US3] Create Search page with results list in frontend/src/pages/Search.vue
- [ ] T067 [US3] Create EquipmentList component with summary cards in frontend/src/components/equipment/EquipmentList.vue
- [ ] T068 [US3] Create BrowseByRoom page with room list in frontend/src/pages/BrowseByRoom.vue
- [ ] T069 [US3] Create BrowseByType page with type list in frontend/src/pages/BrowseByType.vue
- [ ] T070 [US3] Create EmptyState component for no results in frontend/src/components/common/EmptyState.vue

**Checkpoint**: User Story 3 complete - search and browse works independently

---

## Phase 6: User Story 4 - Add New Equipment (Priority: P4)

**Goal**: Any user can add new equipment with type, name, location, and the system generates QR code

**Independent Test**: Tap "Add Equipment", fill form, submit. Verify equipment appears in search with QR code.

### Backend Implementation for US4

- [ ] T071 [US4] Create UID generation utility in backend/src/utils/uid-generator.ts
- [ ] T072 [US4] Create QR code generation utility in backend/src/utils/qr-generator.ts
- [ ] T073 [US4] Extend Equipment service with create method in backend/src/services/equipment.service.ts
- [ ] T074 [US4] Implement POST /api/equipment endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T075 [US4] Implement GET /api/equipment/:id/qrcode endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T076 [US4] Create Zod schema for create equipment in backend/src/schemas/equipment.schema.ts

### Frontend Implementation for US4

- [ ] T077 [US4] Create AddEquipment page with form in frontend/src/pages/AddEquipment.vue
- [ ] T078 [US4] Create EquipmentForm component with validation in frontend/src/components/equipment/EquipmentForm.vue
- [ ] T079 [US4] Create TypeSelector component (dropdown) in frontend/src/components/common/TypeSelector.vue
- [ ] T080 [US4] Create LocationSelector component (dropdown) in frontend/src/components/common/LocationSelector.vue
- [ ] T081 [US4] Create QRCodeDisplay component with save/print in frontend/src/components/equipment/QRCodeDisplay.vue
- [ ] T082 [US4] Add "Add Equipment" button to Home page in frontend/src/pages/Home.vue

**Checkpoint**: User Story 4 complete - equipment creation works independently

---

## Phase 7: User Story 5 - Manage Equipment Types and Locations (Priority: P5)

**Goal**: Admins can manage master data: equipment types and locations (add, edit, deactivate)

**Independent Test**: Access admin settings, add new type, verify it appears in Add Equipment form.

### Backend Implementation for US5

- [ ] T083 [P] [US5] Create EquipmentType service with CRUD in backend/src/services/type.service.ts
- [ ] T084 [P] [US5] Create Location service with CRUD in backend/src/services/location.service.ts
- [ ] T085 [US5] Implement POST /api/types (admin) in backend/src/api/routes/types.routes.ts
- [ ] T086 [US5] Implement PATCH /api/types/:id (admin) in backend/src/api/routes/types.routes.ts
- [ ] T087 [US5] Implement POST /api/locations (admin) in backend/src/api/routes/locations.routes.ts
- [ ] T088 [US5] Implement PATCH /api/locations/:id (admin) in backend/src/api/routes/locations.routes.ts
- [ ] T089 [US5] Create Zod schemas for types/locations in backend/src/schemas/master-data.schema.ts

### Frontend Implementation for US5

- [ ] T090 [US5] Create Admin page with settings menu in frontend/src/pages/Admin.vue
- [ ] T091 [US5] Create ManageTypes page with list and forms in frontend/src/pages/admin/ManageTypes.vue
- [ ] T092 [US5] Create ManageLocations page with list and forms in frontend/src/pages/admin/ManageLocations.vue
- [ ] T093 [P] [US5] Create TypeForm component in frontend/src/components/admin/TypeForm.vue
- [ ] T094 [P] [US5] Create LocationForm component in frontend/src/components/admin/LocationForm.vue
- [ ] T095 [US5] Add admin route guard for role check in frontend/src/router/guards.ts

**Checkpoint**: User Story 5 complete - type/location management works independently

---

## Phase 8: User Story 6 - User Management (Priority: P6)

**Goal**: Admins can add, edit, and deactivate users. New users see pending approval message.

**Independent Test**: Access user management, add user by Telegram ID, verify they can access app.

### Backend Implementation for US6

- [ ] T096 [US6] Create User service with CRUD in backend/src/services/user.service.ts
- [ ] T097 [US6] Implement GET /api/users (admin) in backend/src/api/routes/users.routes.ts
- [ ] T098 [US6] Implement POST /api/users (admin) in backend/src/api/routes/users.routes.ts
- [ ] T099 [US6] Implement GET /api/users/:id (admin) in backend/src/api/routes/users.routes.ts
- [ ] T100 [US6] Implement PATCH /api/users/:id (admin) in backend/src/api/routes/users.routes.ts
- [ ] T101 [US6] Create Zod schemas for user management in backend/src/schemas/user.schema.ts
- [ ] T102 [US6] Implement POST /api/auth/validate with user lookup/creation in backend/src/api/routes/auth.routes.ts

### Frontend Implementation for US6

- [ ] T103 [US6] Create user store with current user state in frontend/src/stores/user.store.ts
- [ ] T104 [US6] Create ManageUsers page with list in frontend/src/pages/admin/ManageUsers.vue
- [ ] T105 [US6] Create UserForm component for add/edit in frontend/src/components/admin/UserForm.vue
- [ ] T106 [US6] Create UserActivityHistory component in frontend/src/components/admin/UserActivityHistory.vue
- [ ] T107 [US6] Create AccessDenied page for unauthorized users in frontend/src/pages/AccessDenied.vue
- [ ] T108 [US6] Create PendingApproval page for new users in frontend/src/pages/PendingApproval.vue
- [ ] T109 [US6] Implement auth flow on app load in frontend/src/App.vue

**Checkpoint**: User Story 6 complete - user management works independently

---

## Phase 9: Equipment Edit & Delete (Cross-cutting for US1-US4)

**Goal**: Any user can edit equipment details, admins can soft-delete equipment

**Purpose**: Implements FR-034 (edit) and FR-032/FR-033 (admin delete)

- [ ] T110 Extend Equipment service with update, softDelete in backend/src/services/equipment.service.ts
- [ ] T111 Implement PATCH /api/equipment/:id endpoint in backend/src/api/routes/equipment.routes.ts
- [ ] T112 Implement DELETE /api/equipment/:id (admin only) in backend/src/api/routes/equipment.routes.ts
- [ ] T113 Create Zod schema for update equipment in backend/src/schemas/equipment.schema.ts
- [ ] T114 Create EditEquipment page with form in frontend/src/pages/EditEquipment.vue
- [ ] T115 Add edit/delete actions to EquipmentDetails in frontend/src/pages/EquipmentDetails.vue
- [ ] T116 Create DeleteConfirmDialog component in frontend/src/components/common/DeleteConfirmDialog.vue

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T117 [P] Add loading skeletons to all list views in frontend/src/components/common/SkeletonLoader.vue
- [ ] T118 [P] Add error boundaries and retry logic to API calls in frontend/src/services/api.ts
- [ ] T119 [P] Ensure all touch targets are 44x44px minimum across all components
- [ ] T120 Add connectivity error handling with user-friendly messages in frontend/src/composables/useApi.ts
- [ ] T121 Create consistent status badges (Available, Checked-Out, etc.) in frontend/src/components/common/StatusBadge.vue
- [ ] T122 [P] Add Telegram theme integration (colors, dark mode) in frontend/src/plugins/vuetify.ts
- [ ] T123 Setup production build script in backend/package.json
- [ ] T124 Configure backend to serve frontend/dist in production in backend/src/server.ts
- [ ] T125 Create deployment instructions in README.md

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-8)**: All depend on Foundational phase completion
  - User stories can proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3 → P4 → P5 → P6)
- **Edit/Delete (Phase 9)**: Depends on US1 and US4 completion (equipment must exist)
- **Polish (Phase 10)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational - Uses location list (minimal dependency)
- **User Story 3 (P3)**: Can start after Foundational - No dependencies on other stories
- **User Story 4 (P4)**: Can start after Foundational - Uses type/location lists
- **User Story 5 (P5)**: Can start after Foundational - No dependencies on other stories
- **User Story 6 (P6)**: Can start after Foundational - No dependencies on other stories

### Within Each User Story

- Models/Services before endpoints
- Backend before frontend (API must exist)
- Core implementation before UI polish
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch backend services in parallel:
Task T029: "Create Equipment service in backend/src/services/equipment.service.ts"
Task T030: "Create EquipmentAction service in backend/src/services/action.service.ts"
Task T031: "Create Attachment service in backend/src/services/attachment.service.ts"

# Launch frontend components in parallel:
Task T043: "Create EquipmentInfo component in frontend/src/components/equipment/EquipmentInfo.vue"
Task T044: "Create EquipmentHistory component in frontend/src/components/equipment/EquipmentHistory.vue"
Task T045: "Create EquipmentDocuments component in frontend/src/components/equipment/EquipmentDocuments.vue"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Scan, View, Attach Files)
4. **STOP and VALIDATE**: Test QR scanning, equipment details, file attachment
5. Deploy/demo if ready - users can already scan and document equipment

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy (MVP!)
3. Add User Story 2 → Test independently → Deploy (checkout/return)
4. Add User Story 3 → Test independently → Deploy (search/browse)
5. Add User Story 4 → Test independently → Deploy (add equipment)
6. Add User Story 5 → Test independently → Deploy (admin types/locations)
7. Add User Story 6 → Test independently → Deploy (admin users)
8. Each story adds value without breaking previous stories

### Single Developer Strategy

Follow priority order: P1 → P2 → P3 → P4 → P5 → P6
Each phase is a complete deliverable that can be tested and demoed.

---

## Summary

| Phase | Description | Task Count |
|-------|-------------|------------|
| 1 | Setup | 9 |
| 2 | Foundational | 19 |
| 3 | User Story 1 (P1) - Scan/View/Files | 20 |
| 4 | User Story 2 (P2) - Checkout/Return | 11 |
| 5 | User Story 3 (P3) - Search/Browse | 11 |
| 6 | User Story 4 (P4) - Add Equipment | 12 |
| 7 | User Story 5 (P5) - Types/Locations | 13 |
| 8 | User Story 6 (P6) - User Management | 14 |
| 9 | Edit/Delete | 7 |
| 10 | Polish | 9 |
| **Total** | | **125** |

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- No tests included (not explicitly requested) - add if needed