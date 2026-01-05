# Feature Specification: Equipment Manager Mini App

**Feature Branch**: `001-equipment-manager`
**Created**: 2026-01-05
**Status**: Draft
**Input**: User description: "Equipment management application for tracking measuring instruments, hardware, and material resources with QR code scanning, location tracking, and file attachments."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Scan, View, and Update Equipment (Priority: P1)

A user opens the app, scans a QR code on a piece of equipment, and immediately sees all relevant information: what the equipment is, where it's supposed to be located, who currently has it, its usage history, and any attached documents. Any user can attach files (photos, manuals, calibration certificates) to equipment without needing to check it out first.

**Why this priority**: This is the core value proposition - instant equipment lookup via QR scanning combined with the ability to immediately document equipment with photos and files. Without this, users cannot access or contribute to equipment information, making all other features useless.

**Independent Test**: Can be fully tested by printing a test QR code, scanning it within Telegram, verifying the equipment details page displays correctly, and attaching a document or photo. Delivers immediate value for equipment lookup and documentation.

**Acceptance Scenarios**:

1. **Given** a user has the Mini App open, **When** they tap "Scan" and point their camera at a valid equipment QR code, **Then** they see the equipment details page with name, type, current location, current holder, and status within 2 seconds.
2. **Given** a user is viewing equipment details, **When** they tap "History", **Then** they see a chronological list of all location changes with timestamps and user names.
3. **Given** a user is viewing equipment details, **When** they tap "Documents", **Then** they see a list of all attached files (documents, photos, videos) with preview thumbnails.
4. **Given** a user is viewing equipment details, **When** they tap "Attach File" or "Add Photo", **Then** they can select files from their device or take a photo, and the file is uploaded and linked to the equipment.
5. **Given** a user has attached a file, **When** the upload completes, **Then** the file appears in the Documents list with the user's name and timestamp, and other users can view it immediately.
6. **Given** a user scans an unrecognized QR code, **When** the scan completes, **Then** they see a clear error message "Equipment not found" with an option to report the issue.

---

### User Story 2 - Check Out Equipment (Priority: P2)

A user needs to take a piece of equipment to another location. They scan the QR code, tap "Take Equipment", specify the destination room/location, and confirm. The system records this action and updates the equipment's current holder and location.

**Why this priority**: After viewing equipment, the most common action is taking it somewhere. This enables the fundamental tracking workflow.

**Independent Test**: Can be tested by scanning equipment, completing the checkout flow, and verifying the equipment record shows the new location and holder.

**Acceptance Scenarios**:

1. **Given** a user is viewing equipment that is marked "Available", **When** they tap "Take Equipment", **Then** they see a form to select or enter a destination location.
2. **Given** a user has selected a destination, **When** they tap "Confirm", **Then** the equipment status changes to "Checked-Out", the user becomes the current holder, and a history entry is created.
3. **Given** a user has equipment checked out to them, **When** they tap "Return Equipment", **Then** the equipment returns to "Available" status at its home location.
4. **Given** equipment is already checked out to another user, **When** a different user views it, **Then** they see who has it, where they took it, and when.

---

### User Story 3 - Search and Browse Equipment (Priority: P3)

A user needs to find equipment without having the physical item to scan. They search by name, type, or room number to locate what they need.

**Why this priority**: Users need to find equipment before they can scan it. Search enables discovery and planning.

**Independent Test**: Can be tested by entering search terms and verifying matching equipment appears in results with correct location information.

**Acceptance Scenarios**:

1. **Given** a user is on the home screen, **When** they tap the search field and enter a partial equipment name, **Then** they see a filtered list of matching equipment with current locations.
2. **Given** a user wants to browse by location, **When** they select "Browse by Room", **Then** they see a list of rooms with equipment counts, and can tap to see equipment in each room.
3. **Given** a user wants to browse by type, **When** they select "Browse by Type", **Then** they see equipment categories (e.g., Oscilloscopes, Cables) and can drill down to specific items.
4. **Given** search returns no results, **When** the user views the empty state, **Then** they see a message suggesting alternative search terms or the option to add new equipment.

---

### User Story 4 - Add New Equipment (Priority: P4)

Any user adds a new piece of equipment to the system. They specify the type, name, description, home location, and optionally attach documents. The system generates a unique ID and QR code for the equipment.

