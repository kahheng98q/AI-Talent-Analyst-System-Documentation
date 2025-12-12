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

As a Company Admin (or Super Admin), I want to add/remove users from authorization groups so that I can control their access levels.

**Why this priority**: Core functionality for managing team permissions dynamically.

**Independent Test**: Add a user to a group and verify they gain the associated permissions; remove them and verify permissions are revoked.

**Acceptance Scenarios**:

1. **Given** I have a user "Alice" and a group "Hiring Managers", **When** I add Alice to the group, **Then** Alice should gain all permissions associated with "Hiring Managers".
2. **Given** Alice is in the "Hiring Managers" group, **When** I remove her from the group, **Then** she should lose those permissions immediately.
3. **Given** Alice belongs to both "Hiring Managers" and "Interviewers" groups, **When** she accesses the system, **Then** she should have the combined (union) of permissions from both groups.

---

### User Story 4 - View and Audit Permission Assignments (Priority: P2)

As an Admin, I want to view which permissions a specific user has (across all their groups) so that I can audit access levels.

**Why this priority**: Important for compliance and troubleshooting access issues.

**Independent Test**: Select a user and view their effective permissions list aggregated from all groups.

**Acceptance Scenarios**:

1. **Given** a user "Bob" belongs to multiple groups, **When** I view Bob's user details, **Then** I should see a complete list of his effective permissions.
2. **Given** I want to understand group membership, **When** I view a group's details, **Then** I should see all users assigned to that group.

---

### User Story 5 - Define Custom Permissions (Priority: P2)

As a Super Admin, I want to define new permissions (e.g., `report.export`, `invoice.manage`) so that I can extend the authorization system as the platform grows.

**Why this priority**: Allows system to scale with new features without code changes.

**Independent Test**: Create a new permission, assign it to a group, and verify it's enforced in the application.

**Acceptance Scenarios**:

1. **Given** I am in the Permissions Management section, **When** I create a new permission `analytics.export`, **Then** it should appear in the available permissions list.
2. **Given** the new permission exists, **When** I assign it to a group, **Then** users in that group should be able to access the protected feature.

---

### User Story 6 - Prevent Orphaned Permissions (Priority: P2)

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

- **Group**: Named collection of permissions. Has `name`, `description`, `company_id` (nullable for global groups), `is_system` (prevents deletion).
- **Permission**: Granular access right. Has `resource`, `action` (e.g., `candidate.view`, `interview.delete`).
- **UserGroup**: Join table linking Users to Groups (many-to-many).
- **GroupPermission**: Join table linking Groups to Permissions (many-to-many).
- **PermissionAuditLog**: Records changes to group assignments and permission grants (timestamp, admin_user_id, action).

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
