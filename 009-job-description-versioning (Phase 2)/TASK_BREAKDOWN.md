# Task Breakdown: Job Description Versioning & History

**Source**: [009-job-description-versioning (Phase 2)/spec.md](./spec.md)  
**Generated**: 2026-02-01  
**Status**: Planning

---

## Overview

This feature implements comprehensive job description versioning to track JD changes, lock versions during interview sessions, support rollback, and provide audit trails for compliance and fair candidate evaluation. The system automatically creates versions on every JD edit, captures and locks the JD version when a candidate starts an interview, and maintains complete audit history for all changes.

**Key Capabilities**:

- Automatic version creation on JD updates
- Version locking at interview session start
- Version history view with side-by-side diff comparison
- Rollback and version management with audit trails
- Multi-tenant isolation and permission enforcement
- Integration with interview sessions, status tracking, and token usage

---

## Database

### Main Task 1: JD Versioning Schema Design

**Description**: Design and implement database schema for storing job description versions, change logs, and version relationships.

**Requirement Reference**: FR-001, FR-008, FR-009, Entity definitions

#### Subtask 1.1: JobDescriptionVersion Table

- **Description**: Create migration for `job_description_versions` table to store all historical JD versions with metadata
- **Columns**:
  - `id` (PK, UUID)
  - `job_id` (FK to jobs, indexed)
  - `company_id` (FK to companies, indexed for multi-tenant isolation)
  - `version_number` (INT, auto-increment per job)
  - `content` (LONGTEXT for small JDs, VARCHAR reference to S3 for large JDs)
  - `content_hash` (VARCHAR, SHA256 for deduplication)
  - `created_at` (TIMESTAMP, indexed for temporal queries)
  - `created_by_user_id` (FK to users)
  - `change_summary` (TEXT, nullable)
  - `is_active` (BOOL, default false, indexed)
  - `is_rollback` (BOOL, default false)
  - `rollback_from_version_id` (FK to job_description_versions, nullable)
  - `is_deleted` (BOOL, default false)
- **Indexes**:
  - Composite: `(job_id, version_number)` for version history queries
  - Composite: `(company_id, created_at)` for audit queries
  - Single: `is_active` for current version lookups
  - Single: `content_hash` for deduplication
- **Constraints**:
  - Unique: `(job_id, version_number)`
  - Check: Only one active version per job
- **Used By**: Backend Tasks 2.1-2.7
- **Acceptance**: Migration runs successfully; sample data inserted and queried

#### Subtask 1.2: JobDescriptionChangeLog Table

- **Description**: Create migration for `job_description_change_logs` audit table tracking all version transitions
- **Columns**:
  - `id` (PK, UUID)
  - `job_id` (FK to jobs, indexed)
  - `company_id` (FK to companies, indexed)
  - `old_version_id` (FK to job_description_versions, nullable)
  - `new_version_id` (FK to job_description_versions)
  - `change_type` (ENUM: 'create', 'update', 'rollback', 'delete')
  - `actor_user_id` (FK to users)
  - `timestamp` (TIMESTAMP, indexed)
  - `reason` (TEXT, nullable)
  - `change_metadata` (JSON, nullable for future field-level diffs)
- **Indexes**:
  - Composite: `(job_id, timestamp)` for audit trail queries
  - Composite: `(company_id, timestamp)` for compliance reporting
  - Single: `actor_user_id` for user activity queries
- **Used By**: Backend Tasks 2.2, 2.5, 2.6
- **Acceptance**: Migration runs successfully; foreign keys validated

#### Subtask 1.3: Modify JobDescription Table

- **Description**: Add columns to `job_descriptions` table to reference current version and cache version count
- **New Columns**:
  - `current_version_id` (FK to job_description_versions, nullable initially)
  - `version_count` (INT, default 1)
- **Migration Strategy**:
  - Add columns with nullable constraint
  - Backfill existing jobs with version 1.0
  - Update foreign key relationships
  - Add NOT NULL constraint after backfill
- **Indexes**: Single index on `current_version_id`
- **Used By**: Backend Tasks 2.1, 2.3
- **Acceptance**: Existing jobs migrated with initial versions; FK constraints enforced

#### Subtask 1.4: Modify InterviewSession Table

- **Description**: Add columns to `interview_sessions` table to capture and lock JD version at session start
- **New Columns**:
  - `jd_version_id` (FK to job_description_versions, NOT NULL)
  - `jd_locked_at` (TIMESTAMP, NOT NULL)
- **Migration Strategy**:
  - Add columns with nullable constraint
  - Backfill existing sessions with current JD version
  - Add NOT NULL constraint after backfill
- **Indexes**: Composite index on `(job_id, jd_version_id)` for version usage queries
- **Used By**: Backend Tasks 3.1, 3.2
- **Acceptance**: All interview sessions reference a valid JD version; queries optimized

#### Subtask 1.5: Modify CandidateStatusHistory Table

- **Description**: Add column to `candidate_status_history` table to track which JD version was active during status changes
- **New Columns**:
  - `jd_version_id` (FK to job_description_versions, nullable)
- **Migration Strategy**:
  - Add column as nullable
  - Backfill from associated interview session's JD version
  - Keep nullable for edge cases (manual status changes)
- **Indexes**: Single index on `jd_version_id` for audit queries
- **Used By**: Backend Task 4.1 (Feature 003 integration)
- **Acceptance**: Historical data backfilled; future status changes auto-populate field

---

## Backend

### Main Task 2: Job Description Versioning Service

**Description**: Implement core service layer for managing JD versions, including automatic version creation, version history queries, and audit logging.

**Requirement Reference**: FR-001, FR-004, FR-007, FR-008

#### Subtask 2.1: Create Initial JD Version on Job Creation

