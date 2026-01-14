# Copilot Instructions: AI-Talent-Analyst-System-Documentation

## Project Overview

This repository documents a comprehensive **AI-powered HR talent management platform** with four interdependent feature areas:

- **Feature 001**: AI-HR Interview System (core interview workflow)
- **Feature 002**: SaaS Admin Portal (multi-tenant governance, god mode, subscription management)
- **Feature 003**: Interview Status Tracking (candidate pipeline visibility)
- **Feature 004**: User Authorization Groups & Permissions (RBAC, groups, granular permissions)

All specs follow a **structured template** focusing on **independently testable user stories with Given-When-Then acceptance scenarios** and explicit priority levels (P1/P2/P3).

## Documentation Structure & Patterns

### Specification Format

Each feature spec (in numbered folders) follows `../Template/spec-template.md`:

1. **Metadata**: Feature branch name (`###-feature-name`), status, input requirements
2. **User Scenarios**: Prioritized stories (P1 = MVP-critical, P2 = enhancement)
3. **Acceptance Criteria**: Given-When-Then scenarios (GWT format)
4. **Requirements & Entities**: Functional requirements and data models
5. **Success Metrics**: Measurable outcomes (SLA response times, data accuracy)

### Key File References

- `../001-ai-hr-interview-system/spec.md` → Interview workflow, AI model integration, resume parsing
- `../002-saas-admin-portal-company-management/spec.md` → Multi-tenant management, God Mode (super admin impersonation), company/user lifecycle, subscription management
- `../003-candidate-interview-status-tracking/spec.md` → Status pipeline management, audit history, state validation
- `../004-user-authorization-groups-permissions/spec.md` → RBAC system, groups, granular permissions, permission auditing

## Architecture & Data Flows

### Conceptual Layers

1. **Multi-Tenant Core**: Companies own Jobs, Candidates, Interviews. Authorization via Groups + Permissions.
2. **Interview Execution**: HR creates Jobs → invites Candidates → Candidates upload resume → AI conducts interview → System ranks by competency scores
3. **Admin Governance**: Super Admin manages Companies, Company Admins manage their users/groups/permissions. "God Mode" allows impersonation for debugging.

### Integration Points

- **AI Model**: Feature 001 requires third-party API integration (OpenAI/Gemini) for question generation, resume scoring, competency assessment
- **Authentication**: Candidates require email verification before accessing unique interview links
- **Persistence**: Interview responses, status history, audit logs stored indefinitely (Feature 001/003)

## Project-Specific Conventions

### User Story Writing

- **Mandatory format**: Priority + Independent Test + 2+ Given-When-Then scenarios
- **P1 stories are MVP slices** → if implemented alone, deliver standalone value
- **Why field required** → explains business justification for priority level
- **Edge cases noted** → e.g., concurrent status updates, invalid state transitions (Feature 003)

### Predefined Design Decisions

- **Competencies**: Hardcoded and universal across all jobs (not configurable per job) — Feature 001
- **Resume Upload**: Mandatory before interview start; failure → interview terminates and must restart — Feature 001
- **Interview Links**: Secured via candidate email verification (no anonymous access) — Feature 001
- **Data Retention**: No automatic deletion; manual HR manager deletion only — Feature 001
- **Status History**: Records timestamp, old/new status, modified-by user — Feature 003

### Authorization Model (Feature 004)

- **Groups** = permission sets (e.g., "Interviewer" group has `[interview.create, interview.view]`)
- **Multi-group users**: Additive permissions (union of all group permissions)
- **Permission Format**: `resource.action` (e.g., `candidate.view`, `job.create`, `report.export`)
- **Company Admin**: Can manage groups/users within their company; Super Admin manages all companies
- **System Groups**: Default groups (Super Admin, Company Admin) cannot be deleted
- **Enforcement**: Permissions checked at API/controller level before actions

### God Mode (Feature 002)

- **Super Admin**: Can impersonate any user via "Login As" button
- **Impersonation UI**: Persistent banner with "Exit" button during impersonation
- **Audit Logging**: All impersonation actions logged with Super Admin actor ID

## Development & Testing Workflow

### Running Specifications

Specs are **documentation-first**, not executable code. To implement a feature:

1. Select a **P1 user story** from the corresponding spec
2. Break it into **task-size work items** based on entities & requirements sections
3. Write tests following the **Given-When-Then scenarios** as test cases
4. Verify your implementation against **Success Metrics** (e.g., "update status in <3 clicks", "100% audit log accuracy")

### Spec Evolution

- Feature branch naming: `###-feature-name` (e.g., `001-ai-hr-interview-system`)
- Update **Status** field: Draft → In Progress → Complete
- Clarifications section: Add Q&A from stakeholder discussions
- Preserve original user input in **Input** field for traceability

## Common Questions During Implementation

**Q: How do I structure database migrations?**  
A: Review your feature's Requirements → Key Entities section. E.g., Feature 003 needs `Candidate.status` field + `CandidateStatusHistory` table.

**Q: What permissions does a "Hiring Manager" have?**  
A: Check Feature 004's user stories and permissions examples. Define custom groups per Feature 004 User Story 1 (Manage Authorization Groups & Permissions).

**Q: How are permissions enforced across features?**  
A: Feature 004 provides RBAC foundation. Each feature references specific permissions (e.g., Feature 001 checks `interview.create`, Feature 003 checks `candidate.status.update`).

**Q: Should status transitions be validated (state machine)?**  
A: Feature 003 notes this as a future enhancement. Start with simple updates; add state machine validation later if needed.

**Q: How does Feature 001 AI integration work?**  
A: Third-party API calls (OpenAI/Gemini) for: (1) question generation from job description, (2) resume scoring against JD, (3) competency-based scoring of interview responses.

## Git & Documentation Maintenance

- Keep this file updated when architecture patterns change or new conventions emerge
- Reference specific lines/sections when creating implementation issues
- Use feature branch naming convention consistently across all repos
- Link implementation PRs back to spec file sections for traceability
