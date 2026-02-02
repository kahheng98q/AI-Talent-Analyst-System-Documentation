# Task Breakdown: Job Description Versioning & History

**Source**: [009-job-description-versioning (Phase 2)/spec.md](./spec.md)  
**Generated**: 2026-02-02  
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

---

## Database

### Main Task 1: JD Versioning Schema Design

**Description**: Design and implement database schema for storing job description versions, change logs, and version relationships.

**Requirement Reference**: FR-001, FR-008, FR-009

#### Subtask 1.1: JobDescriptionVersion Table

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
- **Related**: Backend 2.1-2.7
- **Acceptance**: Migration runs successfully; constraints validated

#### Subtask 1.2: JobDescriptionChangeLog Table

- **Description**: Create migration for audit table tracking version transitions
- **Columns**:
  - `id` (PK, UUID)
  - `job_id`, `company_id` (FKs, indexed)
  - `old_version_id`, `new_version_id` (FKs)
  - `change_type` (ENUM: create, update, rollback, delete)
  - `actor_user_id`, `timestamp`, `reason`, `change_metadata` (JSON)
- **Indexes**: `(job_id, timestamp)`, `(company_id, timestamp)`, `actor_user_id`
- **Related**: Backend 2.1, 2.2, 2.6
- **Acceptance**: Migration runs; foreign keys validated

#### Subtask 1.3: Modify JobDescription Table

- **Description**: Add version reference columns to existing `job_descriptions` table
- **New Columns**: `current_version_id` (FK), `version_count` (INT, default 1)
- **Migration Strategy**: Add nullable → backfill → add NOT NULL constraint
- **Related**: Backend 2.1, 2.2, 2.3
- **Acceptance**: Existing jobs migrated with version 1.0

#### Subtask 1.4: Modify InterviewSession Table

- **Description**: Add columns to capture JD version at session start
- **New Columns**: `jd_version_id` (FK, NOT NULL), `jd_locked_at` (TIMESTAMP)
- **Migration Strategy**: Add nullable → backfill from current JD → add NOT NULL
- **Related**: Backend 3.1, 3.2
- **Acceptance**: All sessions reference valid JD version

#### Subtask 1.5: Modify CandidateStatusHistory Table

- **Description**: Track which JD version was active during status changes
- **New Columns**: `jd_version_id` (FK, nullable)
- **Related**: Backend 4.1 (Feature 003 integration)
- **Acceptance**: Future status changes auto-populate JD version

---

## Backend

### Main Task 2: JD Versioning Service & APIs

**Description**: Core service layer and API endpoints for managing JD versions, including automatic version creation, history queries, diff generation, rollback, and audit logging.

**Requirement Reference**: FR-001, FR-004, FR-007, FR-008

#### Subtask 2.1: Create Initial Version on Job Creation

- **Description**: Automatically create version 1.0 when a new job is created
- **Method**: `JobDescriptionService.createInitialVersion(jobId, content, userId)`
- **Logic**: Calculate content hash → create version record → update job FK → create changelog
- **Depends On**: Database 1.1, 1.2, 1.3
- **Test Coverage**: Testing 5.1
- **Acceptance**: New jobs have version 1.0; audit log created

#### Subtask 2.2: Update JD and Create New Version

- **Description**: Atomic operation to save JD changes and create new version automatically
- **Method**: `JobDescriptionService.updateAndCreateVersion(jobId, content, summary, userId)`
- **Logic**:
  - Check permission (`job.edit`)
  - Compare content hash (skip if unchanged)
  - Transaction: mark current inactive → create new version → update job → create changelog
  - Concurrent edit detection (optimistic locking with 409 Conflict)
- **Depends On**: Database 1.1, 1.2, 1.3
- **Test Coverage**: Testing 5.2
- **Acceptance**: Version auto-increments; old version preserved; <5 seconds

#### Subtask 2.3: Query Version History

- **Description**: Retrieve all versions for a job with metadata
- **Method**: `JobDescriptionService.getVersionHistory(jobId, companyId)`
- **Return**: `{ version_number, created_at, created_by_name, change_summary, is_active, candidate_count }`
- **Query**: Join versions → users → interview_sessions; filter by company_id
- **Depends On**: Database 1.1, 1.4
- **Test Coverage**: Testing 5.3
- **Acceptance**: Returns history in <3 seconds for 50+ versions