- **Description**: Implement logic to automatically create version 1.0 when a new job is created with a job description
- **Service Method**: `JobDescriptionService.createInitialVersion(jobId, content, createdByUserId)`
- **Logic**:
  - Calculate content hash (SHA256)
  - Create `JobDescriptionVersion` record (version_number=1, is_active=true)
  - Update `JobDescription.current_version_id` FK
  - Create `JobDescriptionChangeLog` entry (change_type='create', old_version_id=null)
  - Transaction: All-or-nothing atomic operation
- **Called By**: Backend Task 2.2 (Create Job API)
- **Depends On**: Database Task 1.1, 1.2, 1.3
- **Tested By**: Testing Task 1.1
- **Acceptance**: New jobs have version 1.0; audit log created; FK constraints validated

#### Subtask 2.2: Update JD and Create New Version

- **Description**: Implement atomic operation to save JD changes and create new version automatically
- **Service Method**: `JobDescriptionService.updateAndCreateVersion(jobId, newContent, changeSummary, actorUserId)`
- **Logic**:
  - Validate user has `job.edit` permission (Feature 004 integration)
  - Check if content changed (compare content_hash)
  - If unchanged, return early (no new version)
  - Transaction start:
    - Mark current version as `is_active=false`
    - Increment version_number (query `MAX(version_number) + 1`)
    - Create new `JobDescriptionVersion` (is_active=true)
    - Update `JobDescription.current_version_id` and increment `version_count`
    - Create `JobDescriptionChangeLog` entry (change_type='update')
  - Transaction commit
- **Called By**: Backend Task 2.3 (Update Job API)
- **Depends On**: Database Task 1.1, 1.2, 1.3
- **Tested By**: Testing Task 1.2
- **Acceptance**: Update operation completes in <5 seconds; version number auto-increments; old version preserved

##### Sub-Subtask 2.2.1: Content Hash Calculation

- **Description**: Implement SHA256 hashing for JD content to detect actual changes and enable deduplication
- **Why**: Prevents creating duplicate versions when content hasn't changed (e.g., user saves without edits)
- **Acceptance**: Hash matches for identical content; differs for any character change

##### Sub-Subtask 2.2.2: Concurrent Edit Conflict Detection

- **Description**: Implement optimistic locking or version check to detect concurrent edits (last-write-wins)
- **Why**: Prevents lost updates when two HR managers edit same JD simultaneously
- **Logic**: Compare client's `current_version_id` with server's; if mismatch, return 409 Conflict with "JD was updated by another user"
- **Acceptance**: Second editor notified of conflict; must refresh and retry

#### Subtask 2.3: Query Version History

- **Description**: Implement service method to retrieve all versions for a job with metadata
- **Service Method**: `JobDescriptionService.getVersionHistory(jobId, companyId)`
- **Return**: List of `{ version_number, created_at, created_by_name, change_summary, is_active, candidate_count }`
- **Query**:
  - Join `job_description_versions` with `users` for creator name
  - Join with `interview_sessions` to count candidates per version
  - Filter by `company_id` (multi-tenant isolation)
  - Order by `version_number DESC`
- **Called By**: Frontend Task 1.2 (via Backend Task 2.8)
- **Depends On**: Database Task 1.1, 1.4
- **Tested By**: Testing Task 1.3
- **Acceptance**: Query returns complete history in <3 seconds for 50+ versions; correct candidate counts

#### Subtask 2.4: Retrieve Specific JD Version Content

- **Description**: Implement service method to fetch full content of a specific version
- **Service Method**: `JobDescriptionService.getVersionContent(versionId, companyId)`
- **Logic**:
  - Validate version belongs to company (multi-tenant check)
  - Retrieve `JobDescriptionVersion.content`
  - If content is S3 reference, fetch from blob storage
  - Return content as string or structured object
- **Called By**: Frontend Task 1.3 (via Backend Task 2.9), Backend Task 2.6, 3.2
- **Depends On**: Database Task 1.1
- **Tested By**: Testing Task 1.4
- **Acceptance**: Content retrieval in <2 seconds for 50KB JDs; S3 fetch succeeds for large JDs

#### Subtask 2.5: Compare Two JD Versions (Diff Generation)

- **Description**: Implement diff algorithm to generate side-by-side comparison with additions/deletions/unchanged sections
- **Service Method**: `JobDescriptionService.compareVersions(versionId1, versionId2, companyId)`
- **Return**: Diff object with `{ added: string[], deleted: string[], unchanged: string[] }` or structured field-level diff
- **Algorithm**: Use Myers diff algorithm or similar library (e.g., `diff-match-patch`)
- **Called By**: Frontend Task 1.3 (via Backend Task 2.10)
- **Depends On**: Database Task 1.1, Backend Task 2.4
- **Tested By**: Testing Task 1.5
- **Acceptance**: Diff generation in <2 seconds for 50KB JDs; highlights correct additions/deletions

#### Subtask 2.6: Rollback to Previous Version

- **Description**: Implement service method to revert JD to a previous version by creating new version with old content
- **Service Method**: `JobDescriptionService.rollbackToVersion(jobId, targetVersionId, actorUserId)`
- **Logic**:
  - Validate user has `job.edit` permission
  - Fetch content from target version
  - Transaction start:
    - Mark current version as `is_active=false`
    - Create new version with target content (is_rollback=true, rollback_from_version_id=targetVersionId)
    - Update `JobDescription.current_version_id` and increment `version_count`
    - Create `JobDescriptionChangeLog` entry (change_type='rollback')
  - Transaction commit
- **Called By**: Frontend Task 1.4 (via Backend Task 2.11)
- **Depends On**: Database Task 1.1, 1.2, 1.3, Backend Task 2.4
- **Tested By**: Testing Task 1.6
- **Acceptance**: Rollback completes in <5 seconds; new version created with correct content; audit log shows rollback

##### Sub-Subtask 2.6.1: Activate Previous Version (Non-Destructive Alternative)

