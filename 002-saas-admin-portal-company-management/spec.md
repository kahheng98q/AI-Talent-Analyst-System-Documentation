# Feature Specification: SaaS Admin Portal & God Mode

**Feature Branch**: `002-saas-admin-portal`
**Created**: 2025-11-05
**Status**: Draft
**Input**: User description: "SaaS Admin Portal for Company, User, and System Management"

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### A. Super Admin - Tenant & User Management

### User Story 1 - Create New Company (Priority: P1)

As a Super Admin, I want to onboard a new customer by creating a company profile.

**Why this priority**: Critical path for business growth; without this, no new customers can be onboarded.

**Independent Test**: Can be fully tested by a Super Admin creating a company and verifying it appears in the database/list.

**Acceptance Scenarios**:

1. **Given** I am logged in as a Super Admin, **When** I navigate to "Companies" > "Add New" and submit valid details (Name, Domain, Plan), **Then** the new company should appear in the company list and be active.
2. **Given** I am on the "Add New Company" page, **When** I submit with a duplicate domain, **Then** the system should show an error preventing creation.

### User Story 2 - Create Company Admin (Priority: P1)

As a Super Admin, I want to assign an admin to a company so they can manage their team.

**Why this priority**: Essential for delegating management to the customer; reduces support burden on Super Admin.

**Independent Test**: Create a user with "Company Admin" role and verify they can log in and see company-specific settings.

**Acceptance Scenarios**:

1. **Given** a Company exists, **When** I create a new User and assign the "Company Admin" role linked to that Company, **Then** the user should be successfully created and linked.
2. **Given** the new Company Admin user, **When** they log in, **Then** they should have access to the Company Admin portal for their specific company.

### User Story 3 - Manage User Groups (Priority: P2)

As a Super Admin, I want to organize users into groups for easier permission management.

**Why this priority**: Improves manageability of large user bases and granular permission control.

**Independent Test**: Create a group, add users, and verify group membership.

**Acceptance Scenarios**:

1. **Given** I am a Super Admin, **When** I create a "Group" (e.g., "Beta Testers") and add users to it, **Then** the users should be listed as members of that group.
2. **Given** a group with members, **When** I view the group details, **Then** I should see the correct list of assigned users.

### User Story 4 - Impersonate Tenant (God Mode) (Priority: P1)

As a Super Admin, I want to log in as a specific user to debug issues exactly as they see them.

**Why this priority**: Critical for support and debugging; allows rapid resolution of user-reported issues.

**Independent Test**: Trigger "Login As" and verify the session context changes to the target user.

**Acceptance Scenarios**:

1. **Given** I am a Super Admin viewing a user record, **When** I click "Login As", **Then** I should be redirected to the app authenticated as that user.
2. **Given** I am impersonating a user, **When** I look at the interface, **Then** I should see a persistent "Impersonating" banner with an "Exit" button.
3. **Given** I am impersonating, **When** I click "Exit", **Then** I should be returned to the Super Admin session.

### B. Super Admin - System & AI Management (God Mode)

### User Story 5 - System Dashboard (Priority: P2)

As a Super Admin, I want a high-level view of system health and revenue.

**Why this priority**: Provides visibility into business KPIs and system stability.

**Independent Test**: Load the dashboard and verify data accuracy against known metrics.

**Acceptance Scenarios**:

1. **Given** I am on the Super Admin home page, **When** the dashboard loads, **Then** I should see current MRR, Token Burn Rate, and Live AI Success Rate.
2. **Given** new tenants have joined, **When** I view the "Tenant Growth" chart, **Then** it should reflect the recent sign-ups.

### User Story 6 - AI Configuration (Priority: P2)

As a Super Admin, I want to update system prompts and switch models without code deploys.

**Why this priority**: Allows rapid iteration on AI behavior and cost management without engineering intervention.

**Independent Test**: Change a prompt or model setting and verify the AI output/behavior changes in a new session.

**Acceptance Scenarios**:

1. **Given** I am in AI Configuration, **When** I edit the "Interviewer Persona" prompt and save, **Then** subsequent AI interactions should use the new persona.
2. **Given** I want to change the underlying model, **When** I switch the dropdown from "GPT-4" to "Claude 3.5", **Then** the system should immediately start using the new model for new requests.

### User Story 7 - Cost Auditing (Priority: P3)

As a Super Admin, I want to identify which companies are spending the most on AI tokens.