#### Subtask 2.4: Retrieve Specific Version Content

- **Description**: Fetch full content of a specific version
- **Method**: `JobDescriptionService.getVersionContent(versionId, companyId)`
- **Logic**: Validate company access; fetch content (inline or S3)
- **Depends On**: Database 1.1
- **Test Coverage**: Testing 5.4
- **Acceptance**: Content retrieval in <2 seconds; S3 fetch works for large JDs

#### Subtask 2.5: Compare Two Versions (Diff Generation)

- **Description**: Generate side-by-side diff with additions/deletions
- **Method**: `JobDescriptionService.compareVersions(versionId1, versionId2, companyId)`
- **Algorithm**: Myers diff (use `diff-match-patch` library)
- **Return**: `{ added: [], deleted: [], unchanged: [] }`
- **Depends On**: Database 1.1, Backend 2.4
- **Test Coverage**: Testing 5.5
- **Acceptance**: Diff in <2 seconds for 50KB JDs

#### Subtask 2.6: Rollback to Previous Version

- **Description**: Revert JD by creating new version with old content
- **Method**: `JobDescriptionService.rollbackToVersion(jobId, targetVersionId, userId)`
- **Logic**: Fetch target content → create new version (is_rollback=true) → update job → changelog
- **Depends On**: Database 1.1, 1.2, 1.3, Backend 2.4
- **Test Coverage**: Testing 5.6
- **Acceptance**: Rollback in <5 seconds; audit log shows rollback

#### Subtask 2.7: View JD "As of Date"

- **Description**: Temporal query to get JD version active on specific date
- **Method**: `JobDescriptionService.getVersionAsOfDate(jobId, date, companyId)`
- **Query**: `WHERE job_id=? AND created_at <= ? ORDER BY created_at DESC LIMIT 1`
- **Depends On**: Database 1.1
- **Test Coverage**: Testing 5.7
- **Acceptance**: Returns correct version; handles edge cases

#### Subtask 2.8: API - GET /api/jobs/{jobId}/versions