- **Description**: Implement alternative rollback that promotes previous version to current without creating new version
- **Why**: Supports use case where HR wants to "activate" old version without version number inflation
- **Service Method**: `JobDescriptionService.activatePreviousVersion(jobId, targetVersionId, actorUserId)`
- **Logic**: Mark target version as `is_active=true`, mark others as `is_active=false`
- **Acceptance**: Version activated; no new version created; audit log records activation

#### Subtask 2.7: View JD "As of Date"

- **Description**: Implement temporal query to retrieve JD version that was active on a specific date
- **Service Method**: `JobDescriptionService.getVersionAsOfDate(jobId, date, companyId)`
- **Query**: `SELECT * FROM job_description_versions WHERE job_id=? AND created_at <= ? ORDER BY created_at DESC LIMIT 1`
- **Return**: Version content and metadata
- **Called By**: Frontend Task 1.3 (via Backend Task 2.12)
- **Depends On**: Database Task 1.1
- **Tested By**: Testing Task 1.7
- **Acceptance**: Query returns correct version for past dates; handles edge cases (date before job created)

#### Subtask 2.8: API Endpoint - GET /api/jobs/{jobId}/versions

- **Description**: Implement API endpoint to retrieve version history for a job
- **Request**: Path param `jobId`; Query param `includeDeleted` (optional, default false)
- **Response**: `{ versions: [{ id, version_number, created_at, created_by, change_summary, is_active, candidate_count }] }`
- **Authorization**: Requires `job.view` or `job.version.view` permission
- **Service Method**: Calls `JobDescriptionService.getVersionHistory()` from Backend Task 2.3
- **Called By**: Frontend Task 1.2
- **Tested By**: Testing Task 2.1
- **Acceptance**: Returns version history in <3 seconds; respects multi-tenant isolation; 401 if unauthorized

#### Subtask 2.9: API Endpoint - GET /api/jobs/{jobId}/versions/{versionId}

- **Description**: Implement API endpoint to retrieve specific version content
- **Request**: Path params `jobId`, `versionId`
- **Response**: `{ version: { id, version_number, content, created_at, created_by, change_summary } }`
- **Authorization**: Requires `job.version.view` permission
- **Service Method**: Calls `JobDescriptionService.getVersionContent()` from Backend Task 2.4
- **Called By**: Frontend Task 1.3
- **Tested By**: Testing Task 2.2
- **Acceptance**: Returns version content in <2 seconds; handles large JDs (S3 fetch)

#### Subtask 2.10: API Endpoint - POST /api/jobs/{jobId}/versions/compare

- **Description**: Implement API endpoint to compare two JD versions
- **Request**: Body `{ version1_id: string, version2_id: string }`
- **Response**: `{ diff: { added: string[], deleted: string[], unchanged: string[] } }`
- **Authorization**: Requires `job.version.view` permission
- **Service Method**: Calls `JobDescriptionService.compareVersions()` from Backend Task 2.5
- **Called By**: Frontend Task 1.3
- **Tested By**: Testing Task 2.3
- **Acceptance**: Returns diff in <2 seconds for 50KB JDs; correct highlighting

#### Subtask 2.11: API Endpoint - POST /api/jobs/{jobId}/versions/rollback

- **Description**: Implement API endpoint to rollback JD to a previous version
- **Request**: Body `{ target_version_id: string, reason?: string }`
- **Response**: `{ new_version: { id, version_number, content } }`
- **Authorization**: Requires `job.edit` permission
- **Service Method**: Calls `JobDescriptionService.rollbackToVersion()` from Backend Task 2.6
- **Called By**: Frontend Task 1.4
- **Tested By**: Testing Task 2.4
- **Acceptance**: Rollback completes in <5 seconds; returns new version; audit log created

#### Subtask 2.12: API Endpoint - GET /api/jobs/{jobId}/versions/as-of-date

- **Description**: Implement API endpoint to retrieve JD version active on a specific date
- **Request**: Query param `date` (ISO 8601 format)
- **Response**: `{ version: { id, version_number, content, created_at } }`
- **Authorization**: Requires `job.version.view` permission
- **Service Method**: Calls `JobDescriptionService.getVersionAsOfDate()` from Backend Task 2.7
- **Called By**: Frontend Task 1.3
- **Tested By**: Testing Task 2.5
- **Acceptance**: Returns correct version for past dates; 404 if date before job creation

#### Subtask 2.13: API Endpoint - PUT /api/jobs/{jobId}

- **Description**: Modify existing Update Job API to trigger automatic version creation when JD content changes
- **Request**: Body `{ title?, description?, requirements?, ... changeSummary?: string }`
- **Response**: `{ job: { id, title, current_version_id, version_count, ... } }`
- **Authorization**: Requires `job.edit` permission
- **Logic**: If any JD content fields changed, call `JobDescriptionService.updateAndCreateVersion()`
- **Service Method**: Calls `JobDescriptionService.updateAndCreateVersion()` from Backend Task 2.2
- **Called By**: Frontend Task 1.1
- **Tested By**: Testing Task 2.6
- **Acceptance**: Update creates new version; no version if content unchanged; returns updated job with new version metadata

---

### Main Task 3: Interview Session JD Locking

**Description**: Implement logic to capture and lock JD version when candidate starts interview session, ensuring immutable version reference throughout interview.

**Requirement Reference**: FR-002, FR-003, FR-009

#### Subtask 3.1: Capture JD Version at Session Start

- **Description**: Implement logic to store current JD version ID when candidate opens interview link and verifies email
- **Service Method**: `InterviewSessionService.startSession(candidateId, jobId)`
- **Logic**:
  - Fetch current JD version from `JobDescription.current_version_id`
  - Create `InterviewSession` record with `jd_version_id` and `jd_locked_at = NOW()`
  - Return session ID for subsequent interview operations
- **Called By**: Backend Task 3.3 (Start Interview API), Feature 001 interview workflow
- **Depends On**: Database Task 1.3, 1.4
- **Tested By**: Testing Task 3.1
- **Acceptance**: 100% of new sessions have non-null `jd_version_id`; locked at correct timestamp

