# Feature Specification: System Behavior - Limit Enforcement

**Feature Branch**: `008-system-behavior-limit-enforcement`

**Created**: 2025-12-22
**Status**: Draft
**Input**: Extracted from Feature 002 - System-level behavior for plan limit enforcement

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Limit Enforcement (Priority: P1)

As a System, I must block actions that exceed plan limits.

**Why this priority**: Prevents revenue leakage and resource abuse.

**Independent Test**: Attempt to exceed a defined limit (e.g., candidate count) and verify blocking.

**Acceptance Scenarios**:

1. **Given** a Company with a "Starter" plan (limit 10 candidates) that already has 10 candidates, **When** they try to add an 11th candidate, **Then** the system should block the request and show an "Upgrade Required" message.
2. **Given** a Company with a "Pro" plan (limit 50 jobs), **When** they have 50 active jobs and try to create another, **Then** the system should prevent creation with a clear error message.
3. **Given** a Company at their storage limit, **When** they try to upload additional files, **Then** the system should reject the upload with limit details.

---

### Edge Cases

- **Downgrade Handling**: If a company downgrades to a plan with lower limits (e.g., 15/10 candidates), existing data remains accessible (read-only), but new creation is blocked until they are under the limit.
- **Race Conditions**: Multiple concurrent requests that would exceed limits are handled atomically; only requests within limit succeed.
- **Grace Period**: Companies get a 24-hour grace period after subscription expiration before hard limits are enforced.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST enforce plan limits at API/service layer before any database writes.
- **FR-002**: System MUST provide clear, actionable error messages when limits are exceeded.
- **FR-003**: System MUST handle downgrade scenarios by preventing new creations without deleting existing data.
- **FR-004**: System MUST implement atomic checks to prevent race conditions on limit enforcement.
- **FR-005**: System MUST integrate with Feature 005 (Credit & Subscription Management) for current plan limits.

### Key Entities

- **Plan**: Subscription tier with specific Limits (candidates, jobs, storage, etc.).
- **Company**: Tenant organization with current usage counts.
- **LimitCheck**: Service/middleware that validates actions against plan limits.
- **UsageMetric**: Real-time tracking of resource consumption per company.

### Dependencies

- **Feature 002**: SaaS Admin Portal (for company and plan data).
- **Feature 005**: Credit & Subscription Management (for plan definitions and limits).

## Assumptions

- **Limit Types**: Plans define limits for: candidates, jobs, active users, storage (GB), AI tokens/month.
- **Check Timing**: Limits checked before action execution, not after.
- **Soft Limits**: Warning notifications sent at 80% of limit; hard block at 100%.
- **Caching**: Current usage and limits cached for 5 minutes to reduce database load.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: 100% of limit-exceeding actions blocked before execution.
- **SC-002**: Limit check adds < 50ms latency to request processing.
- **SC-003**: Zero race conditions resulting in limit overages.
- **SC-004**: Error messages include current usage, limit, and upgrade CTA.
