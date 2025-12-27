# Feature Specification: User Authorization Groups & Permissions

**Feature Branch**: `004-user-authorization-groups-permissions`  
**Created**: 2025-12-12  
**Status**: Draft  
**Input**: User description: "User Authorization, Groups, and Permissions Management System"

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Manage Authorization Groups & Permissions (Priority: P1)

As a Super Admin (or Company Admin), I want to create groups with specific permission sets (e.g., "Interviewer", "Hiring Manager") so that I can control access to sensitive features and data.

**Why this priority**: Essential for security and ensuring users only see what they are supposed to see (Principle of Least Privilege).

**Independent Test**: Create a restricted group, add a user, and verify they cannot access forbidden routes/actions.

**Acceptance Scenarios**:

1. **Given** I am an Admin, **When** I create a new Group "Junior Recruiters" and select permissions `[candidate.view, interview.create]`, **Then** the group should be saved with those specific rights.
2. **Given** the "Junior Recruiters" group does NOT have `salary.view` permission, **When** a user in this group views a candidate profile, **Then** the salary field should be hidden or masked.
3. **Given** a user belongs to multiple groups, **When** they access a resource, **Then** they should receive the union of all permissions (additive access).

---

### User Story 2 - Manage Backoffice Staff & Permissions (Priority: P1)

As a Backoffice Super Admin, I want to manage the accounts and permissions of my internal team (e.g., creating Support Agents, Sales staff, Developers) so they can access the admin portal with appropriate restrictions.

**Why this priority**: Necessary to scale the internal team while maintaining security (not everyone should have God Mode).

**Independent Test**: Create a "Support Agent" user with read-only access to customer data but no ability to delete, and verify their restricted view.

**Acceptance Scenarios**:

1. **Given** I am the main Super Admin, **When** I create a new internal user "John Doe" and assign the "Support" role, **Then** John should be able to log in to the Backoffice.
2. **Given** the "Support" role has `impersonation.allow` but `company.delete` denied, **When** John logs in, **Then** he can impersonate users but the "Delete Company" button is hidden/disabled.

---

### User Story 3 - Assign Users to Groups (Priority: P1)

As a Company Admin (or Super Admin), I want to add/remove users from authorization groups so that I can control their access levels, with automatic validation to prevent cross-company or incompatible user type assignments.

**Why this priority**: Core functionality for managing team permissions dynamically with built-in security to prevent misconfiguration.

**Independent Test**: Add a user to a group and verify they gain the associated permissions; remove them and verify permissions are revoked. Attempt invalid assignments (wrong company/user type) and verify they are blocked.

**Acceptance Scenarios**:

1. **Given** I have a user "Alice" and a group "Hiring Managers", **When** I add Alice to the group, **Then** Alice should gain all permissions associated with "Hiring Managers".
2. **Given** Alice is in the "Hiring Managers" group, **When** I remove her from the group, **Then** she should lose those permissions immediately.
3. **Given** Alice belongs to both "Hiring Managers" and "Interviewers" groups, **When** she accesses the system, **Then** she should have the combined (union) of permissions from both groups.
4. **Given** I am a Company Admin for "Acme Corp" with a client user "Bob" from "TechStart Inc", **When** I attempt to assign Bob to my company's group, **Then** the system should reject the assignment with an error message about company mismatch.
5. **Given** a group "Support Agents" has `applicable_user_type='backoffice'`, **When** I attempt to assign a client user to this group, **Then** the system should reject the assignment with an error about incompatible user type.
6. **Given** a global group (company=None) "Platform Admins", **When** I attempt to assign a client user to this group, **Then** the system should reject the assignment as client users cannot belong to global groups.

---

### User Story 4 - Company Data Isolation (Priority: P1)

As a Client User, I should only be able to access groups, permissions, and assignments within my own company, ensuring complete data isolation from other client companies.

**Why this priority**: Critical security requirement to prevent data breaches and maintain multi-tenant isolation.

**Independent Test**: Create users from different companies and verify they cannot view or modify each other's groups/assignments.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin for "Acme Corp", **When** I create a new group "Sales Team", **Then** the group should automatically be scoped to my company.
2. **Given** groups exist for both "Acme Corp" and "TechStart Inc", **When** I view the groups list as an Acme Corp user, **Then** I should only see Acme Corp's groups (and global groups if applicable).
3. **Given** I am a Company Admin for "Acme Corp", **When** I attempt to update a group from "TechStart Inc", **Then** the system should reject the request with a permission denied error.
4. **Given** I have `candidate.view` permission through a group in my company, **When** I access candidate data, **Then** I should only see candidates associated with my company.
5. **Given** I am a client user checking permissions for a resource, **When** the system evaluates my permissions, **Then** it should only consider permissions from groups belonging to my company.

---

