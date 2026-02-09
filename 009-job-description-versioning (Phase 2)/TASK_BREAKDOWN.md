# Task Breakdown: Job Description Versioning & History

**Source**: [009-job-description-versioning (Phase 2)/spec.md](./spec.md)  
**Generated**: 2026-02-02  
**Updated**: 2026-02-02 (Aligned with simplified skill format)  
**Status**: Planning

---

## Overview

This feature implements comprehensive job description versioning to track JD changes, lock versions during interview sessions, support rollback, and provide audit trails for compliance and fair candidate evaluation.

**Key Capabilities**:

- Automatic version creation on JD updates
- Version locking at interview session start
- Version history view with side-by-side diff comparison
- Rollback and version management with audit trails
- Multi-tenant isolation and permission enforcement

**Cross-Referencing System**:

This task breakdown uses **3 essential linkage types** to show relationships:

- **Depends On**: What must exist first (e.g., Database 1.1)
- **Related**: Connected tasks in other areas (e.g., Frontend 5.1, Backend 2.3)
- **Test Coverage**: Which test validates this task (e.g., Testing 6.1)

---

## Database

### Main Task 1: JD Versioning Schema Design

**Description**: Design and implement database schema for storing job description versions, change logs, and version relationships.

**Spec Reference**: FR-001, FR-008, FR-009

#### Subtask 1.1: JobDescriptionVersion Table

- **Spec Reference**: Entity: JobDescriptionVersion, FR-001, FR-008
- **Description**: Create migration for `job_description_versions` table
- **Columns**:
  - `id` (PK, UUID)
  - `job_id` (FK to jobs, indexed)
  - `company_id` (FK to companies, indexed)
  - `version_number` (INT, auto-increment per job)
  - `content` (LONGTEXT / S3 reference for large JDs)
  - `content_hash` (VARCHAR, SHA256)
  - `created_at`, `created_by_user_id`
  - `change_summary` (TEXT, nullable)
  - `is_active`, `is_rollback`, `rollback_from_version_id`, `is_deleted`
- **Indexes**: `(job_id, version_number)`, `(company_id, created_at)`, `is_active`, `content_hash`
- **Related**: Backend 2.1, Backend 2.2, Backend 2.3, Backend 2.4, Backend 2.5, Backend 2.6, Backend 2.7
- **Test Coverage**: Testing 6.1, Testing 6.2, Testing 6.3
- **Acceptance**: Migration runs successfully; constraints validated

#### Subtask 1.2: JobDescriptionChangeLog Table

- **Spec Reference**: Entity: JobDescriptionChangeLog, FR-008
- **Description**: Create migration for audit table tracking version transitions
- **Columns**:
  - `id` (PK, UUID)
  - `job_id`, `company_id` (FKs, indexed)
  - `old_version_id`, `new_version_id` (FKs)
  - `change_type` (ENUM: create, update, rollback, delete)
  - `actor_user_id`, `timestamp`, `reason`, `change_metadata` (JSON)
- **Indexes**: `(job_id, timestamp)`, `(company_id, timestamp)`, `actor_user_id`
- **Related**: Backend 2.1, Backend 2.2, Backend 2.6
- **Test Coverage**: Testing 6.1, Testing 6.2, Testing 6.6
- **Acceptance**: Migration runs; foreign keys validated

#### Subtask 1.3: Modify JobDescription Table

- **Spec Reference**: Entity: JobDescription, FR-001
- **Description**: Add version reference columns to existing `job_descriptions` table
- **New Columns**: `current_version_id` (FK), `version_count` (INT, default 1)
- **Migration Strategy**: Add nullable → backfill → add NOT NULL constraint
- **Related**: Backend 2.1, Backend 2.2, Backend 2.13
- **Test Coverage**: Testing 6.1, Testing 7.6
- **Acceptance**: Existing jobs migrated with version 1.0

#### Subtask 1.4: Modify InterviewSession Table

- **Spec Reference**: Entity: InterviewSession, FR-002, FR-009
- **Description**: Add columns to capture JD version at session start
- **New Columns**: `jd_version_id` (FK, NOT NULL), `jd_locked_at` (TIMESTAMP)
- **Migration Strategy**: Add nullable → backfill from current JD → add NOT NULL
- **Related**: Backend 3.1, Backend 3.2, Backend 3.3
- **Test Coverage**: Testing 8.1, Testing 8.2, Testing 8.3
- **Acceptance**: All sessions reference valid JD version

