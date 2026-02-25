# Feature Specification: Job Description Versioning & History

**Feature Branch**: `009-job-description-versioning`  
**Created**: 2026-01-13  
**Status**: Draft  
**Input**: User description: "Add job description versioning to track JD changes, lock versions during interview sessions, support rollback, and provide audit trails for compliance and fair candidate evaluation."

> **Admin Portal Status**: Partially implemented. A version history UI panel exists in `job-description-details-view.tsx` but currently uses hardcoded mock data (`MOCK_VERSIONS` array) — it is NOT connected to a real API. US2–US4 are not implemented.

## Clarifications

### Session 2026-01-13

- Q: Should JD version creation happen automatically on every edit, or require explicit "save as version"? → A: Automatic version creation on any content change; simpler and safer for compliance.
- Q: When does the JD version get locked for an interview session? → A: At the moment candidate opens the interview link and confirms email verification.
- Q: Can HR "activate" a previous version for new candidates, or must they rollback? → A: Rollback creates a new version; explicit "Activate Previous" action is available as non-destructive alternative.
- Q: How long are versions retained? → A: See FR-011 (indefinite retention; manual deletion only by HR).
- Q: Should version history affect "Job Description Analysis" token tracking (Feature 005)? → A: Yes—allocate tokens to specific JD versions; support cost-per-version reporting.
- Q: How does JD versioning interact with job status changes (open/filled/on-hold)? → A: Status and version are independent; both tracked separately in audit trails.

## User Scenarios & Testing _(mandatory)_

### User Story 1 - HR Updates Job Description & Creates Version (Priority: P1) `🔶 Partially Implemented`

> Version history UI panel exists in the JD details page, but uses `MOCK_VERSIONS` hardcoded data — not connected to a real API. Editing the JD form is functional but does not persist versioning.

As an HR manager, I want to update a job description during active recruitment, so that I can refine requirements while maintaining audit history of all changes.

**Why this priority**: This is the core functionality—without the ability to update and version JDs, the system lacks operational flexibility and audit capabilities.

**Independent Test**: An HR manager can edit a job description, system automatically creates a new version, and both the current and previous versions are accessible via version history.

**Acceptance Scenarios**:

1. **Given** I have an active job with JD version 1.0, **When** I navigate to the job and click "Edit Description", **Then** I can modify the job description content.

2. **Given** I have edited the job description (e.g., increased salary range), **When** I click "Save Changes", **Then** the system creates a new version (1.1), marks it as current, and stores the previous version (1.0) with the timestamp and my user ID.

3. **Given** I have made changes to the job description, **When** I save, **Then** a `JobDescriptionChangeLog` entry is created with: old version ID, new version ID, timestamp, my user ID, and optional change summary field populated.

4. **Given** I am on the job details page, **When** I view the version history panel, **Then** I see a list of all versions (1.0, 1.1, 1.2, etc.) with creation date, creator name, and change summary.

5. **Given** I want to document why I changed the JD, **When** I click "Save with Summary", **Then** I can enter optional text (e.g., "Fixed salary based on market research") that appears in the version history next to the version number.

---

### User Story 2 - Lock JD Version for Interview Session (Priority: P1) `❌ Not Implemented`

As the system, I must capture and lock the current JD version when a candidate opens an interview, so that resume scoring and question generation use the same JD version throughout the interview session.

**Why this priority**: Critical for fairness—ensures each candidate is evaluated against a single, immutable JD version; prevents mid-interview requirement changes from affecting scoring.

**Independent Test**: When a candidate starts an interview session, the system captures the current JD version ID and stores it in the interview session record. Multiple candidates receive the same interview questions when interviewing for the same job on the same day, but different questions if JD was updated between their sessions.

**Acceptance Scenarios**:

1. **Given** a candidate receives an interview invitation for Job X with JD version 2.0, **When** the candidate clicks the interview link and verifies their email, **Then** the system creates an `InterviewSession` record with `jd_version_id = 2.0`.

2. **Given** the interview session has captured JD version 2.0, **When** HR updates the job description (creating version 2.1) while the candidate is mid-interview, **Then** the candidate's interview questions continue using JD 2.0 (no interruption or re-generation).