### User Story 5 - Cross-Company Access for Backoffice (Priority: P1)

As a Backoffice User (Support/Admin), I want to access data across all client companies when granted cross-company permissions, while respecting company boundaries for standard permissions.

**Why this priority**: Essential for internal operations like customer support, analytics, and platform administration.

**Independent Test**: Grant a backoffice user cross-company permissions and verify they can access data from all companies. Verify standard permissions remain company-scoped.

**Acceptance Scenarios**:

1. **Given** I am a Support Agent with `ticket.view` permission marked as `is_cross_company=True`, **When** I access the tickets list, **Then** I should see tickets from all client companies.
2. **Given** I am a Support Agent with `candidate.view` permission (not cross-company), **When** I access candidates, **Then** I should only see candidates within the company context I'm operating in.
3. **Given** I am a Backoffice Admin, **When** I view the groups list, **Then** I should see groups from all companies plus global groups.
4. **Given** I have `analytics.export` with `is_cross_company=True`, **When** I generate a report, **Then** the report should include aggregated data from all companies.
5. **Given** I am a backoffice user assigning users to groups, **When** I assign a client user to a group, **Then** the system should validate company matching even though I have cross-company access.

---

### User Story 6 - View and Audit Permission Assignments (Priority: P2)

As an Admin, I want to view which permissions a specific user has (across all their groups) so that I can audit access levels and understand effective permissions including user type and company restrictions.

**Why this priority**: Important for compliance and troubleshooting access issues.

**Independent Test**: Select a user and view their effective permissions list aggregated from all groups, including applicable user type and company scope.

**Acceptance Scenarios**:

1. **Given** a user "Bob" belongs to multiple groups, **When** I view Bob's user details, **Then** I should see a complete list of his effective permissions with company scope indicators.
2. **Given** I want to understand group membership, **When** I view a group's details, **Then** I should see all users assigned to that group with assignment metadata (assigned_by, assigned_at, expires_at).
3. **Given** a group has `applicable_user_type='client'`, **When** I view the group details, **Then** I should see the user type restriction clearly indicated.
4. **Given** a permission has `is_cross_company=True`, **When** I view permission details, **Then** I should see a clear indicator that this grants cross-company access.

---

### User Story 7 - Define Custom Permissions (Priority: P2)

As a Super Admin, I want to define new permissions (e.g., `report.export`, `invoice.manage`) with appropriate user type restrictions and cross-company flags so that I can extend the authorization system as the platform grows.

**Why this priority**: Allows system to scale with new features without code changes.

**Independent Test**: Create a new permission with specific attributes, assign it to a group, and verify it's enforced correctly in the application.

**Acceptance Scenarios**:

1. **Given** I am in the Permissions Management section, **When** I create a new permission `analytics.export` with `applicable_user_type='backoffice'` and `is_cross_company=True`, **Then** it should appear in the available permissions list with those attributes.
2. **Given** the new permission exists, **When** I assign it to a group, **Then** users in that group should be able to access the protected feature.
3. **Given** I create a permission with `applicable_user_type='client'`, **When** I attempt to assign it to a backoffice-only group, **Then** the system should allow it but the permission won't be effective for backoffice users.
4. **Given** I create a cross-company permission, **When** a backoffice user has this permission, **Then** they should be able to access resources across all companies for that specific resource.action.

---

### User Story 8 - Prevent Orphaned Permissions (Priority: P2)

As a System, I must ensure that deleted groups don't leave users without necessary access.

**Why this priority**: Prevents accidental lockouts and maintains system integrity.

**Independent Test**: Delete a group and verify users are warned or permissions are reassigned.

**Acceptance Scenarios**:

1. **Given** a group "Temp Contractors" has users assigned, **When** I attempt to delete the group, **Then** the system should warn me about affected users.
2. **Given** I confirm deletion of a group, **When** the deletion completes, **Then** users formerly in that group should lose only those specific permissions.

---

### Edge Cases

- **Circular Dependencies**: What happens if permissions reference each other?
- **Permission Conflicts**: If a user has `candidate.read` from one group but `candidate.deny` from another, which takes precedence? (Default: Allow-first/union strategy)
- **Super Admin Bypass**: Should Super Admins bypass all permission checks, or should they also be subject to groups?
- **Deletion of Critical Groups**: What happens if someone tries to delete the "Company Admin" group that all admins belong to?
- **Empty Groups**: Should groups with no users be automatically deleted, or should they persist?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST support creation of Authorization Groups with custom names and descriptions.
- **FR-002**: System MUST allow assigning multiple Permissions to a Group.
- **FR-003**: System MUST allow assigning Users to multiple Groups (many-to-many relationship).
- **FR-004**: System MUST calculate effective permissions as the union of all groups a user belongs to (additive model).
- **FR-005**: System MUST enforce permissions at the API/controller level before allowing actions.
- **FR-006**: System MUST provide a UI to view, create, edit, and delete Groups.
- **FR-007**: System MUST provide a UI to assign/unassign Users to Groups.
- **FR-008**: System MUST support defining Custom Permissions dynamically.
- **FR-009**: System MUST audit permission changes (who added/removed which permission, when).
- **FR-010**: System MUST prevent deletion of Groups that are marked as "System Critical" (e.g., Company Admin, Super Admin).