#### Subtask 1.5: Modify CandidateStatusHistory Table

- **Spec Reference**: Entity: CandidateStatusHistory, FR-010
- **Description**: Track which JD version was active during status changes
- **New Columns**: `jd_version_id` (FK, nullable)
- **Related**: Backend 4.1
- **Test Coverage**: Testing 8.4
- **Acceptance**: Future status changes auto-populate JD version

---

## Backend

### Main Task 2: JD Versioning Service & APIs

**Description**: Core service layer and API endpoints for managing JD versions, including automatic version creation, history queries, diff generation, rollback, and audit logging.

**Spec Reference**: FR-001, FR-004, FR-007, FR-008

#### Subtask 2.1: Create Initial Version on Job Creation

- **Spec Reference**: User Story 1, FR-001
- **Description**: Automatically create version 1.0 when a new job is created
- **Methods**: `JobDescriptionService.createInitialVersion(jobId, content, userId)`
- **Logic**: Calculate content hash → create version record → update job FK → create changelog
- **Depends On**: Database 1.1, Database 1.2, Database 1.3
- **Related**: Frontend 5.1
- **Test Coverage**: Testing 6.1
- **Acceptance**: New jobs have version 1.0; audit log created
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.2: Update JD and Create New Version

- **Spec Reference**: User Story 1, FR-001, FR-008, FR-012
- **Description**: Atomic operation to save JD changes and create new version automatically
- **Methods**: `JobDescriptionService.updateAndCreateVersion(jobId, content, summary, userId)`
- **Logic**:
  - Check permission (`job.edit`)
  - Compare content hash (skip if unchanged)
  - Transaction: mark current inactive → create new version → update job → create changelog
  - Concurrent edit detection (optimistic locking with 409 Conflict)
- **Depends On**: Database 1.1, Database 1.2, Database 1.3
- **Related**: Frontend 5.1
- **Test Coverage**: Testing 6.2, Testing 7.6
- **Acceptance**: Version auto-increments; old version preserved; <5 seconds
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.3: Query Version History

- **Spec Reference**: User Story 3, FR-004
- **Description**: Retrieve all versions for a job with metadata
- **Methods**: `JobDescriptionService.getVersionHistory(jobId, companyId)`
- **Return**: `{ version_number, created_at, created_by_name, change_summary, is_active, candidate_count }`
- **Query**: Join versions → users → interview_sessions; filter by company_id
- **Depends On**: Database 1.1, Database 1.4
- **Related**: Frontend 5.2
- **Test Coverage**: Testing 6.3, Testing 7.1
- **Acceptance**: Returns history in <3 seconds for 50+ versions
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.4: Retrieve Specific Version Content

- **Spec Reference**: User Story 3, FR-004
- **Description**: Fetch full content of a specific version
- **Methods**: `JobDescriptionService.getVersionContent(versionId, companyId)`
- **Logic**: Validate company access; fetch content (inline or S3)
- **Depends On**: Database 1.1
- **Related**: Frontend 5.3, Backend 3.2
- **Test Coverage**: Testing 6.4, Testing 7.2
- **Acceptance**: Content retrieval in <2 seconds; S3 fetch works for large JDs
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.5: Compare Two Versions (Diff Generation)

- **Spec Reference**: User Story 3, FR-005
- **Description**: Generate side-by-side diff with additions/deletions
- **Methods**: `JobDescriptionService.compareVersions(versionId1, versionId2, companyId)`
- **Algorithm**: Myers diff (use `diff-match-patch` library)
- **Return**: `{ added: [], deleted: [], unchanged: [] }`
- **Depends On**: Database 1.1, Backend 2.4
- **Related**: Frontend 5.3
- **Test Coverage**: Testing 6.5, Testing 7.3
- **Acceptance**: Diff in <2 seconds for 50KB JDs
- **Files**: `services/JobDescriptionService.js`, `utils/diffGenerator.js`

#### Subtask 2.6: Rollback to Previous Version

- **Spec Reference**: User Story 4, FR-007, FR-008
- **Description**: Revert JD by creating new version with old content
- **Methods**: `JobDescriptionService.rollbackToVersion(jobId, targetVersionId, userId)`
- **Logic**: Fetch target content → create new version (is_rollback=true) → update job → changelog
- **Depends On**: Database 1.1, Database 1.2, Database 1.3, Backend 2.4
- **Related**: Frontend 5.4
- **Test Coverage**: Testing 6.6, Testing 7.4
- **Acceptance**: Rollback in <5 seconds; audit log shows rollback
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.7: View JD "As of Date"