**Why this priority**: The system needs equipment data to be useful. All users should be empowered to add equipment when they encounter items that aren't yet tracked.

**Independent Test**: Can be tested by completing the add equipment form and verifying the new item appears in search results with a generated QR code.

**Acceptance Scenarios**:

1. **Given** any user taps "Add Equipment", **When** they fill in required fields (type, name, location), **Then** they can submit and the system creates the equipment with a unique ID.
2. **Given** new equipment is created, **When** the user views the equipment details, **Then** they see a QR code that can be printed or saved as an image.
3. **Given** equipment is being added, **When** the user attaches files (photos, manuals, calibration certificates), **Then** the files are linked to the equipment record.
4. **Given** a user tries to submit without required fields, **When** they tap submit, **Then** they see validation errors indicating which fields are missing.

---

### User Story 5 - Manage Equipment Types and Locations (Priority: P5)

An administrator manages the master data: equipment types/categories and room/location definitions. This ensures consistent data entry and enables meaningful filtering.

**Why this priority**: Master data management is foundational but less frequently used than daily equipment operations.

**Independent Test**: Can be tested by adding a new equipment type or room, then verifying it appears in the selection lists when adding equipment.

**Acceptance Scenarios**:

1. **Given** an admin accesses settings, **When** they tap "Manage Equipment Types", **Then** they see a list of existing types with options to add, edit, or deactivate.
2. **Given** an admin is adding a new equipment type, **When** they enter a name and optional description, **Then** the type becomes available in the "Add Equipment" form.
3. **Given** an admin accesses settings, **When** they tap "Manage Locations", **Then** they see a list of rooms/locations with options to add, edit, or deactivate.
4. **Given** an admin deactivates a location, **When** users try to check out equipment, **Then** that location no longer appears as a selectable destination.

---

### User Story 6 - User Management (Priority: P6)

An administrator adds new users to the system and assigns roles (regular user vs. admin). Users are identified by their Telegram identity.

**Why this priority**: User management is needed for multi-user operation but is typically done during initial setup rather than daily use.

**Independent Test**: Can be tested by adding a new user via their Telegram username and verifying they can access the app with appropriate permissions.

**Acceptance Scenarios**:

1. **Given** a new person opens the Mini App for the first time, **When** they are not in the system, **Then** they see a message that they need to be added by an administrator.
2. **Given** an admin accesses user management, **When** they tap "Add User", **Then** they can enter a Telegram username and assign a role (User or Admin).
3. **Given** an admin views the user list, **When** they tap a user, **Then** they see that user's activity history and can change their role or deactivate them.
4. **Given** a deactivated user tries to access the app, **When** the app loads, **Then** they see a message that their access has been revoked.

---

### Edge Cases

- What happens when a user scans a QR code for equipment that has been deleted? System shows "Equipment not found" with option to report.
- What happens when multiple users try to check out the same equipment simultaneously? First confirmed checkout wins; second user sees "Equipment is no longer available."
- How does the system handle equipment with no assigned home location? Equipment can exist without a home location but displays "No home location set" in details.
- What happens when a user loses connectivity mid-checkout? Action is queued locally and synced when connectivity returns; user sees pending status.
- How does the system handle corrupted or unreadable QR codes? Scanner shows "Could not read code" with option to retry or enter ID manually.
- What happens when file uploads fail? User sees retry option; partial uploads are discarded.

## Requirements *(mandatory)*

### Functional Requirements

**Equipment Management**

- **FR-001**: System MUST generate a unique alphanumeric identifier for each equipment item upon creation.
- **FR-002**: System MUST encode equipment identifiers as QR codes that can be printed.
- **FR-003**: System MUST display equipment identifiers in both QR code and human-readable alphanumeric format.
- **FR-004**: System MUST track equipment status: Available, Checked-Out, Maintenance, or Broken.
- **FR-005**: System MUST record equipment type, name, description, and home location.
- **FR-006**: System MUST support hierarchical equipment categorization (type → individual unit).

**Location Tracking**

- **FR-007**: System MUST maintain a list of valid room/location identifiers.
- **FR-008**: System MUST record equipment current location and home location separately.
- **FR-009**: System MUST track which user currently holds checked-out equipment.
- **FR-010**: System MUST record all location changes with timestamp, user, from-location, and to-location.