3. **Given** two candidates interviewing for the same job, **When** candidate A interviews on Day 1 (JD version 2.0) and candidate B interviews on Day 2 (JD version 2.1 created overnight), **Then** the system generates different questions for each candidate based on their respective locked JD versions.

4. **Given** an interview session is complete, **When** I view the candidate's results, **Then** the result record displays which JD version was used (e.g., "Evaluated using JD Version 2.0 from 2026-01-10").

5. **Given** an HR manager reviewing candidate status history, **When** they view the audit log, **Then** each status change includes a reference to the JD version that was active when the candidate was assessed.

---

### User Story 3 - View & Compare JD Versions (Priority: P2) `❌ Not Implemented`

As an HR manager, I want to see a detailed history of how the job description has changed and compare versions, so I can understand what requirements changed and how they affected candidate pool.

**Why this priority**: Operational visibility; helps HR understand candidate fairness across different JD versions and supports compliance auditing.

**Independent Test**: An HR manager can access a version history view, see all versions in chronological order with metadata, and display a side-by-side diff showing what changed between versions.

**Acceptance Scenarios**:

1. **Given** I am on a job's details page with multiple JD versions, **When** I click "View Version History", **Then** I see a table with columns: Version Number, Created Date, Created By, Change Summary, and Actions.

2. **Given** I am viewing the version history, **When** I select two versions (e.g., 1.0 and 2.0), **Then** the system displays a side-by-side diff highlighting the additions, deletions, and modifications between them.

3. **Given** I am comparing JD versions, **When** I view the diff, **Then** added text is highlighted in green, deleted text in red, and unchanged sections in neutral color.

4. **Given** the version history view, **When** I click "View as of [Date]", **Then** I can see what the job description looked like on that specific date (what candidates would have seen if they opened the link then).

5. **Given** I want to see candidate distribution across versions, **When** I view the version history, **Then** I see a badge for each version showing "Used by X candidates" (e.g., "Version 1.0: Used by 15 candidates").

6. **Given** multiple candidates interviewing for the same job, **When** I generate a ranking report, **Then** the report groups or annotates candidates by JD version used (e.g., "Candidates 1-15 evaluated using JD v1.0; Candidates 16-23 evaluated using JD v2.0").

---

### User Story 4 - Rollback & Version Management (Priority: P2) `❌ Not Implemented`

As an HR manager, I want to revert to a previous job description version if I made a mistake or want to test an earlier version, so I can recover from errors without manual workarounds.

**Why this priority**: Operational safety; reduces risk of erroneous JD edits and supports A/B testing different requirements.

**Independent Test**: An HR manager can select a previous JD version and activate it. A new version is created (rollback version) that restores the previous content, and a new interview link for future candidates uses the activated version.

**Acceptance Scenarios**:

1. **Given** I have JD versions 1.0, 2.0, and 2.1 (current), **When** I click "Revert to Version 2.0" on the version history, **Then** the system creates a new version (2.2) with the same content as 2.0, marks it as current, and logs the rollback action.

2. **Given** I have reverted to version 2.0 (now version 2.2 is active), **When** I send new interview invitations, **Then** new candidates receive invitations with JD version 2.2 (the rollback version).

3. **Given** a previous interview session used JD version 1.0, **When** I revert to version 1.0 now (creating version 2.2), **Then** the previous session's version reference is NOT changed; it still shows version 1.0 in the audit trail.

4. **Given** I am on the version history, **When** I click "Activate Previous Version 1.5" (non-destructive alternative), **Then** version 1.5 is promoted to current without creating a rollback; existing versions are preserved unchanged.

5. **Given** multiple JD versions exist for a job, **When** Super Admin views the job (via Feature 006 God Mode), **Then** they can delete or archive older versions (e.g., versions >6 months old) to manage storage.

6. **Given** I revert a JD by mistake, **When** I immediately click "Undo Rollback", **Then** the system reverts to the version immediately before the rollback (one step backward in version history).

---

### Edge Cases