#### Subtask 3.2: Use Locked JD Version for AI Operations

- **Description**: Implement logic to pass locked JD version content to AI services (resume scoring, question generation)
- **Service Method**: `InterviewSessionService.getLockedJDVersion(sessionId)`
- **Logic**:
  - Fetch `InterviewSession.jd_version_id`
  - Retrieve version content via `JobDescriptionService.getVersionContent()`
  - Return content to AI service caller
- **Called By**: Feature 001 AI integration (resume scoring, question generation services)
- **Depends On**: Database Task 1.4, Backend Task 2.4
- **Tested By**: Testing Task 3.2
- **Acceptance**: AI operations use locked version; version unchanged even if JD updated mid-interview

#### Subtask 3.3: API Endpoint - POST /api/interviews/start

- **Description**: Modify existing Start Interview API to capture JD version at session start
- **Request**: Body `{ candidate_id: string, job_id: string }`
- **Response**: `{ session_id: string, jd_version_id: string, jd_locked_at: timestamp }`
- **Authorization**: Requires candidate email verification (Feature 001)
- **Service Method**: Calls `InterviewSessionService.startSession()` from Backend Task 3.1
- **Called By**: Frontend (candidate interview page), Feature 001 workflow
- **Tested By**: Testing Task 3.3
- **Acceptance**: Session created with JD version locked; returns session metadata

---

### Main Task 4: Feature Integration

**Description**: Integrate JD versioning with other features (Status Tracking, Permissions, Token Tracking).

**Requirement Reference**: FR-010, FR-012, Dependencies section

#### Subtask 4.1: Populate JD Version in Candidate Status History

- **Description**: Modify candidate status change logic to capture JD version when status is updated
- **Service Method**: `CandidateService.updateStatus(candidateId, newStatus, actorUserId)`
- **Logic**:
  - When status changes, fetch `jd_version_id` from candidate's most recent `InterviewSession`
  - Populate `CandidateStatusHistory.jd_version_id`
  - Continue with existing status update logic (Feature 003)
- **Called By**: Feature 003 status tracking workflow
- **Depends On**: Database Task 1.5, Feature 003 implementation
- **Tested By**: Testing Task 4.1
- **Acceptance**: All status changes include JD version reference; audit trail complete

#### Subtask 4.2: Enforce Permissions for JD Version Operations

- **Description**: Integrate with Feature 004 RBAC to enforce permissions on JD version operations
- **Permissions**:
  - `job.edit` → Create/update/rollback versions
  - `job.version.view` → View version history, retrieve specific versions
  - `job.history.view` → View audit logs (`JobDescriptionChangeLog`)
- **Service Layer**: Add permission checks in all `JobDescriptionService` methods
- **API Layer**: Add middleware/decorators to enforce permissions in API endpoints
- **Called By**: All Backend Task 2 methods
- **Depends On**: Feature 004 permissions system
- **Tested By**: Testing Task 4.2
- **Acceptance**: Unauthorized users receive 403; permission checks logged in audit trail

##### Sub-Subtask 4.2.1: Define New Permissions in Feature 004

- **Description**: Add `job.version.view` and `job.history.view` permissions to Feature 004 permission registry
- **Why**: Allows granular control (e.g., some users view current JD but not version history)
- **Acceptance**: Permissions registered; assignable to groups; enforced at API level

#### Subtask 4.3: Allocate Tokens to JD Versions (Feature 005 Integration)

- **Description**: Track "Job Description Analysis" token usage per JD version for cost reporting
- **Service Method**: `TokenUsageService.recordJDAnalysis(jdVersionId, tokenCount, operationType)`
- **Logic**: When AI analyzes JD (resume scoring, question generation), log token usage with `jd_version_id` reference
- **Called By**: Feature 001 AI integration, Feature 005 token tracking
- **Depends On**: Feature 005 implementation
- **Tested By**: Testing Task 4.3
- **Acceptance**: Token usage attributable to specific JD versions; cost-per-version report accurate

---

## Frontend

### Main Task 1: Job Description Editor & Version Management UI

**Description**: Build UI components for editing job descriptions, viewing version history, comparing versions, and managing rollbacks.

**User Story Reference**: User Story 1 (HR Updates JD), User Story 3 (View & Compare), User Story 4 (Rollback)

#### Subtask 1.1: Enhanced Job Description Editor

- **Description**: Modify existing job edit form to support version creation with optional change summary
- **Components**:
  - `JobDescriptionEditor` (existing) → Add "Save with Summary" button
  - `ChangeSummaryDialog` (new) → Modal for entering change summary
- **Uses**: Backend Task 2.13 (PUT /api/jobs/{jobId})
- **Tested By**: Testing Task 5.1
- **Acceptance**: HR can edit JD, optionally add summary, save in <5 seconds; UI updates with new version number
- **Files**: `components/JobDescriptionEditor.tsx`, `components/ChangeSummaryDialog.tsx`

##### Sub-Subtask 1.1.1: Concurrent Edit Conflict Notification

- **Description**: Display notification when save fails due to concurrent edit (409 Conflict response)
- **Why**: Prevents lost updates; alerts HR to refresh and retry
- **UI**: Toast notification: "Job description was updated by another user. Please refresh and try again."
- **Acceptance**: User sees notification; offered option to refresh or view diff with conflicting version

#### Subtask 1.2: Version History View

- **Description**: Create UI to display all JD versions in a table with metadata and candidate usage counts
- **Components**:
  - `VersionHistoryPanel` (new) → Collapsible panel on job details page
  - `VersionHistoryTable` (new) → Table with columns: Version, Date, Author, Summary, Candidates, Actions
- **Uses**: Backend Task 2.8 (GET /api/jobs/{jobId}/versions)
- **Tested By**: Testing Task 5.2
- **Acceptance**: Version history loads in <3 seconds for 50+ versions; displays candidate counts per version; sortable/filterable
- **Files**: `components/VersionHistoryPanel.tsx`, `components/VersionHistoryTable.tsx`