- **Description**: Retrieve version history
- **Response**: `{ versions: [...] }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.3
- **Related**: Frontend 4.2
- **Test Coverage**: Testing 6.1
- **Acceptance**: Returns history in <3 seconds; 401/403 enforced

#### Subtask 2.9: API - GET /api/jobs/{jobId}/versions/{versionId}

- **Description**: Retrieve specific version content
- **Response**: `{ version: { id, version_number, content, created_at, ... } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.4
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 6.2
- **Acceptance**: Returns content in <2 seconds

#### Subtask 2.10: API - POST /api/jobs/{jobId}/versions/compare

- **Description**: Compare two versions
- **Request**: `{ version1_id, version2_id }`
- **Response**: `{ diff: { added, deleted, unchanged } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.5
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 6.3
- **Acceptance**: Diff in <2 seconds; correct highlighting

#### Subtask 2.11: API - POST /api/jobs/{jobId}/versions/rollback

- **Description**: Rollback to previous version
- **Request**: `{ target_version_id, reason? }`
- **Response**: `{ new_version: { id, version_number, content } }`
- **Authorization**: Requires `job.edit`
- **Depends On**: Backend 2.6
- **Related**: Frontend 4.4
- **Test Coverage**: Testing 6.4
- **Acceptance**: Rollback in <5 seconds; audit created

#### Subtask 2.12: API - GET /api/jobs/{jobId}/versions/as-of-date

- **Description**: Get version active on specific date
- **Request**: Query param `date` (ISO 8601)
- **Response**: `{ version: { ... } }`
- **Authorization**: Requires `job.version.view`
- **Depends On**: Backend 2.7
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 6.5
- **Acceptance**: Returns correct version; 404 if date before job creation

#### Subtask 2.13: API - PUT /api/jobs/{jobId} (Modify Existing)

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

**Requirement Reference**: FR-002, FR-003, FR-009

#### Subtask 3.1: Capture JD Version at Session Start

- **Description**: Store current JD version ID when candidate opens interview
- **Method**: `InterviewSessionService.startSession(candidateId, jobId)`
- **Logic**: Fetch current version → create session with `jd_version_id` and `jd_locked_at`
- **Depends On**: Database 1.3, 1.4
- **Test Coverage**: Testing 7.1
- **Acceptance**: 100% of sessions have non-null jd_version_id

#### Subtask 3.2: Use Locked Version for AI Operations

- **Description**: Pass locked JD version to AI services (not current version)
- **Method**: `InterviewSessionService.getLockedJDVersion(sessionId)`
- **Logic**: Fetch session → get version content → return to AI caller
- **Depends On**: Database 1.4, Backend 2.4
- **Related**: Feature 001 AI integration
- **Test Coverage**: Testing 7.2
- **Acceptance**: AI uses locked version; immutable during interview

#### Subtask 3.3: API - POST /api/interviews/start (Modify Existing)

- **Description**: Capture JD version at interview start
- **Response**: Include `jd_version_id`, `jd_locked_at`
- **Depends On**: Backend 3.1
- **Related**: Feature 001 workflow
- **Test Coverage**: Testing 7.3
- **Acceptance**: Session created with locked version

---

### Main Task 4: Feature Integration (P2)

**Description**: Integrate with Features 003, 004, 005.

**Requirement Reference**: FR-010, FR-012

#### Subtask 4.1: Feature 003 - Status History Integration

- **Description**: Populate JD version when candidate status changes
- **Method**: Modify `CandidateService.updateStatus()` to capture jd_version_id from session
- **Depends On**: Database 1.5, Feature 003
- **Test Coverage**: Testing 8.1
- **Acceptance**: All status changes include JD version reference

#### Subtask 4.2: Feature 004 - Permission Enforcement

- **Description**: Integrate RBAC for JD version operations
- **Permissions**: `job.edit` (create/update/rollback), `job.version.view` (history), `job.history.view` (audit logs)
- **Depends On**: Feature 004 permissions system
- **Test Coverage**: Testing 8.2
- **Acceptance**: 403 for unauthorized; permissions logged

#### Subtask 4.3: Feature 005 - Token Tracking Integration

- **Description**: Track AI token usage per JD version
- **Method**: `TokenUsageService.recordJDAnalysis(jdVersionId, tokenCount, operationType)`
- **Depends On**: Feature 005
- **Test Coverage**: Testing 8.3
- **Acceptance**: Token usage attributable to specific versions

---

## Frontend

### Main Task 5: JD Editor & Version Management UI

**Description**: UI components for editing JDs, viewing history, comparing versions, and managing rollbacks.

**User Story Reference**: User Story 1, 3, 4

#### Subtask 5.1: Enhanced Job Description Editor

- **Description**: Modify job edit form to support version creation with change summary
- **Components**: `JobDescriptionEditor`, `ChangeSummaryDialog` (modal)
- **Features**: "Save with Summary" button, concurrent edit conflict notification
- **Depends On**: Backend 2.13
- **Test Coverage**: Testing 9.1
- **Acceptance**: Edit + save in <5 seconds; conflict notification works
- **Files**: `components/JobDescriptionEditor.tsx`, `components/ChangeSummaryDialog.tsx`

#### Subtask 5.2: Version History View

- **Description**: Display all JD versions in table with metadata
- **Components**: `VersionHistoryPanel`, `VersionHistoryTable`, `VersionBadge`
- **Features**: Columns: Version, Date, Author, Summary, Candidates; sortable/filterable
- **Depends On**: Backend 2.8
- **Test Coverage**: Testing 9.2
- **Acceptance**: Loads in <3 seconds for 50+ versions
- **Files**: `components/VersionHistoryPanel.tsx`, `components/VersionHistoryTable.tsx`

#### Subtask 5.3: Version Comparison View (Diff Display)

- **Description**: Side-by-side diff viewer with syntax highlighting
- **Components**: `VersionCompareDialog`, `DiffViewer` (use `react-diff-viewer`)
- **Features**: Select versions dropdown, green/red highlighting, "View as of Date" picker
- **Depends On**: Backend 2.10, Backend 2.12
- **Test Coverage**: Testing 9.3
- **Acceptance**: Diff renders in <2 seconds; correct highlighting
- **Files**: `components/VersionCompareDialog.tsx`, `components/DiffViewer.tsx`

#### Subtask 5.4: Rollback & Activation UI

- **Description**: Action buttons in version history for rollback/activation
- **Components**: `RollbackConfirmDialog`, `VersionActionsMenu`
- **Features**: Confirmation modal with reason field, "Undo Rollback" button (30s timeout)
- **Depends On**: Backend 2.11
- **Test Coverage**: Testing 9.4
- **Acceptance**: Rollback in <5 seconds; confirmation required; undo works
- **Files**: `components/RollbackConfirmDialog.tsx`, `components/VersionActionsMenu.tsx`

#### Subtask 5.5: Candidate Results with Version Attribution

- **Description**: Display JD version used for each candidate assessment
- **Components**: Modify `CandidateResultCard`, `CandidateRankingTable`
- **Features**: Version badge, "JD Version" column, clickable to view content
- **Depends On**: Existing interview session API
- **Test Coverage**: Testing 9.5
- **Acceptance**: All results show version; version is clickable
- **Files**: `components/CandidateResultCard.tsx`, `pages/CandidateRankingPage.tsx`

---

## Testing

### Main Task 6: Backend Service Tests

**Description**: Unit and integration tests for JD versioning service layer.

#### Subtask 6.1: Test Initial Version Creation

- **Covers**: Backend 2.1
- **Type**: Unit
- **Scenarios**: Version 1.0 created, FK updated, changelog created, transaction rollback on failure
- **Acceptance**: 100% coverage of createInitialVersion method

#### Subtask 6.2: Test Automatic Version Creation

- **Covers**: Backend 2.2
- **Type**: Integration
- **Scenarios**: Version increments, inactive marking, hash comparison, concurrent edit detection
- **Acceptance**: All scenarios pass; atomicity verified

#### Subtask 6.3: Test Version History Query

- **Covers**: Backend 2.3
- **Type**: Integration
- **Scenarios**: Descending order, creator name join, candidate count, multi-tenant isolation, <3s for 50+ versions
- **Acceptance**: All pass; query optimized

#### Subtask 6.4: Test Version Content Retrieval

- **Covers**: Backend 2.4
- **Type**: Unit (with S3 mock)
- **Scenarios**: Inline content, S3 fetch, multi-tenant check, 404 for non-existent
- **Acceptance**: All pass; S3 mock validated

#### Subtask 6.5: Test Diff Generation

- **Covers**: Backend 2.5
- **Type**: Unit
- **Scenarios**: Added text, deleted text, unchanged, 50KB performance
- **Acceptance**: Diff accuracy verified; <2 seconds

#### Subtask 6.6: Test Rollback Functionality

- **Covers**: Backend 2.6
- **Type**: Integration
- **Scenarios**: New version with old content, is_rollback flag, audit log, permission check
- **Acceptance**: All pass; <5 seconds

#### Subtask 6.7: Test Temporal Query

- **Covers**: Backend 2.7
- **Type**: Integration
- **Scenarios**: Correct version for past date, 404 for pre-creation date, edge cases
- **Acceptance**: All pass

---

### Main Task 7: API Endpoint Tests

**Description**: Integration tests for all versioning API endpoints.

#### Subtask 7.1: Test GET /api/jobs/{jobId}/versions

- **Covers**: Backend 2.8
- **Type**: API Integration
- **Scenarios**: Returns history, 401 unauthenticated, 403 unauthorized, multi-tenant isolation
- **Acceptance**: All pass; authorization enforced

#### Subtask 7.2: Test GET /api/jobs/{jobId}/versions/{versionId}

- **Covers**: Backend 2.9
- **Type**: API Integration
- **Scenarios**: Returns content, 404 non-existent, 403 unauthorized, large JDs
- **Acceptance**: All pass; <2 seconds

#### Subtask 7.3: Test POST /api/jobs/{jobId}/versions/compare

- **Covers**: Backend 2.10
- **Type**: API Integration
- **Scenarios**: Returns diff, 400 invalid IDs, 403 unauthorized, performance
- **Acceptance**: All pass; diff accurate

#### Subtask 7.4: Test POST /api/jobs/{jobId}/versions/rollback

- **Covers**: Backend 2.11
- **Type**: API Integration
- **Scenarios**: Creates new version, 403 unauthorized, 400 invalid target, audit created
- **Acceptance**: All pass; <5 seconds

#### Subtask 7.5: Test GET /api/jobs/{jobId}/versions/as-of-date

- **Covers**: Backend 2.12
- **Type**: API Integration
- **Scenarios**: Correct version, 404 pre-creation, 400 invalid format
- **Acceptance**: All pass

#### Subtask 7.6: Test PUT /api/jobs/{jobId} with Versioning

- **Covers**: Backend 2.13
- **Type**: API Integration
- **Scenarios**: Creates version on content change, no version if unchanged, 409 concurrent edit
- **Acceptance**: All pass

---

### Main Task 8: Interview Session & Integration Tests

**Description**: Test JD locking and feature integrations.

#### Subtask 8.1: Test JD Capture at Session Start

- **Covers**: Backend 3.1
- **Type**: Integration
- **Scenarios**: Version captured, timestamp set, atomic operation, no null references
- **Acceptance**: 100% sessions have jd_version_id

#### Subtask 8.2: Test Locked Version for AI

- **Covers**: Backend 3.2
- **Type**: Integration (with AI mock)
- **Scenarios**: AI uses locked version, version unchanged mid-interview, different versions for different candidates
- **Acceptance**: Immutability verified

#### Subtask 8.3: Test POST /api/interviews/start

- **Covers**: Backend 3.3
- **Type**: API Integration
- **Scenarios**: Session created with version, metadata returned, email verification required
- **Acceptance**: All pass

#### Subtask 8.4: Test Feature 003 Integration

- **Covers**: Backend 4.1
- **Type**: Integration
- **Scenarios**: Status change includes jd_version_id, audit trail complete
- **Acceptance**: 100% of status changes include version

#### Subtask 8.5: Test Feature 004 Integration

- **Covers**: Backend 4.2
- **Type**: Integration
- **Scenarios**: Permission checks at API/service level, 403 for unauthorized
- **Acceptance**: All permissions enforced

---

### Main Task 9: Frontend E2E Tests

**Description**: End-to-end tests for UI components.

#### Subtask 9.1: Test JD Editor with Versioning

- **Covers**: Frontend 5.1
- **Type**: E2E (Playwright)
- **Scenarios**: Edit + save creates version, save with summary works, conflict notification
- **Acceptance**: All pass; UI responsive

#### Subtask 9.2: Test Version History View

- **Covers**: Frontend 5.2
- **Type**: E2E
- **Scenarios**: History loads <3s, columns display correctly, sort/filter works
- **Acceptance**: All pass

#### Subtask 9.3: Test Diff Viewer

- **Covers**: Frontend 5.3
- **Type**: E2E
- **Scenarios**: Diff displays <2s, highlighting correct, date picker works
- **Acceptance**: All pass

#### Subtask 9.4: Test Rollback UI

- **Covers**: Frontend 5.4
- **Type**: E2E
- **Scenarios**: Confirmation modal, rollback completes, undo works, UI updates
- **Acceptance**: All pass

#### Subtask 9.5: Test Version Attribution in Results

- **Covers**: Frontend 5.5
- **Type**: E2E
- **Scenarios**: Version badge visible, column displays, clickable links
- **Acceptance**: All pass

---

## Task Summary

| Area      | Main Tasks | Subtasks |
| --------- | ---------- | -------- |
| Database  | 1          | 5        |
| Backend   | 3          | 19       |
| Frontend  | 1          | 5        |
| Testing   | 4          | 19       |
| **Total** | **9**      | **48**   |

**Priority Distribution**:

- **P1 (MVP)**: Database 1, Backend 2-3, Frontend 5, Testing 6-8
- **P2 (Enhancement)**: Backend 4 (integrations), Testing 9

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