**Why this priority**: Necessary for identifying abuse, optimizing costs, and potential upselling.

**Independent Test**: Generate some token usage and verify it appears in the "Top Spenders" list.

**Acceptance Scenarios**:

1. **Given** multiple companies using AI, **When** I view the "Top Spenders" list, **Then** it should be sorted by total cost/token usage.
2. **Given** a specific company, **When** I drill down into their usage, **Then** I should see detailed logs of their token consumption.

### C. Company Admin - Self Service

### User Story 8 - Manage HR Users (Priority: P2)

As a Company Admin, I want to manage my team's access.

**Why this priority**: Basic administration capability required for any B2B SaaS.

**Independent Test**: Add/remove a user as a Company Admin and verify access.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin, **When** I view my user list, **Then** I should see all users belonging to my company.
2. **Given** an employee has left, **When** I remove/deactivate their user account, **Then** they should no longer be able to log in.

### User Story 9 - Invite Users (Priority: P2)

As a Company Admin, I want to invite colleagues via email.

**Why this priority**: Streamlines user onboarding and reduces admin friction.

**Independent Test**: Send an invite and verify the email receipt and registration flow.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin, **When** I send an invitation to `colleague@example.com`, **Then** the user should receive an email with a unique registration link.
2. **Given** an uninvited user, **When** they try to register, **Then** they should be blocked (assuming closed registration).
3. **Given** a user with an invite link, **When** they complete the form, **Then** they should be able to set a password and join the company.

### User Story 10 - Manage Subscription (Priority: P1)

As a Company Admin, I want to upgrade my plan to access more features.

**Why this priority**: Directly drives revenue; essential for monetization.

**Independent Test**: Simulate an upgrade flow and verify plan limits change.

**Acceptance Scenarios**:

1. **Given** I am on a "Starter" plan, **When** I click "Upgrade" and complete the Stripe Checkout, **Then** my plan should be updated to "Pro".
2. **Given** the upgrade is successful, **When** I check my limits (e.g., max candidates), **Then** they should reflect the new plan's allowances immediately.

### D. System Behavior

### User Story 11 - Limit Enforcement (Priority: P1)

As a System, I must block actions that exceed plan limits.

**Why this priority**: Prevents revenue leakage and resource abuse.

**Independent Test**: Attempt to exceed a defined limit (e.g., candidate count) and verify blocking.

**Acceptance Scenarios**:

1. **Given** a Company with a "Starter" plan (limit 10 candidates) that already has 10 candidates, **When** they try to add an 11th candidate, **Then** the system should block the request and show an "Upgrade Required" message.

---

### Edge Cases

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right edge cases.
-->

- What happens when a Company Admin tries to delete their own account? (Should be prevented or warn about losing access).
- How does the system handle a Stripe webhook failure during an upgrade? (Should retry or maintain old state until confirmed).
- What happens if a Super Admin impersonates a user who is currently suspended?
- How does the system handle concurrent edits to Global AI Configurations? (Last write wins vs locking).

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST provide a Super Admin Portal for Company, User, and Group management.
- **FR-002**: System MUST enforce Role-Based Access Control (Super Admin, Company Admin, HR User).
- **FR-003**: System MUST implement Multi-tenancy with strong data isolation (silo/dedicated schema logic).
- **FR-004**: System MUST allow Super Admins to Impersonate any tenant user.
- **FR-005**: System MUST provide a Dashboard for MRR, Token Usage, and AI Health monitoring.
- **FR-006**: System MUST allow Dynamic AI Configuration (Prompts, Models) via UI without deployment.
- **FR-007**: System MUST handle Subscription management (Stripe integration) and Limit Enforcement.
- **FR-008**: System MUST support User Invitation via email.

### Key Entities

- **Company**: Tenant organization.
- **User**: System user with Role.
- **Group**: User collection for permissions.
- **Plan**: Subscription tier with Limits.
- **Subscription**: Link between Company and Plan.
- **Invitation**: Pending user access.
- **SystemSetting**: Global configs (e.g., active model).
- **TokenUsage**: Cost tracking records.

## Assumptions

- **Stripe**: Used for all billing.
- **Isolation**: Data is filtered by `company_id` at the query level.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Onboard new company in < 5 mins.
- **SC-002**: Support tickets requiring login resolved 30% faster via Impersonation.
- **SC-003**: System supports 1,000 concurrent companies.
- **SC-004**: 95% of admin actions complete in < 2 seconds.