##### Sub-Subtask 1.2.1: Version Badge Component

- **Description**: Create reusable badge component to display version number (e.g., "v2.3", "v1.0 (current)")
- **Why**: Consistent visual representation of versions across UI
- **Props**: `versionNumber: number`, `isCurrent: boolean`, `isRollback: boolean`
- **Acceptance**: Badge styled differently for current vs historical vs rollback versions

#### Subtask 1.3: Version Comparison View (Diff Display)

- **Description**: Create side-by-side diff viewer showing additions/deletions/unchanged content between two versions
- **Components**:
  - `VersionCompareDialog` (new) → Full-screen modal for diff view
  - `DiffViewer` (new) → Syntax-highlighted diff display (use library like `react-diff-viewer`)
- **Features**:
  - Select two versions from dropdown
  - Highlight: green for added, red for deleted, neutral for unchanged
  - Line numbers and expand/collapse unchanged sections
- **Uses**: Backend Task 2.10 (POST /api/jobs/{jobId}/versions/compare)
- **Tested By**: Testing Task 5.3
- **Acceptance**: Diff renders in <2 seconds for 50KB JDs; correct highlighting; readable formatting
- **Files**: `components/VersionCompareDialog.tsx`, `components/DiffViewer.tsx`

##### Sub-Subtask 1.3.1: "View as of Date" Feature

- **Description**: Add date picker to version history allowing HR to see JD as it appeared on specific date
- **Why**: Compliance requirement: "What did candidates see when they applied on [date]?"
- **Uses**: Backend Task 2.12 (GET /api/jobs/{jobId}/versions/as-of-date)
- **Acceptance**: Date picker renders; selecting date displays correct version content; handles edge cases

#### Subtask 1.4: Rollback & Version Activation UI

- **Description**: Add action buttons in version history to rollback or activate previous versions
- **Components**:
  - `RollbackConfirmDialog` (new) → Confirmation modal with reason field
  - `VersionActionsMenu` (new) → Dropdown menu per version row with "Revert to This Version", "Activate This Version", "View Content"
- **Uses**: Backend Task 2.11 (POST /api/jobs/{jobId}/versions/rollback)
- **Tested By**: Testing Task 5.4
- **Acceptance**: Rollback completes in <5 seconds; UI updates to show new version; confirmation required before rollback
- **Files**: `components/RollbackConfirmDialog.tsx`, `components/VersionActionsMenu.tsx`

##### Sub-Subtask 1.4.1: Undo Rollback Feature

- **Description**: Add "Undo Rollback" button that appears temporarily after a rollback action
- **Why**: Safety feature for accidental rollbacks
- **Logic**: Button visible for 30 seconds after rollback; clicking reverts to version before rollback
- **Acceptance**: Undo button appears after rollback; clicking reverts successfully; button disappears after timeout

#### Subtask 1.5: Candidate Results with JD Version Attribution

- **Description**: Modify candidate results/ranking views to display which JD version was used for assessment
- **Components**:
  - `CandidateResultCard` (existing) → Add version badge next to result score
  - `CandidateRankingTable` (existing) → Add "JD Version" column
- **Uses**: Existing interview session API (includes `jd_version_id` field)
- **Tested By**: Testing Task 5.5
- **Acceptance**: All candidate results show JD version used; version is clickable to view version content
- **Files**: `components/CandidateResultCard.tsx`, `pages/CandidateRankingPage.tsx`

---

## Testing

### Main Task 1: Backend Unit & Integration Tests

**Description**: Test JD versioning service layer and API endpoints with comprehensive scenarios including edge cases.

#### Subtask 1.1: Test Initial Version Creation on Job Creation

- **Description**: Unit test for `JobDescriptionService.createInitialVersion()`
- **Tests**: Backend Task 2.1
- **Test Type**: Unit test
- **Scenarios**:
  - New job creates version 1.0 with correct content
  - `JobDescription.current_version_id` points to new version
  - `JobDescriptionChangeLog` entry created with change_type='create'
  - Transaction rolls back if any step fails
- **Acceptance**: All tests pass; 100% coverage of createInitialVersion method

#### Subtask 1.2: Test Automatic Version Creation on JD Update

- **Description**: Integration test for `JobDescriptionService.updateAndCreateVersion()`
- **Tests**: Backend Task 2.2
- **Test Type**: Integration test (database)
- **Scenarios**:
  - Update increments version number (1.0 → 2.0)
  - Previous version marked inactive
  - Content hash comparison prevents duplicate versions
  - Concurrent edit detection (optimistic locking)
  - Change summary saved correctly
  - Transaction atomicity (all-or-nothing)
- **Acceptance**: All tests pass; version count increases; old version preserved unchanged

#### Subtask 1.3: Test Version History Query