- What happens if HR deletes a job entirely? All JD versions are soft-deleted with the job; still accessible for audit purposes.
- What if multiple HR managers try to edit the same JD simultaneously? Last-write-wins; second editor sees a "version updated" notification and must refresh to see new version before editing.
- What if a JD version is created but no candidate ever interviews using it? The version remains in history with "0 candidates" badge.
- How does versioning handle very large JDs (100KB+)? Store content in blob storage (S3) with version pointers; history table stores version metadata and content hash for deduplication.
- What if HR reverts to a version, then reverts again? Full version chain is preserved (e.g., 1.0 → 2.0 → 1.0_rollback → 1.0_rollback_undo_rollback); chain is linear and traceable.
- Can a company admin see JD versions from other companies? No; strict multi-tenant isolation (Feature 004 enforces query-level filtering).

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST create a new JD version automatically whenever HR saves changes to the job description content (atomic operation: old version marked inactive, new version marked active).
- **FR-002**: System MUST capture the current JD version ID at the moment a candidate opens the interview link and verifies email (before resume upload or question generation).
- **FR-003**: System MUST use the locked JD version for all AI operations (resume scoring, question generation) throughout the entire interview session, even if JD is updated after session starts.
- **FR-004**: System MUST provide a version history UI showing all historical versions with: version number, creation date/time, creator user name, change summary (optional), and count of candidates assessed using that version.
- **FR-005**: System MUST support side-by-side diff view comparing any two JD versions, highlighting additions (green), deletions (red), and unchanged sections.
- **FR-006**: System MUST allow HR to view the JD as it appeared on a specific date ("View as of [Date]").
- **FR-007**: System MUST allow HR to revert to a previous JD version; reverting creates a new version with the previous content restored, rather than destructively overwriting.
- **FR-008**: System MUST record all JD version changes in an audit log (`JobDescriptionChangeLog`) with: timestamp, actor user ID, old version ID, new version ID, reason/summary, and rollback indicator.
- **FR-009**: System MUST associate each interview session with the JD version used (via `jd_version_id` field), making it immutable and permanently queryable.
- **FR-010**: System MUST associate each candidate status change with the JD version that was active at the time of assessment (Feature 003 integration).
- **FR-011**: System MUST retain all JD versions indefinitely (until job is deleted); no automatic purging or archival.
- **FR-012**: System MUST enforce permission checks for JD edits: only users with `job.edit` permission can create new versions (Feature 004 integration).
- **FR-013**: System MUST support versioning of JD content across all supported formats (plain text, structured fields like qualifications/requirements/salary).

### Key Entities

- **JobDescription** (existing table, modified):
  - `current_version_id` (FK to JobDescriptionVersion) — points to active version
  - `version_count` (INT) — cached count of total versions for quick queries
  - Existing fields: `job_id`, `company_id`, `title`, `status`, `created_at`, `created_by_id`, `is_deleted`

- **JobDescriptionVersion** (new table):
  - `id` (PK, UUID)
  - `job_id` (FK to Job) — which job this version belongs to
  - `company_id` (FK to Company) — multi-tenant isolation
  - `version_number` (INT) — auto-increment per job (1, 2, 3, ...)
  - `content` (LONGTEXT or BLOB reference) — full JD content (plain text or JSON with structured fields)
  - `content_hash` (VARCHAR, SHA256) — for deduplication and change detection
  - `created_at` (TIMESTAMP) — when version was created
  - `created_by_user_id` (FK to User) — HR manager who created this version
  - `change_summary` (TEXT, nullable) — optional human-readable summary of changes
  - `is_active` (BOOL) — true if this is the current version
  - `is_rollback` (BOOL) — true if created by reverting to a previous version
  - `rollback_from_version_id` (FK, nullable) — if is_rollback=true, points to which version was reverted
  - `created_at_index` — for efficient "as of [date]" queries
  - Soft delete: `is_deleted` (BOOL) — mark as deleted when job is deleted, but retain for audit

- **InterviewSession** (existing table, modified):
  - `jd_version_id` (FK to JobDescriptionVersion, NOT NULL) — immutable reference to JD version used
  - `jd_locked_at` (TIMESTAMP) — when JD version was captured (same as session start time)
  - Existing fields: `interview_id`, `candidate_id`, `job_id`, `company_id`, `created_at`, etc.

- **Candidate** (existing table, modified, via Feature 003):
  - No direct changes; versioning handled via interview session

- **CandidateStatusHistory** (existing table, modified, Feature 003 integration):
  - `jd_version_id` (FK to JobDescriptionVersion, nullable) — which JD version was active when this status was assigned
  - Existing fields: `candidate_id`, `old_status`, `new_status`, `timestamp`, `modified_by_user_id`