### Key Entities

- **auth_group**: Stores authorization group definitions (e.g., Admins, Editors). Columns: `id`, `name`.
- **auth_permission**: Defines granular actions (add/change/delete/etc.) tied to models. Columns: `id`, `name`, `content_type_id` → `django_content_type.id`, `codename` (maps to `resource.action` like `candidate.view`).
- **auth_group_permissions**: Junction table linking groups to permissions. Columns: `id`, `group_id` → `auth_group.id`, `permission_id` → `auth_permission.id`.
- **django_content_type**: Registry of installed models used for permission scoping. Columns: `id`, `app_label`, `model`.
- **django_admin_log**: Audit log of admin actions (incl. group/permission changes). Columns: `id`, `action_time`, `object_id`, `object_repr`, `action_flag`, `change_message`, `content_type_id` → `django_content_type.id`, `user_id`.

### Implementation Notes (Current Build)

- We use custom models `AuthGroup`, `AuthPermission`, and `UserGroupAssignment` instead of Django’s built-in `auth_group`/`auth_permission`/`auth_group_permissions` tables.
- Permissions are modeled as `resource.action` fields (e.g., `candidate.view`) rather than `content_type + codename`, to simplify additive checks and avoid tight coupling to Django ContentType.
- Groups are company-scoped (optional `company_id`), marked by `is_system_critical` for system groups, and carry `applicable_user_type`, descriptions, and audit fields to satisfy FR-009 and FR-010 and the multi-tenant assumption.
- Assignments are explicit (`UserGroupAssignment`) with `assigned_by`, `assigned_at`, `expires_at`, `notes`, and `is_active` to support auditing (FR-009) and temporary memberships.
- Rationale: aligns with FR-001..FR-010 and the “Company Isolation” assumption while keeping permission checks performant and additive. If alignment with Django built-in tables is later required, we can adapt the serializers/services to bridge to core auth tables.

#### Multi-Tenant Security Enhancements

**System Groups:**

- Groups marked with `is_system_critical=True` are system groups that cannot be deleted or modified (except permissions).
- System groups are created during system initialization (e.g., "Super Admin", "Company Admin").
- This replaces the former `group_type` field with a simpler boolean flag approach.
  **User Type Isolation:**

- `AuthGroup.applicable_user_type` (choices: `client`, `backoffice`, `both`) restricts which user types can be assigned to a group.
- `AuthGroup.can_assign_user(user)` validation method enforces:
  - User type matches group's `applicable_user_type`
  - Client users cannot be assigned to global groups (`company=None`)
  - Client users can only join groups in their own company
- `AuthPermission.applicable_user_type` restricts which user types can receive a permission.

**Company-Scoped Access:**

- `PermissionService.has_permission()` accepts `company_id` parameter for context-specific checks.
- Client users' permissions are filtered to only include groups from their company.
- `AuthGroupViewSet` enforces company isolation: client users can only create/update groups in their own company.
- `UserGroupAssignmentViewSet.assign_user()` validates company matching for client users.

**Cross-Company Access for Backoffice:**

- `AuthPermission.is_cross_company` flag enables backoffice users to access resources across all companies.
- When `True`, backoffice users with this permission bypass company filtering.
- Use case: Support agents viewing tickets across all client companies, analytics for all customers.

**Security Benefits:**

- Prevents client users from accessing other companies' data
- Prevents user type mismatches (e.g., client users in backoffice-only groups)
- Enables controlled cross-company access for internal staff
- Maintains additive permission model while enforcing isolation boundaries

## Assumptions

- **Additive Permissions**: Users accumulate permissions from all their groups (no deny rules by default).
- **Company Isolation**: Company Admins can only create/manage groups within their own company; Super Admins can manage global groups.
- **System Groups**: Some groups (e.g., "Super Admin", "Company Admin") are created by default and cannot be deleted.
- **Permission Format**: Permissions follow the format `resource.action` (e.g., `candidate.view`, `job.create`, `report.export`).

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Admins can create a new group and assign permissions in < 2 minutes.
- **SC-002**: Permission enforcement blocks 100% of unauthorized access attempts.
- **SC-003**: Effective permissions for a user are calculated in < 100ms.
- **SC-004**: Audit logs capture 100% of permission changes with accurate timestamps and actor identification.
- **SC-005**: System supports at least 50 custom groups per company without performance degradation.
