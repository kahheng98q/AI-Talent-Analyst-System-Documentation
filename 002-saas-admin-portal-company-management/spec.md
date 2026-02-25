# Feature Specification: SaaS Admin Portal - Tenant Management

**Feature Branch**: `002-saas-admin-portal`

**Created**: 2025-11-05
**Status**: In Progress
**Input**: User description: "SaaS Admin Portal for Company, User, and System Management"

> **Portal Scope Clarification**: This feature covers **Back-Office** (Super Admin) capabilities for onboarding companies. The **Company HR Admin Portal** (`ai-talent-analyst-system-admin-portal`, port 8032) is the tenant-facing portal used by HR users — its scope falls under Feature 007 (Company Admin Self Service) and Feature 001 (AI HR Interview System). Super Admin company management is handled in `ai-talent-analyst-system-back-office`.

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### A. Super Admin - Tenant & User Management

### User Story 1 - Create New Company (Priority: P1) `✅ Implemented (Back-Office)`

As a Super Admin, I want to onboard a new customer by creating a company profile.

**Why this priority**: Critical path for business growth; without this, no new customers can be onboarded.

**Independent Test**: Can be fully tested by a Super Admin creating a company and verifying it appears in the database/list.

**Acceptance Scenarios**:

1. **Given** I am logged in as a Super Admin, **When** I navigate to "Companies" > "Add New" and submit valid details (Name, Domain, Plan), **Then** the new company should appear in the company list and be active.
2. **Given** I am on the "Add New Company" page, **When** I submit with a duplicate domain, **Then** the system should show an error preventing creation.

### User Story 2 - Create Company Admin (Priority: P1) `✅ Implemented (Back-Office)`

As a Super Admin, I want to assign an admin to a company so they can manage their team.

**Why this priority**: Essential for delegating management to the customer; reduces support burden on Super Admin.

**Independent Test**: Create a user with "Company Admin" role and verify they can log in and see company-specific settings.

**Acceptance Scenarios**:

1. **Given** a Company exists, **When** I create a new User and assign the "Company Admin" role linked to that Company, **Then** the user should be successfully created and linked.
2. **Given** the new Company Admin user, **When** they log in, **Then** they should have access to the Company Admin portal for their specific company.

### User Story 4 - Manage Client Company Super Admins (Priority: P2) `❌ Not Implemented (Back-Office)`

As a Backoffice Super Admin, I want to manage the designated "Super Admin" for a specific client company (e.g., reset their credentials, changing the owner) so that I can resolve high-level access issues or security incidents.

**Why this priority**: Critical for handling "break-glass" scenarios where a customer is locked out or an admin account is compromised.

**Independent Test**: Select a specific company and forcibly reset the password/MFA for their main admin user.

**Acceptance Scenarios**:

1. **Given** a client company where the admin has lost access, **When** I navigate to the Company Details > "Admins" tab and click "Reset Credentials", **Then** a temporary password or reset link is generated for that specific user.
2. **Given** a dispute where the original admin left the company, **When** I promote a different user in that company to "Company Super Admin" status, **Then** the new user gains full administrative rights over their tenant.

---

### Edge Cases

- **Concurrent Company Creation**: Duplicate domain checks use database constraints to prevent race conditions.
- **Orphaned Admins**: System prevents deletion of the last Company Admin for a given company.
- **Multi-Domain Companies**: Companies can have multiple domains; primary domain used for branding.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST provide a Super Admin Portal for Company and User management.
- **FR-002**: System MUST implement Multi-tenancy with strong data isolation (silo/dedicated schema logic).
- **FR-003**: System MUST support Company creation with domain validation.
- **FR-004**: System MUST allow Super Admins to create and manage Company Admin users.
- **FR-005**: System MUST integrate with Feature 004 (User Authorization Groups & Permissions) for RBAC enforcement.

### Key Entities

- **Company** (`atas.companies`): `id`, `name`, `industry`, `location`, `phone`, `email`, `created_by`, `updated_by`, `created_at`, `updated_at`, `deleted_at` (soft-delete).
- **AuthUser** (`public.auth_user`): `email`, `username`, `user_type` (enum: `client` / `backoffice`), `first_name`, `last_name`, `is_active`, `is_superuser`, `joined_date`, `last_login`. Managed via back-office `users-page.tsx`.
- **BackofficeUser** profile (`public.bo_user_profile`): Extends `AuthUser` for back-office staff. `user` (OneToOne PK), `employee_id`, `role` (admin/manager/analyst/support/viewer), `department`, `access_level` (1–5), `status` (active/inactive/suspended/pending), `phone`, `notes`, `created_by`.
- **ClientUser** profile (`public.client_user`): Extends `AuthUser` for company HR users. `user` (OneToOne PK), `company` (FK), `permission_level` (owner/admin/manager/member/viewer), `status` (active/inactive/suspended/trial/expired), `subscription_tier` (free/basic/professional/enterprise), `max_sessions`, `sessions_used`, `subscription_start_date`, `subscription_end_date`, `invited_by`, `position`.

> **Not Yet Implemented**: `Plan` as a standalone entity — subscription plans live in `atas.subscription_plans` (Feature 005). `CompanyAdmin` is not a separate model; it is expressed via `ClientUser.permission_level = 'admin'`.

### Dependencies

- **Feature 004**: User Authorization Groups & Permissions (for RBAC and permission enforcement).
- **Feature 005**: Credit & Subscription Management (for plan definitions).
- **Feature 006**: God Mode - System & AI Management (extracted Super Admin system management capabilities).
- **Feature 007**: Company Admin Self Service (extracted Company Admin user and subscription management).
- **Feature 008**: System Behavior - Limit Enforcement (extracted system-level limit enforcement logic).

## Assumptions

- **Isolation**: Data is filtered by `company_id` at the query level for all multi-tenant queries.
- **Deletion**: "Removing" a user performs a soft delete to preserve historical data and audit trails.
- **Domain Uniqueness**: Company domains must be globally unique across the platform.
- **Initial Admin**: Each company must have at least one Company Admin user at creation time.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Onboard new company in < 5 mins.
- **SC-002**: System supports 1,000 concurrent companies.
- **SC-003**: 95% of admin actions complete in < 2 seconds.