- **JobDescriptionChangeLog** (new audit table):
  - `id` (PK, UUID)
  - `job_id` (FK to Job)
  - `company_id` (FK to Company) — multi-tenant isolation
  - `old_version_id` (FK to JobDescriptionVersion, nullable) — null if first version
  - `new_version_id` (FK to JobDescriptionVersion, NOT NULL)
  - `change_type` (ENUM: 'create', 'update', 'rollback', 'delete') — semantic marker
  - `actor_user_id` (FK to User, NOT NULL) — who made the change
  - `timestamp` (TIMESTAMP) — when change occurred
  - `reason` (TEXT, nullable) — optional explanation
  - `change_metadata` (JSON, nullable) — field-level diffs or structured change info (future enhancement)

### Dependencies

- **Feature 001** (AI-HR Interview System): Must capture `jd_version_id` at interview session creation; must pass version ID to AI scoring/question generation services.
- **Feature 003** (Status Tracking): Must populate `jd_version_id` in `CandidateStatusHistory` when candidate status changes.
- **Feature 004** (Permissions): Permission enforcement on `job.edit` and new `job.version.view` / `job.history.view` permissions.
- **Feature 005** (Subscriptions): Optional—allocate "Job Description Analysis" tokens to specific JD versions; support cost-per-version reporting (future enhancement).
- **Feature 006** (God Mode): Optional—Admin dashboard showing top-changed JDs, cost impact of version changes, global version cleanup.
- **Feature 007** (Company Admin Self-Service): Optional—UI for company admins to view/manage JD versions for their company.

### Assumptions

- **Automatic Versioning**: See FR-001 (automatic version creation on every content change).
- **Indefinite Retention**: See FR-011 (retain all versions indefinitely; no automatic purging).
- **Immutable Interview Sessions**: Once JD version is locked at interview session start, it cannot be changed (audit integrity).
- **Last-Write-Wins**: Concurrent edits to same JD; second editor's version overwrites first, second editor notified of conflict.
- **Rollback Creates New Version**: See FR-007 (rollback creates a new version; non-destructive).
- **Multi-Tenant Isolation**: All queries filtered by `company_id`; no cross-company visibility of JD versions.
- **Content Storage**: Small JDs (<100KB) stored inline in `JobDescriptionVersion.content`; large JDs use blob reference (S3 key).
- **Version Numbering**: Auto-increment per job (Job 1: versions 1.0, 2.0, 3.0; Job 2: versions 1.0, 2.0); not globally sequential.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: HR can update a JD and create a new version in <5 seconds (end-to-end); page updates immediately showing new version number.
- **SC-002**: 100% of interview sessions capture JD version ID at session start; zero sessions with missing `jd_version_id`.
- **SC-003**: JD version is immutable during interview session; even if JD is updated mid-interview, candidate's assessment uses original locked version.
- **SC-004**: Version history view loads and renders 20+ versions with metadata in <3 seconds.
- **SC-005**: Side-by-side diff comparison of two versions renders in <2 seconds for JDs up to 50KB.
- **SC-006**: Reverting to a previous version takes <5 seconds; new version created with correct content and metadata.
- **SC-007**: 100% of candidate status changes include `jd_version_id` in audit log (Feature 003 integration).
- **SC-008**: All JD version operations (create, update, revert, view) log entries in `JobDescriptionChangeLog` with complete actor/timestamp/reason metadata.
- **SC-009**: No cross-company data leakage; client users only see JD versions for their own company (multi-tenant isolation verified).
- **SC-010**: Feature 005 token tracking identifies "Job Description Analysis" operations and allocates tokens to correct JD version (future billing accuracy).

## Success Metrics for Future Integration

- **Feature 005 Integration**: "Cost per JD version" reporting—show which JD versions consumed most tokens.
- **Feature 006 Integration**: God Mode dashboard showing "Most Changed JDs" and "Cost impact of version iterations".
- **Compliance**: 100% of candidate assessments attributable to specific JD version; supports audit trails for hiring fairness compliance (EEOC, discrimination investigations).
- **Operational**: Time-to-revert on erroneous JD edits: <2 minutes end-to-end.
- **Product Quality**: Candidate fairness: JD versions clearly identified in ranking reports; support for fairness analysis (e.g., "Did we change criteria mid-recruitment?").