**QR Code Scanning**

- **FR-011**: System MUST integrate with device camera for QR code scanning.
- **FR-012**: System MUST provide haptic feedback upon successful scan.
- **FR-013**: System MUST support continuous scanning mode for multiple items.
- **FR-014**: System MUST display clear error messages for unrecognized or invalid codes.

**File Attachments**

- **FR-015**: System MUST allow any user to attach documents, photos, and videos to equipment records without requiring checkout.
- **FR-016**: System MUST display attached files with preview thumbnails where applicable.
- **FR-017**: System MUST record attachment upload timestamp and uploading user.
- **FR-018**: System MUST enforce file size limits (maximum 10MB per file).
- **FR-019**: System MUST allow users to capture photos directly from the equipment details screen and attach them immediately.

**History and Audit**

- **FR-020**: System MUST maintain complete, immutable history of all equipment actions.
- **FR-021**: System MUST display equipment usage history in chronological order.
- **FR-022**: System MUST record user identity and timestamp for every action.

**User Management**

- **FR-023**: System MUST authenticate users via Telegram identity (no separate login).
- **FR-024**: System MUST support two user roles: Regular User and Administrator.
- **FR-025**: System MUST restrict administrative functions (user management, type/location management) to Administrator role.
- **FR-026**: System MUST allow all users to add equipment and attach files.
- **FR-027**: System MUST allow administrators to add, edit, and deactivate users.

**Search and Browse**

- **FR-028**: System MUST provide search by equipment name, type, and location.
- **FR-029**: System MUST allow browsing equipment by room/location.
- **FR-030**: System MUST allow browsing equipment by type/category.
- **FR-031**: System MUST display current location and status in search results.

### Key Entities

- **Equipment**: Individual trackable item with unique ID, name, type, description, home location, current location, status, and attached files. Core entity of the system.

- **Equipment Type**: Category of equipment (e.g., "Oscilloscope", "Network Analyzer", "Cable"). Enables consistent categorization and filtering.

- **Location**: Physical room or area where equipment can be stored or used. Identified by room number or name.

- **User**: Person who can interact with the system. Identified by Telegram identity. Has a role (User or Admin) and activity history.

- **Equipment Action**: Immutable record of an action taken on equipment. Includes action type (checkout, return, status change), timestamp, user, and before/after state.

- **Attachment**: File (document, photo, video) linked to an equipment item. Includes file metadata, upload timestamp, and uploading user.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can locate equipment by scanning its QR code within 3 seconds of pointing the camera.
- **SC-002**: Users can complete equipment checkout (scan → select destination → confirm) in under 30 seconds.
- **SC-003**: Users can attach a photo or document to equipment within 15 seconds of viewing the equipment details.
- **SC-004**: Search results appear within 1 second of entering search text.
- **SC-005**: System supports at least 1,000 equipment items without performance degradation.
- **SC-006**: System supports at least 50 concurrent users without errors.
- **SC-007**: 95% of users successfully complete their first equipment lookup without assistance.
- **SC-008**: Equipment location accuracy is 100% - the system always reflects the last recorded location.
- **SC-009**: Zero data loss for equipment actions, even during connectivity interruptions.
- **SC-010**: Any user can add new equipment in under 2 minutes.
- **SC-011**: Complete audit trail is available for any equipment item within 2 taps from the details screen.

## Assumptions

The following reasonable defaults have been assumed based on the feature description and industry standards:

1. **Authentication**: Users are authenticated via Telegram identity; no separate registration or login flow is needed.
2. **Role-based access**: Two roles (User and Admin) are sufficient. All users can add equipment, attach files, check out equipment, and search. Only admins can manage users, equipment types, and locations.
3. **File types**: Standard document formats (PDF, images, videos) are supported; specialized CAD or scientific formats are not required.
4. **Single organization**: The system serves one organization; multi-tenant architecture is not required.
5. **Language**: The interface is in English; internationalization can be added later if needed.
6. **Equipment ID format**: Alphanumeric IDs (e.g., "EQ-001-ABC") are generated automatically; custom ID schemes are not required.
7. **Notifications**: The system does not send proactive notifications (e.g., "Your equipment is overdue"); users check status manually.
8. **File attachments**: Users can attach files without checking out equipment; checkout is only required to physically take equipment to another location.