- **Description**: Integration test for `JobDescriptionService.getVersionHistory()`
- **Tests**: Backend Task 2.3
- **Test Type**: Integration test (database + joins)
- **Scenarios**:
  - Returns all versions in descending order
  - Includes creator name (join with users table)
  - Includes candidate count (join with interview_sessions)
  - Multi-tenant isolation (company A can't see company B versions)
  - Performance: <3 seconds for 50+ versions
- **Acceptance**: All tests pass; query optimized; correct data returned

#### Subtask 1.4: Test Version Content Retrieval

- **Description**: Unit test for `JobDescriptionService.getVersionContent()`
- **Tests**: Backend Task 2.4
- **Test Type**: Unit test (with S3 mock for large JDs)
- **Scenarios**:
  - Retrieves inline content for small JDs
  - Fetches from S3 for large JDs (mock S3 call)
  - Multi-tenant check prevents cross-company access
  - Returns 404 for non-existent version
- **Acceptance**: All tests pass; S3 mock validated; multi-tenant enforced

#### Subtask 1.5: Test Version Diff Generation

- **Description**: Unit test for `JobDescriptionService.compareVersions()`
- **Tests**: Backend Task 2.5
- **Test Type**: Unit test
- **Scenarios**:
  - Correct diff for added text (highlighted green)
  - Correct diff for deleted text (highlighted red)
  - Unchanged sections preserved
  - Handles large diffs (50KB)
  - Performance: <2 seconds for 50KB JDs
- **Acceptance**: All tests pass; diff accuracy validated; performance target met

#### Subtask 1.6: Test Rollback Functionality

- **Description**: Integration test for `JobDescriptionService.rollbackToVersion()`
- **Tests**: Backend Task 2.6
- **Test Type**: Integration test (database)
- **Scenarios**:
  - Rollback creates new version with old content
  - `is_rollback` flag set correctly
  - `rollback_from_version_id` references target version
  - Audit log created with change_type='rollback'
  - Permission check enforced
  - Previous interview sessions unchanged (still reference old version IDs)
- **Acceptance**: All tests pass; rollback completes in <5 seconds; audit trail accurate

#### Subtask 1.7: Test Temporal Query (As of Date)

- **Description**: Integration test for `JobDescriptionService.getVersionAsOfDate()`
- **Tests**: Backend Task 2.7
- **Test Type**: Integration test (database)
- **Scenarios**:
  - Returns correct version for past date
  - Returns 404 for date before job creation
  - Handles edge cases (date = version creation timestamp)
  - Multi-tenant isolation enforced
- **Acceptance**: All tests pass; correct version returned for various dates

---

### Main Task 2: API Endpoint Tests

**Description**: Test all JD versioning API endpoints with request/response validation, authorization, and error handling.

#### Subtask 2.1: Test GET /api/jobs/{jobId}/versions

- **Description**: API integration test for version history endpoint
- **Tests**: Backend Task 2.8, Frontend Task 1.2
- **Test Type**: API integration test
- **Scenarios**:
  - Returns version history for authorized user
  - Returns 401 for unauthenticated request
  - Returns 403 for user without `job.version.view` permission
  - Multi-tenant isolation (user A can't access company B's versions)
  - Query param `includeDeleted` filters soft-deleted versions
- **Acceptance**: All tests pass; authorization enforced; response schema validated

#### Subtask 2.2: Test GET /api/jobs/{jobId}/versions/{versionId}

- **Description**: API integration test for specific version retrieval
- **Tests**: Backend Task 2.9, Frontend Task 1.3
- **Test Type**: API integration test
- **Scenarios**:
  - Returns version content for authorized user
  - Returns 404 for non-existent version
  - Returns 403 for unauthorized access
  - Handles large JDs (S3 fetch)
- **Acceptance**: All tests pass; content retrieved correctly; performance <2 seconds

#### Subtask 2.3: Test POST /api/jobs/{jobId}/versions/compare

- **Description**: API integration test for version comparison endpoint
- **Tests**: Backend Task 2.10, Frontend Task 1.3
- **Test Type**: API integration test
- **Scenarios**:
  - Returns diff for two valid versions
  - Returns 400 for invalid version IDs
  - Returns 403 for unauthorized access
  - Performance: <2 seconds for 50KB JDs
- **Acceptance**: All tests pass; diff accurate; response time meets target

#### Subtask 2.4: Test POST /api/jobs/{jobId}/versions/rollback

- **Description**: API integration test for rollback endpoint
- **Tests**: Backend Task 2.11, Frontend Task 1.4
- **Test Type**: API integration test
- **Scenarios**:
  - Rollback creates new version with old content
  - Returns 403 for user without `job.edit` permission
  - Returns 400 for invalid target version
  - Audit log created correctly
- **Acceptance**: All tests pass; rollback completes in <5 seconds; new version returned

#### Subtask 2.5: Test GET /api/jobs/{jobId}/versions/as-of-date

- **Description**: API integration test for temporal query endpoint
- **Tests**: Backend Task 2.12, Frontend Task 1.3
- **Test Type**: API integration test
- **Scenarios**:
  - Returns correct version for valid date
  - Returns 404 for date before job creation
  - Returns 400 for invalid date format
  - Multi-tenant isolation enforced
- **Acceptance**: All tests pass; correct version returned; edge cases handled

#### Subtask 2.6: Test PUT /api/jobs/{jobId} with Version Creation

- **Description**: API integration test for job update triggering version creation
- **Tests**: Backend Task 2.13, Frontend Task 1.1
- **Test Type**: API integration test
- **Scenarios**:
  - Update with JD content change creates new version
  - Update without JD change does not create version (same content_hash)
  - Change summary saved when provided
  - Returns updated job with new `current_version_id` and `version_count`
  - Returns 409 for concurrent edit conflict
- **Acceptance**: All tests pass; version creation triggered correctly; concurrent edits handled

---

### Main Task 3: Interview Session JD Locking Tests

**Description**: Test JD version capture at interview session start and immutability during interview.

#### Subtask 3.1: Test JD Version Capture at Session Start

- **Description**: Integration test for `InterviewSessionService.startSession()`
- **Tests**: Backend Task 3.1
- **Test Type**: Integration test (database)
- **Scenarios**:
  - New session captures current JD version ID
  - `jd_locked_at` timestamp set correctly
  - Session references correct version even if JD updated immediately after
  - 100% of sessions have non-null `jd_version_id`
- **Acceptance**: All tests pass; version captured atomically; no null references

#### Subtask 3.2: Test Locked JD Version Used for AI Operations

- **Description**: Integration test for `InterviewSessionService.getLockedJDVersion()`
- **Tests**: Backend Task 3.2
- **Test Type**: Integration test (with AI service mock)
- **Scenarios**:
  - AI resume scoring uses locked version (not current version)
  - AI question generation uses locked version
  - Version unchanged even if HR updates JD mid-interview
  - Two candidates interviewing on same job with different JD versions get different questions
- **Acceptance**: All tests pass; AI operations use correct locked version; immutability verified

#### Subtask 3.3: Test POST /api/interviews/start API

- **Description**: API integration test for interview start with JD version locking
- **Tests**: Backend Task 3.3, Frontend (Feature 001)
- **Test Type**: API integration test
- **Scenarios**:
  - Session created with JD version locked
  - Returns session metadata including `jd_version_id` and `jd_locked_at`
  - Requires candidate email verification (Feature 001 integration)
  - Returns 401 for unverified candidate
- **Acceptance**: All tests pass; JD version locked at session start; metadata returned correctly

---

### Main Task 4: Feature Integration Tests

**Description**: Test integration with Features 003, 004, 005 (Status Tracking, Permissions, Token Tracking).

#### Subtask 4.1: Test JD Version in Candidate Status History

- **Description**: Integration test for Feature 003 status tracking with JD version reference
- **Tests**: Backend Task 4.1, Frontend Task 1.5
- **Test Type**: Integration test (Feature 003 + 009)
- **Scenarios**:
  - Status change populates `jd_version_id` from interview session
  - Status history includes correct JD version for each change
  - Handles edge case: manual status change without interview (nullable `jd_version_id`)
  - Audit trail shows which JD version was active during status assignment
- **Acceptance**: All tests pass; 100% of interview-based status changes include JD version

#### Subtask 4.2: Test Permission Enforcement for JD Version Operations

- **Description**: Integration test for Feature 004 RBAC permission checks
- **Tests**: Backend Task 4.2
- **Test Type**: Integration test (Feature 004 + 009)
- **Scenarios**:
  - User with `job.edit` can create/update/rollback versions
  - User without `job.edit` receives 403 on update/rollback
  - User with `job.version.view` can view history and specific versions
  - User without `job.version.view` receives 403 on version queries
  - Permission checks logged in audit trail
- **Acceptance**: All tests pass; permissions enforced at API and service layers; audit logs accurate

#### Subtask 4.3: Test Token Allocation to JD Versions

- **Description**: Integration test for Feature 005 token tracking per JD version
- **Tests**: Backend Task 4.3
- **Test Type**: Integration test (Feature 005 + 009)
- **Scenarios**:
  - AI resume scoring logs token usage with `jd_version_id`
  - AI question generation logs token usage with `jd_version_id`
  - Cost-per-version report aggregates tokens correctly
  - Multiple versions of same job have separate token counts
- **Acceptance**: All tests pass; token usage attributable to specific JD versions; cost reporting accurate

---

### Main Task 5: Frontend UI Tests

**Description**: Test frontend components for JD editing, version history, diff view, and rollback with user interactions.

#### Subtask 5.1: Test Job Description Editor with Version Creation

- **Description**: E2E test for editing JD and creating version with optional summary
- **Tests**: Frontend Task 1.1, Backend Task 2.13
- **Test Type**: E2E test (Playwright/Cypress)
- **Scenarios**:
  - HR edits JD, clicks "Save", version created in <5 seconds
  - HR edits JD, clicks "Save with Summary", modal opens, summary saved
  - Concurrent edit conflict notification displayed (409 response)
  - UI updates with new version number and version count
- **Acceptance**: All tests pass; version creation smooth; UI responsive; conflict notification works

#### Subtask 5.2: Test Version History View

- **Description**: E2E test for version history panel and table
- **Tests**: Frontend Task 1.2, Backend Task 2.8
- **Test Type**: E2E test
- **Scenarios**:
  - Version history panel loads in <3 seconds for 50+ versions
  - Table displays all columns: Version, Date, Author, Summary, Candidates
  - Candidate count badge shows correct number per version
  - Sortable/filterable by date, author, version number
- **Acceptance**: All tests pass; history loads fast; data accurate; UI intuitive

#### Subtask 5.3: Test Version Comparison View (Diff Display)

- **Description**: E2E test for side-by-side diff viewer
- **Tests**: Frontend Task 1.3, Backend Task 2.10
- **Test Type**: E2E test
- **Scenarios**:
  - User selects two versions, diff displays in <2 seconds
  - Added text highlighted green, deleted text red, unchanged neutral
  - Line numbers displayed, unchanged sections collapsible
  - "View as of Date" picker shows correct version for selected date
- **Acceptance**: All tests pass; diff accurate; highlighting correct; performance meets target

#### Subtask 5.4: Test Rollback & Version Activation UI

- **Description**: E2E test for rollback and activate previous version actions
- **Tests**: Frontend Task 1.4, Backend Task 2.11
- **Test Type**: E2E test
- **Scenarios**:
  - User clicks "Revert to This Version", confirmation modal opens
  - User enters reason, confirms, rollback completes in <5 seconds
  - UI updates to show new version (rollback version)
  - "Undo Rollback" button appears, clicking reverts to previous version
  - "Activate This Version" creates no new version, just activates
- **Acceptance**: All tests pass; rollback smooth; confirmation required; undo works; UI updates correctly

#### Subtask 5.5: Test Candidate Results with JD Version Attribution

- **Description**: E2E test for displaying JD version in candidate results
- **Tests**: Frontend Task 1.5
- **Test Type**: E2E test
- **Scenarios**:
  - Candidate result card displays version badge (e.g., "Evaluated using JD v2.0")
  - Ranking table includes "JD Version" column
  - Version badge is clickable, opens version content view
  - Grouping by version in ranking report works
- **Acceptance**: All tests pass; version attribution clear; UI consistent; clickable links work

---

## Task Summary

- **Total Main Tasks**: 17
  - Database: 1
  - Backend: 4
  - Frontend: 1
  - Testing: 5
- **Total Subtasks**: 45
- **Total Sub-Subtasks**: 7
- **Priority Distribution**:
  - **P1 (MVP)**: 10 main tasks (Database schema, core versioning service, interview locking, API endpoints, basic UI)
  - **P2 (Enhancement)**: 7 main tasks (diff view, rollback UI, feature integrations, comprehensive testing)

---

## Dependencies & Sequencing

### Phase 1 (Must complete first) - Foundation

**Duration**: 2-3 weeks

1. **Database Task 1.1-1.5**: Create all schema migrations (JobDescriptionVersion, JobDescriptionChangeLog, modify existing tables)
2. **Backend Task 2.1**: Implement initial version creation on job creation
3. **Backend Task 2.2**: Implement automatic version creation on JD update
4. **Backend Task 2.3-2.4**: Implement version history query and content retrieval
5. **Backend Task 2.8-2.9, 2.13**: Implement basic API endpoints (GET versions, GET specific version, PUT job with versioning)

**Blocking**: All other work depends on schema and basic version creation logic.

---

### Phase 2 (Can parallelize) - Core Features

**Duration**: 3-4 weeks

**Track A - Interview Integration** (can parallelize with Track B):

1. **Backend Task 3.1-3.2**: Implement JD version locking at interview session start
2. **Backend Task 3.3**: Implement/modify interview start API
3. **Testing Task 3.1-3.3**: Test interview session JD locking

**Track B - Advanced Versioning Features** (can parallelize with Track A):

1. **Backend Task 2.5**: Implement diff generation
2. **Backend Task 2.6**: Implement rollback logic
3. **Backend Task 2.7**: Implement temporal query (as of date)
4. **Backend Task 2.10-2.12**: Implement advanced API endpoints (compare, rollback, as-of-date)
5. **Testing Task 1.1-1.7**: Backend unit and integration tests
6. **Testing Task 2.1-2.6**: API endpoint tests

**Track C - Frontend Development** (can start after Backend Task 2.8-2.9 complete):

1. **Frontend Task 1.1**: Enhanced job description editor
2. **Frontend Task 1.2**: Version history view
3. **Frontend Task 1.3**: Version comparison view (diff display)
4. **Frontend Task 1.4**: Rollback and activation UI
5. **Frontend Task 1.5**: Candidate results with JD version attribution

---

### Phase 3 (Final integration) - Feature Integration & Testing

**Duration**: 2 weeks

1. **Backend Task 4.1**: Integrate with Feature 003 status tracking
2. **Backend Task 4.2**: Integrate with Feature 004 permissions
3. **Backend Task 4.3**: Integrate with Feature 005 token tracking
4. **Testing Task 4.1-4.3**: Feature integration tests
5. **Testing Task 5.1-5.5**: Frontend E2E tests
6. **Final QA**: End-to-end testing across all features, performance validation, compliance audit

---

## Notes

### Technical Considerations

1. **Content Storage Strategy**: Small JDs (<100KB) stored inline in `JobDescriptionVersion.content`; large JDs (>100KB) stored in S3 with reference key. Decision point: implement S3 integration in Phase 1 or defer to Phase 2 optimization.

2. **Version Number Scheme**: Auto-increment per job (Job 1: 1.0, 2.0, 3.0; Job 2: 1.0, 2.0). Alternative: Semantic versioning (1.0, 1.1, 2.0 for major changes). Current decision: simple auto-increment.

3. **Diff Algorithm**: Use Myers diff algorithm via `diff-match-patch` library or similar. Consider field-level diff for structured JDs (future enhancement: store change_metadata JSON in JobDescriptionChangeLog).

4. **Performance Optimization**:
   - Index `(job_id, version_number)` for version history queries
   - Index `(company_id, created_at)` for audit/compliance queries
   - Index `is_active` for current version lookups
   - Cache current version content in Redis to avoid DB hits on every interview session start

5. **Multi-Tenant Isolation**: All queries MUST filter by `company_id`. Add database row-level security policies (Postgres RLS) or ORM-level global filters to prevent cross-company data leakage.

6. **Concurrent Edit Handling**: Last-write-wins strategy with optimistic locking. Compare client's `current_version_id` with server's; if mismatch, return 409 Conflict. Future enhancement: implement collaborative editing or lock-based editing.

### Open Questions

1. **Q**: Should version history be exportable for compliance (e.g., PDF report of all versions)? → A: Defer to Phase 3; add export API if required by compliance team.

2. **Q**: How to handle very large version histories (1000+ versions)? → A: Implement pagination in version history API; lazy-load in UI; archive old versions (>2 years) to cold storage.

3. **Q**: Should Super Admin be able to delete individual versions (not just entire job)? → A: Yes, add delete API in Feature 006 God Mode integration; requires special permission and audit logging.

4. **Q**: Should we support version branching (e.g., create version 2.1a and 2.1b for A/B testing)? → A: Defer to future enhancement; current linear version chain sufficient for MVP.

5. **Q**: How to handle JD translations (multiple languages)? → A: Out of scope for Phase 2; defer to internationalization feature (potential Feature 011).

### Assumptions Validation

- **Assumption**: Automatic version creation on every save. **Validation**: Confirmed with stakeholders; simpler than manual "save as version" workflow.
- **Assumption**: Indefinite retention of all versions. **Validation**: Required for compliance; manual deletion only via Super Admin.
- **Assumption**: Content hash prevents duplicate versions. **Validation**: Edge case: whitespace-only changes create duplicate versions; decision: store normalized content (trim whitespace) before hashing.
- **Assumption**: Last-write-wins for concurrent edits. **Validation**: Acceptable for MVP; implement lock-based editing if conflicts become frequent.

### Risk Mitigation

- **Risk**: Large JD content (>100KB) slows down queries. **Mitigation**: Implement S3 storage with content reference; benchmark query performance before production.
- **Risk**: Version history UI slow with 100+ versions. **Mitigation**: Implement pagination and virtual scrolling in frontend; limit initial load to 20 versions.
- **Risk**: Multi-tenant isolation failure (cross-company data leakage). **Mitigation**: Add integration tests specifically for multi-tenant scenarios; code review all queries for `company_id` filtering.
- **Risk**: AI operations use wrong JD version (cache staleness). **Mitigation**: Always fetch locked version from DB at interview session start; avoid caching JD content in interview session record.