- **Spec Reference**: User Story 3, FR-006
- **Description**: Temporal query to get JD version active on specific date
- **Methods**: `JobDescriptionService.getVersionAsOfDate(jobId, date, companyId)`
- **Query**: `WHERE job_id=? AND created_at <= ? ORDER BY created_at DESC LIMIT 1`
- **Depends On**: Database 1.1
- **Related**: Frontend 5.3
- **Test Coverage**: Testing 6.7, Testing 7.5
- **Acceptance**: Returns correct version; handles edge cases
- **Files**: `services/JobDescriptionService.js`

#### Subtask 2.8: API - GET /api/jobs/{jobId}/versions

- **Spec Reference**: User Story 3, FR-004, FR-012
- **Description**: Retrieve version history
- **Request**: Headers: Authorization (JWT with company_id)
- **Response**: `{ versions: [...] }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.3
- **Related**: Frontend 5.2
- **Test Coverage**: Testing 7.1
- **Acceptance**: Returns history in <3 seconds; 401/403 enforced
- **Files**: `controllers/JobDescriptionController.js`

#### Subtask 2.9: API - GET /api/jobs/{jobId}/versions/{versionId}

- **Spec Reference**: User Story 3, FR-004
- **Description**: Retrieve specific version content
- **Request**: Headers: Authorization
- **Response**: `{ version: { id, version_number, content, created_at, ... } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.4
- **Related**: Frontend 5.3
- **Test Coverage**: Testing 7.2
- **Acceptance**: Returns content in <2 seconds
- **Files**: `controllers/JobDescriptionController.js`

#### Subtask 2.10: API - POST /api/jobs/{jobId}/versions/compare

- **Spec Reference**: User Story 3, FR-005
- **Description**: Compare two versions
- **Request**: `{ version1_id, version2_id }`
- **Response**: `{ diff: { added, deleted, unchanged } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.5
- **Related**: Frontend 5.3
- **Test Coverage**: Testing 7.3
- **Acceptance**: Diff in <2 seconds; correct highlighting
- **Files**: `controllers/JobDescriptionController.js`

#### Subtask 2.11: API - POST /api/jobs/{jobId}/versions/rollback

- **Spec Reference**: User Story 4, FR-007, FR-012
- **Description**: Rollback to previous version
- **Request**: `{ target_version_id, reason? }`
- **Response**: `{ new_version: { id, version_number, content } }`
- **Authorization**: Requires `job.edit`
- **Depends On**: Backend 2.6
- **Related**: Frontend 5.4
- **Test Coverage**: Testing 7.4
- **Acceptance**: Rollback in <5 seconds; audit created
- **Files**: `controllers/JobDescriptionController.js`

#### Subtask 2.12: API - GET /api/jobs/{jobId}/versions/as-of-date

- **Spec Reference**: User Story 3, FR-006
- **Description**: Get version active on specific date
- **Request**: Query param `date` (ISO 8601)
- **Response**: `{ version: { ... } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.7
- **Related**: Frontend 5.3
- **Test Coverage**: Testing 7.5
- **Acceptance**: Returns correct version; 404 if date before job creation
- **Files**: `controllers/JobDescriptionController.js`

#### Subtask 2.13: API - PUT /api/jobs/{jobId} (Modify Existing)

- **Spec Reference**: User Story 1, FR-001, FR-012
- **Description**: Trigger automatic version creation when JD content changes
- **Logic**: If JD fields changed, call `updateAndCreateVersion()`
- **Authorization**: Requires `job.edit`
- **Depends On**: Backend 2.2
- **Related**: Frontend 5.1
- **Test Coverage**: Testing 7.6
- **Acceptance**: Creates version only when content changed; returns updated job
- **Files**: `controllers/JobDescriptionController.js`

- **Spec Reference**: User Story 3, FR-005
- **Description**: Compare two versions
- **Request**: `{ version1_id, version2_id }`
- **Response**: `{ diff: { added, deleted, unchanged } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.5
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 6.3
- **Acceptance**: Diff in <2 seconds; correct highlighting

#### Subtask 2.11: API - POST /api/jobs/{jobId}/versions/rollback

- **Spec Reference**: User Story 4, FR-007, FR-012
- **Description**: Rollback to previous version
- **Request**: `{ target_version_id, reason? }`
- **Response**: `{ new_version: { id, version_number, content } }`
- **Authorization**: Requires `job.edit`
- **Depends On**: Backend 2.6
- **Related**: Frontend 4.4
- **Test Coverage**: Testing 6.4
- **Acceptance**: Rollback in <5 seconds; audit created

#### Subtask 2.12: API - GET /api/jobs/{jobId}/versions/as-of-date

- **Spec Reference**: User Story 3, FR-006
- **Description**: Get version active on specific date
- **Request**: Query param `date` (ISO 8601)
- **Response**: `{ version: { ... } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.7
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 6.5
- **Acceptance**: Returns correct version; 404 if date before job creation

#### Subtask 2.13: API - PUT /api/jobs/{jobId} (Modify Existing)

- **Spec Reference**: User Story 1, FR-001, FR-012
- **Description**: Trigger automatic version creation when JD content changes
- **Logic**: If JD fields changed, call `updateAndCreateVersion()`
- **Authorization**: Requires `job.edit`
- **Depends On**: Backend 2.2
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 6.6
- **Acceptance**: Creates version only when content changed; returns updated job

---

### Main Task 3: Interview Session JD Locking

**Description**: Capture and lock JD version when candidate starts interview session.

**Spec Reference**: FR-002, FR-003, FR-009

#### Subtask 3.1: Capture JD Version at Session Start

- **Spec Reference**: User Story 2, FR-002, FR-009
- **Description**: Store current JD version ID when candidate opens interview
- **Methods**: `InterviewSessionService.startSession(candidateId, jobId)`
- **Logic**: Fetch current version → create session with `jd_version_id` and `jd_locked_at`
- **Depends On**: Database 1.3, Database 1.4
- **Related**: Frontend 5.5, Backend 3.2
- **Test Coverage**: Testing 8.1, Testing 8.3
- **Acceptance**: 100% of sessions have non-null jd_version_id
- **Files**: `services/InterviewSessionService.js`

#### Subtask 3.2: Use Locked Version for AI Operations

- **Spec Reference**: User Story 2, FR-003
- **Description**: Pass locked JD version to AI services (not current version)
- **Methods**: `InterviewSessionService.getLockedJDVersion(sessionId)`
- **Logic**: Fetch session → get version content → return to AI caller
- **Depends On**: Database 1.4, Backend 2.4
- **Related**: Feature 001 AI integration
- **Test Coverage**: Testing 8.2
- **Acceptance**: AI uses locked version; immutable during interview
- **Files**: `services/InterviewSessionService.js`

#### Subtask 3.3: API - POST /api/interviews/start (Modify Existing)

- **Spec Reference**: User Story 2, FR-002
- **Description**: Capture JD version at interview start
- **Request**: `{ candidate_id, job_id }`
- **Response**: Include `jd_version_id`, `jd_locked_at`
- **Depends On**: Backend 3.1
- **Related**: Feature 001 workflow
- **Test Coverage**: Testing 8.3
- **Acceptance**: Session created with locked version
- **Files**: `controllers/InterviewController.js`

---

### Main Task 4: Feature Integration (P2)

**Description**: Integrate with Features 003, 004, 005.

**Spec Reference**: FR-010, FR-012

#### Subtask 4.1: Feature 003 - Status History Integration

- **Spec Reference**: FR-010, Feature 003 dependency
- **Description**: Populate JD version when candidate status changes
- **Methods**: Modify `CandidateService.updateStatus()` to capture jd_version_id from session
- **Depends On**: Database 1.5, Feature 003
- **Test Coverage**: Testing 8.4
- **Acceptance**: All status changes include JD version reference
- **Files**: `services/CandidateService.js`

#### Subtask 4.2: Feature 004 - Permission Enforcement

- **Spec Reference**: FR-012, Feature 004 dependency
- **Description**: Integrate RBAC for JD version operations
- **Permissions**: `job.edit` (create/update/rollback), `job.version.view` (history), `job.history.view` (audit logs)
- **Depends On**: Feature 004 permissions system
- **Test Coverage**: Testing 8.5
- **Acceptance**: 403 for unauthorized; permissions logged
- **Files**: `middleware/permissionMiddleware.js`

#### Subtask 4.3: Feature 005 - Token Tracking Integration

- **Spec Reference**: Feature 005 dependency, Success Metrics
- **Description**: Track AI token usage per JD version
- **Methods**: `TokenUsageService.recordJDAnalysis(jdVersionId, tokenCount, operationType)`
- **Depends On**: Feature 005
- **Test Coverage**: Testing 8.6
- **Acceptance**: Token usage attributable to specific versions
- **Files**: `services/TokenUsageService.js`

---

## Frontend

### Main Task 5: JD Editor & Version Management UI

**Description**: UI components for editing JDs, viewing history, comparing versions, and managing rollbacks.

**Spec Reference**: User Story 1, 3, 4

#### Subtask 5.1: Enhanced Job Description Editor

- **Spec Reference**: User Story 1, FR-001, FR-012
- **Description**: Modify job edit form to support version creation with change summary
- **Components**: `JobDescriptionEditor`, `ChangeSummaryDialog` (modal)
- **Features**: "Save with Summary" button, concurrent edit conflict notification
- **Depends On**: Backend 2.13
- **Test Coverage**: Testing 9.1
- **Acceptance**: Edit + save in <5 seconds; conflict notification works
- **Files**: `components/JobDescriptionEditor.tsx`, `components/ChangeSummaryDialog.tsx`

#### Subtask 5.2: Version History View

- **Spec Reference**: User Story 3, FR-004
- **Description**: Display all JD versions in table with metadata
- **Components**: `VersionHistoryPanel`, `VersionHistoryTable`, `VersionBadge`
- **Features**: Columns: Version, Date, Author, Summary, Candidates; sortable/filterable
- **Depends On**: Backend 2.8
- **Related**: Backend 2.3
- **Test Coverage**: Testing 9.2
- **Acceptance**: Loads in <3 seconds for 50+ versions
- **Files**: `components/VersionHistoryPanel.tsx`, `components/VersionHistoryTable.tsx`

#### Subtask 5.3: Version Comparison View (Diff Display)

- **Spec Reference**: User Story 3, FR-005, FR-006
- **Description**: Side-by-side diff viewer with syntax highlighting
- **Components**: `VersionCompareDialog`, `DiffViewer` (use `react-diff-viewer`)
- **Features**: Select versions dropdown, green/red highlighting, "View as of Date" picker
- **Depends On**: Backend 2.10, Backend 2.12
- **Related**: Backend 2.5, Backend 2.7
- **Test Coverage**: Testing 9.3
- **Acceptance**: Diff renders in <2 seconds; correct highlighting
- **Files**: `components/VersionCompareDialog.tsx`, `components/DiffViewer.tsx`

#### Subtask 5.4: Rollback & Activation UI

- **Spec Reference**: User Story 4, FR-007
- **Description**: Action buttons in version history for rollback/activation
- **Components**: `RollbackConfirmDialog`, `VersionActionsMenu`
- **Features**: Confirmation modal with reason field, "Undo Rollback" button (30s timeout)
- **Depends On**: Backend 2.11
- **Related**: Backend 2.6
- **Test Coverage**: Testing 9.4
- **Acceptance**: Rollback in <5 seconds; confirmation required; undo works
- **Files**: `components/RollbackConfirmDialog.tsx`, `components/VersionActionsMenu.tsx`

#### Subtask 5.5: Candidate Results with Version Attribution

- **Spec Reference**: User Story 2, User Story 3, FR-009
- **Description**: Display JD version used for each candidate assessment
- **Components**: Modify `CandidateResultCard`, `CandidateRankingTable`
- **Features**: Version badge, "JD Version" column, clickable to view content
- **Depends On**: Backend 3.1
- **Test Coverage**: Testing 9.5
- **Acceptance**: All results show version; version is clickable
- **Files**: `components/CandidateResultCard.tsx`, `pages/CandidateRankingPage.tsx`

---

## Testing

### Main Task 6: Backend Service Tests

**Description**: Unit and integration tests for JD versioning service layer.

**Spec Reference**: FR-001, FR-004, FR-005, FR-007, FR-008

#### Subtask 6.1: Test Initial Version Creation

- **Spec Reference**: User Story 1 (GWT scenarios), FR-001
- **Description**: Test version 1.0 creation on new job
- **Covers**: Backend 2.1
- **Type**: Unit
- **Scenarios**: Version 1.0 created, FK updated, changelog created, transaction rollback on failure
- **Acceptance**: 100% coverage of createInitialVersion method
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.2: Test Automatic Version Creation

- **Spec Reference**: User Story 1 (GWT scenarios), FR-001, FR-008
- **Description**: Test version creation on JD update
- **Covers**: Backend 2.2
- **Type**: Integration
- **Scenarios**: Version increments, inactive marking, hash comparison, concurrent edit detection
- **Acceptance**: All scenarios pass; atomicity verified
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.3: Test Version History Query

- **Spec Reference**: User Story 3 (GWT scenarios), FR-004, SC-004
- **Description**: Test history retrieval with metadata
- **Covers**: Backend 2.3
- **Type**: Integration
- **Scenarios**: Descending order, creator name join, candidate count, multi-tenant isolation, <3s for 50+ versions
- **Acceptance**: All pass; query optimized
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.4: Test Version Content Retrieval

- **Spec Reference**: User Story 3 (GWT scenarios), FR-004
- **Description**: Test specific version content fetch
- **Covers**: Backend 2.4
- **Type**: Unit (with S3 mock)
- **Scenarios**: Inline content, S3 fetch, multi-tenant check, 404 for non-existent
- **Acceptance**: All pass; S3 mock validated
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.5: Test Diff Generation

- **Spec Reference**: User Story 3 (GWT scenarios), FR-005, SC-005
- **Description**: Test version comparison and diff output
- **Covers**: Backend 2.5
- **Type**: Unit
- **Scenarios**: Added text, deleted text, unchanged, 50KB performance
- **Acceptance**: Diff accuracy verified; <2 seconds
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.6: Test Rollback Functionality

- **Spec Reference**: User Story 4 (GWT scenarios), FR-007, FR-008, SC-006
- **Description**: Test rollback operation
- **Covers**: Backend 2.6
- **Type**: Integration
- **Scenarios**: New version with old content, is_rollback flag, audit log, permission check
- **Acceptance**: All pass; <5 seconds
- **Files**: `tests/services/JobDescriptionService.test.js`

#### Subtask 6.7: Test Temporal Query

- **Spec Reference**: User Story 3 (GWT scenarios), FR-006
- **Description**: Test "as of date" version retrieval
- **Covers**: Backend 2.7
- **Type**: Integration
- **Scenarios**: Correct version for past date, 404 for pre-creation date, edge cases
- **Acceptance**: All pass
- **Files**: `tests/services/JobDescriptionService.test.js`

---

### Main Task 7: API Endpoint Tests

**Description**: Integration tests for all versioning API endpoints.

**Spec Reference**: FR-004, FR-005, FR-006, FR-007, FR-012

#### Subtask 7.1: Test GET /api/jobs/{jobId}/versions

- **Spec Reference**: User Story 3, FR-004, FR-012
- **Description**: Test version history API endpoint
- **Covers**: Backend 2.8
- **Type**: API Integration
- **Scenarios**: Returns history, 401 unauthenticated, 403 unauthorized, multi-tenant isolation
- **Acceptance**: All pass; authorization enforced
- **Files**: `tests/api/JobVersionsAPI.test.js`

#### Subtask 7.2: Test GET /api/jobs/{jobId}/versions/{versionId}

- **Spec Reference**: User Story 3, FR-004
- **Description**: Test specific version retrieval API
- **Covers**: Backend 2.9
- **Type**: API Integration
- **Scenarios**: Returns content, 404 non-existent, 403 unauthorized, large JDs
- **Acceptance**: All pass; <2 seconds
- **Files**: `tests/api/JobVersionsAPI.test.js`

#### Subtask 7.3: Test POST /api/jobs/{jobId}/versions/compare

- **Spec Reference**: User Story 3, FR-005, SC-005
- **Description**: Test version comparison API
- **Covers**: Backend 2.10
- **Type**: API Integration
- **Scenarios**: Returns diff, 400 invalid IDs, 403 unauthorized, performance
- **Acceptance**: All pass; diff accurate
- **Files**: `tests/api/JobVersionsAPI.test.js`

#### Subtask 7.4: Test POST /api/jobs/{jobId}/versions/rollback

- **Spec Reference**: User Story 4, FR-007, FR-012, SC-006
- **Description**: Test rollback API endpoint
- **Covers**: Backend 2.11
- **Type**: API Integration
- **Scenarios**: Creates new version, 403 unauthorized, 400 invalid target, audit created
- **Acceptance**: All pass; <5 seconds
- **Files**: `tests/api/JobVersionsAPI.test.js`

#### Subtask 7.5: Test GET /api/jobs/{jobId}/versions/as-of-date

- **Spec Reference**: User Story 3, FR-006
- **Description**: Test temporal query API
- **Covers**: Backend 2.12
- **Type**: API Integration
- **Scenarios**: Correct version, 404 pre-creation, 400 invalid format
- **Acceptance**: All pass
- **Files**: `tests/api/JobVersionsAPI.test.js`

#### Subtask 7.6: Test PUT /api/jobs/{jobId} with Versioning

- **Spec Reference**: User Story 1, FR-001, SC-001
- **Description**: Test JD update triggers versioning
- **Covers**: Backend 2.13
- **Type**: API Integration
- **Scenarios**: Creates version on content change, no version if unchanged, 409 concurrent edit
- **Acceptance**: All pass
- **Files**: `tests/api/JobsAPI.test.js`

---

### Main Task 8: Interview Session & Integration Tests

**Description**: Test JD locking and feature integrations.

**Spec Reference**: FR-002, FR-003, FR-009, FR-010, FR-012

#### Subtask 8.1: Test JD Capture at Session Start

- **Spec Reference**: User Story 2 (GWT scenarios), FR-002, FR-009, SC-002
- **Description**: Test version capture on interview start
- **Covers**: Backend 3.1
- **Type**: Integration
- **Scenarios**: Version captured, timestamp set, atomic operation, no null references
- **Acceptance**: 100% sessions have jd_version_id
- **Files**: `tests/services/InterviewSessionService.test.js`

#### Subtask 8.2: Test Locked Version for AI

- **Spec Reference**: User Story 2 (GWT scenarios), FR-003, SC-003
- **Description**: Test AI uses locked version
- **Covers**: Backend 3.2
- **Type**: Integration (with AI mock)
- **Scenarios**: AI uses locked version, version unchanged mid-interview, different versions for different candidates
- **Acceptance**: Immutability verified
- **Files**: `tests/services/InterviewSessionService.test.js`

#### Subtask 8.3: Test POST /api/interviews/start

- **Spec Reference**: User Story 2, FR-002
- **Description**: Test interview start API with version capture
- **Covers**: Backend 3.3
- **Type**: API Integration
- **Scenarios**: Session created with version, metadata returned, email verification required
- **Acceptance**: All pass
- **Files**: `tests/api/InterviewsAPI.test.js`

#### Subtask 8.4: Test Feature 003 Integration

- **Spec Reference**: FR-010, SC-007
- **Description**: Test status history includes JD version
- **Covers**: Backend 4.1
- **Type**: Integration
- **Scenarios**: Status change includes jd_version_id, audit trail complete
- **Acceptance**: 100% of status changes include version
- **Files**: `tests/integration/Feature003Integration.test.js`

#### Subtask 8.5: Test Feature 004 Integration

- **Spec Reference**: FR-012, SC-009
- **Description**: Test permission enforcement
- **Covers**: Backend 4.2
- **Type**: Integration
- **Scenarios**: Permission checks at API/service level, 403 for unauthorized
- **Acceptance**: All permissions enforced
- **Files**: `tests/integration/Feature004Integration.test.js`

#### Subtask 8.6: Test Feature 005 Integration

- **Spec Reference**: Feature 005 dependency
- **Description**: Test token usage tracking per version
- **Covers**: Backend 4.3
- **Type**: Integration
- **Scenarios**: Token usage recorded, attributable to versions
- **Acceptance**: All token events tracked
- **Files**: `tests/integration/Feature005Integration.test.js`

---

### Main Task 9: Frontend E2E Tests

**Description**: End-to-end tests for UI components.

**Spec Reference**: User Story 1, 2, 3, 4

#### Subtask 9.1: Test JD Editor with Versioning

- **Spec Reference**: User Story 1 (GWT scenarios), SC-001
- **Description**: Test editor saves with version creation
- **Covers**: Frontend 5.1
- **Type**: E2E (Playwright)
- **Scenarios**: Edit + save creates version, save with summary works, conflict notification
- **Acceptance**: All pass; UI responsive
- **Files**: `e2e/JobDescriptionEditor.spec.ts`

#### Subtask 9.2: Test Version History View

- **Spec Reference**: User Story 3 (GWT scenarios), SC-004
- **Description**: Test version history display
- **Covers**: Frontend 5.2
- **Type**: E2E
- **Scenarios**: History loads <3s, columns display correctly, sort/filter works
- **Acceptance**: All pass
- **Files**: `e2e/VersionHistory.spec.ts`

#### Subtask 9.3: Test Diff Viewer

- **Spec Reference**: User Story 3 (GWT scenarios), SC-005
- **Description**: Test version comparison UI
- **Covers**: Frontend 5.3
- **Type**: E2E
- **Scenarios**: Diff displays <2s, highlighting correct, date picker works
- **Acceptance**: All pass
- **Files**: `e2e/VersionComparison.spec.ts`

#### Subtask 9.4: Test Rollback UI

- **Spec Reference**: User Story 4 (GWT scenarios), SC-006
- **Description**: Test rollback workflow
- **Covers**: Frontend 5.4
- **Type**: E2E
- **Scenarios**: Confirmation modal, rollback completes, undo works, UI updates
- **Acceptance**: All pass
- **Files**: `e2e/VersionRollback.spec.ts`

#### Subtask 9.5: Test Version Attribution in Results

- **Spec Reference**: User Story 2, User Story 3 (GWT scenarios)
- **Description**: Test version display in candidate results
- **Covers**: Frontend 5.5
- **Type**: E2E
- **Scenarios**: Version badge visible, column displays, clickable links
- **Acceptance**: All pass
- **Files**: `e2e/CandidateResults.spec.ts`

---

## Task Summary

| Area      | Main Tasks | Subtasks |
| --------- | ---------- | -------- |
| Database  | 1          | 5        |
| Backend   | 4          | 20       |
| Frontend  | 1          | 5        |
| Testing   | 4          | 20       |
| **Total** | **10**     | **50**   |

**Priority Distribution**:

- **P1 (MVP)**: Database 1, Backend 2-3, Frontend 5, Testing 6-9
- **P2 (Enhancement)**: Backend 4 (integrations)

---

## Dependencies & Sequencing

### Phase 1: Foundation (Week 1-2)

**Must complete first - blocks all other work**

1. Database 1.1-1.5: All schema migrations
2. Backend 2.1-2.2: Core version creation logic
3. Backend 2.3-2.4: Version history and retrieval
4. Backend 2.8-2.9, 2.13: Basic API endpoints

### Phase 2: Core Features (Week 3-4)

**Can parallelize across tracks**

**Track A - Interview Integration:**

- Backend 3.1-3.3: JD version locking
- Testing 8.1-8.3: Session tests

**Track B - Advanced Versioning:**

- Backend 2.5-2.7: Diff, rollback, temporal query
- Backend 2.10-2.12: Advanced APIs
- Testing 6.1-6.7, 7.1-7.6: Backend/API tests

**Track C - Frontend:**

- Frontend 5.1-5.5: All UI components

### Phase 3: Integration & Polish (Week 5-6)

1. Backend 4.1-4.3: Feature integrations (003, 004, 005)
2. Testing 8.4-8.5: Integration tests
3. Testing 9.1-9.5: Frontend E2E tests
4. Final QA and performance validation

---

## Notes

### Technical Decisions

- **Content Storage**: Inline for <100KB, S3 for larger JDs
- **Version Numbering**: Simple auto-increment (1, 2, 3...)
- **Diff Algorithm**: Myers diff via `diff-match-patch` library
- **Concurrent Edits**: Last-write-wins with optimistic locking (409 Conflict)

### Performance Targets

- Version history query: <3 seconds for 50+ versions
- Diff generation: <2 seconds for 50KB JDs
- Rollback operation: <5 seconds

### Multi-Tenant Security

- All queries MUST filter by `company_id`
- Consider Postgres RLS policies for additional security

### Open Questions

1. Export version history for compliance (PDF)? → Defer to Phase 3
2. Very large histories (1000+ versions)? → Pagination + lazy loading
3. Version branching for A/B testing? → Future enhancement

---

## Validation Summary

✅ All references point to existing tasks  
✅ Dependency flow: Database → Backend → Frontend → Testing  
✅ P1 tasks have test coverage  
✅ Simplified linkages: Depends On, Related, Test Coverage
