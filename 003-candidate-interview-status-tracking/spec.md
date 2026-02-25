# Feature Specification: Candidate Interview Status Tracking

**Feature Branch**: `003-interview-status-tracking`  
**Created**: 2025-12-07  
**Status**: Draft  
**Input**: User description: "add status of the candidate interview status, so the HR can trace back"

> **Admin Portal Note**: The admin portal currently tracks **Interview Session** statuses (Upcoming, In Progress, Completed, Expired, Cancelled) as part of Feature 001. Candidate-level status tracking (this feature) has NOT been implemented in the admin portal yet.

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
-->

### User Story 1 - View and Update Candidate Status (Priority: P1) `❌ Not Implemented`

As an HR personnel, I want to see the current interview status of a candidate and be able to update it, so that I can track their progress in the hiring pipeline.

**Why this priority**: Core functionality. Without this, there is no status tracking.

**Independent Test**: Can be fully tested by creating a candidate, verifying the default status, and then changing the status to a new value.

**Acceptance Scenarios**:

1. **Given** a new candidate is created, **When** I view the candidate list, **Then** the status should be "New" (or default).
2. **Given** a candidate with status "New", **When** I change the status to "Interview Scheduled", **Then** the status is updated and reflected in the UI.

---

### User Story 2 - Trace Status History (Priority: P2) `❌ Not Implemented`

As an HR personnel, I want to see the history of status changes for a candidate, including who changed it and when, so that I can audit the recruitment process.

**Why this priority**: "Trace back" was explicitly requested. It adds accountability and context.

**Independent Test**: Change a candidate's status multiple times, then view the history log to verify all changes are recorded chronologically.

**Acceptance Scenarios**:

1. **Given** a candidate who has had status changes, **When** I view the candidate's details/history, **Then** I see a list of past statuses with timestamps.

---

### Edge Cases

- What happens when two HR users try to update status simultaneously?
- What happens if a status transition is invalid (e.g., from "Hired" back to "New")? (State machine logic might be needed eventually, but maybe simple for now).
- How does the system handle status updates for deleted candidates?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST allow defining a set of candidate interview statuses (e.g., New, Screening, Interviewing, Offer, Hired, Rejected).
- **FR-002**: System MUST store the current status for each candidate.
- **FR-003**: System MUST provide an interface for HR to update the status of a candidate.
- **FR-004**: System MUST record a history log of status changes (timestamp, old status, new status, modified by).
- **FR-005**: System MUST display the status history to the user.

### Key Entities

- **Candidate**: Needs a new field `status`.
- **CandidateStatusHistory**: New entity/table to link Candidate, Status, Timestamp, and User.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: HR can update a candidate's status in less than 3 clicks.
- **SC-002**: Status history is accurate and persists 100% of changes.
- **SC-003**: 100% of candidates have a valid status assigned.
