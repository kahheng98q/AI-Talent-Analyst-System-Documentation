# Feature Specification: User Authorization Groups & Permissions

**Feature Branch**: `004-user-authorization-groups-permissions`  
**Created**: 2025-12-12  
**Status**: Draft  
**Input**: User description: "User Authorization, Groups, and Permissions Management System"

> **Status**: Implemented in `ai-talent-analyst-system-back-office`. The back-office portal provides: Groups management (`groups-page.tsx`), Permissions management (`permissions-page.tsx`), User management (`users-page.tsx`), User Assignments (`user-assignments-page.tsx`), and UI Route Permission mapping (`route-permissions-page.tsx`). Backend RBAC models (`AuthGroup`, `AuthPermission`, `UserGroupAssignment`, `UIRoutePermission`, `PermissionAuditLog`) are fully defined in the Django API.

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Manage Authorization Groups & Permissions (Priority: P1) `✅ Implemented (Back-Office)`

As a Super Admin (or Company Admin), I want to create groups with specific permission sets (e.g., "Interviewer", "Hiring Manager") so that I can control access to sensitive features and data.

**Why this priority**: Essential for security and ensuring users only see what they are supposed to see (Principle of Least Privilege).

**Independent Test**: Create a restricted group, add a user, and verify they cannot access forbidden routes/actions.

**Acceptance Scenarios**:

1. **Given** I am an Admin, **When** I create a new Group "Junior Recruiters" and select permissions `[candidate.view, interview.create]`, **Then** the group should be saved with those specific rights.
2. **Given** the "Junior Recruiters" group does NOT have `salary.view` permission, **When** a user in this group views a candidate profile, **Then** the salary field should be hidden or masked.
3. **Given** a user belongs to multiple groups, **When** they access a resource, **Then** they should receive the union of all permissions (additive access).

---

### User Story 2 - Manage Backoffice Staff & Permissions (Priority: P1) `✅ Implemented (Back-Office)`

As a Backoffice Super Admin, I want to manage the accounts and permissions of my internal team (e.g., creating Support Agents, Sales staff, Developers) so they can access the admin portal with appropriate restrictions.

**Why this priority**: Necessary to scale the internal team while maintaining security (not everyone should have system-level administrative capabilities).

**Independent Test**: Create a "Support Agent" user with read-only access to customer data but no ability to delete, and verify their restricted view.

**Acceptance Scenarios**:

1. **Given** I am the main Super Admin, **When** I create a new internal user "John Doe" and assign the "Support" role, **Then** John should be able to log in to the Backoffice.
2. **Given** the "Support" role has `ticket.view` but not `company.delete`, **When** John logs in, **Then** he can view support tickets but the "Delete Company" button is hidden/disabled.

---

### User Story 3 - Assign Users to Groups (Priority: P1) `🔶 Partially Implemented (Back-Office)`

> **Note**: `user-assignments-page.tsx` renders the assignment UI and reads groups/users from the API, but the save/sync back to the API is still a TODO (`handleUsersChange` only updates local state).

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

### User Story 4 - Company Data Isolation (Priority: P1) `✅ Implemented (Back-Office)`

> **Note**: Enforced at the API model level via `AuthGroup.can_assign_user()` and `AuthUser.get_groups()`/`get_permissions()` which automatically scope queries to the user's company for client users.

As a Client User, I should only be able to access groups, permissions, and assignments within my own company, ensuring complete data isolation from other client companies. Cross-company access is exclusively reserved for backoffice users.

**Why this priority**: Critical security requirement to prevent data breaches and maintain multi-tenant isolation. Client users must never access other companies' data under any circumstances.

**Independent Test**: Create users from different companies and verify they cannot view or modify each other's groups/assignments.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin for "Acme Corp", **When** I create a new group "Sales Team", **Then** the group should automatically be scoped to my company.
2. **Given** groups exist for both "Acme Corp" and "TechStart Inc", **When** I view the groups list as an Acme Corp user, **Then** I should only see Acme Corp's groups (and global groups if applicable).
3. **Given** I am a Company Admin for "Acme Corp", **When** I attempt to update a group from "TechStart Inc", **Then** the system should reject the request with a permission denied error.
4. **Given** I have `candidate.view` permission through a group in my company, **When** I access candidate data, **Then** I should only see candidates associated with my company.
5. **Given** I am a client user checking permissions for a resource, **When** the system evaluates my permissions, **Then** it should only consider permissions from groups belonging to my company.

---

### User Story 5 - Cross-Company Access for Backoffice (Priority: P1) `✅ Implemented (Back-Office)`

> **Note**: Implemented via `AuthPermission.is_cross_company` flag and `AuthUser.has_permission_code()` which performs a second cross-company permission check for backoffice users.

As a Backoffice User (Support/Admin), I want to access data across all client companies when granted cross-company permissions, while respecting company boundaries for standard permissions. This capability is exclusively available to backoffice users - client users are always restricted to their own company.

**Why this priority**: Essential for internal operations like customer support, analytics, and platform administration. This is a backoffice-only privilege to maintain strict multi-tenant security.

**Independent Test**: Grant a backoffice user cross-company permissions and verify they can access data from all companies. Verify standard permissions remain company-scoped.

**Acceptance Scenarios**:

1. **Given** I am a Support Agent with `ticket.view` permission marked as `is_cross_company=True`, **When** I access the tickets list, **Then** I should see tickets from all client companies.
2. **Given** I am a Support Agent with `candidate.view` permission (not cross-company), **When** I access candidates, **Then** I should only see candidates within the company context I'm operating in.
3. **Given** I am a Backoffice Admin, **When** I view the groups list, **Then** I should see groups from all companies plus global groups.
4. **Given** I have `analytics.export` with `is_cross_company=True`, **When** I generate a report, **Then** the report should include aggregated data from all companies.
5. **Given** I am a backoffice user assigning users to groups, **When** I assign a client user to a group, **Then** the system should validate company matching even though I have cross-company access.

---

### User Story 6 - View and Audit Permission Assignments (Priority: P2) `❌ Not Implemented`

As an Admin, I want to view which permissions a specific user has (across all their groups) so that I can audit access levels and understand effective permissions including user type and company restrictions.

**Why this priority**: Important for compliance and troubleshooting access issues.

**Independent Test**: Select a user and view their effective permissions list aggregated from all groups, including applicable user type and company scope.

**Acceptance Scenarios**:

1. **Given** a user "Bob" belongs to multiple groups, **When** I view Bob's user details, **Then** I should see a complete list of his effective permissions with company scope indicators.
2. **Given** I want to understand group membership, **When** I view a group's details, **Then** I should see all users assigned to that group with assignment metadata (assigned_by, assigned_at, expires_at).
3. **Given** a group has `applicable_user_type='client'`, **When** I view the group details, **Then** I should see the user type restriction clearly indicated.
4. **Given** a permission has `is_cross_company=True`, **When** I view permission details, **Then** I should see a clear indicator that this grants cross-company access.

---

### User Story 7 - Define Custom Permissions (Priority: P2) `✅ Implemented (Back-Office)`

> **Note**: `permissions-page.tsx` (`PermissionManager` component) provides full CRUD for `AuthPermission` records, including resource, action, category, applicable_user_type, and is_cross_company fields.

As a Super Admin, I want to define new permissions (e.g., `report.export`, `invoice.manage`) with appropriate user type restrictions and cross-company flags so that I can extend the authorization system as the platform grows.

**Why this priority**: Allows system to scale with new features without code changes.

**Independent Test**: Create a new permission with specific attributes, assign it to a group, and verify it's enforced correctly in the application.

**Acceptance Scenarios**:

1. **Given** I am in the Permissions Management section, **When** I create a new permission `analytics.export` with `applicable_user_type='backoffice'` and `is_cross_company=True`, **Then** it should appear in the available permissions list with those attributes.
2. **Given** the new permission exists, **When** I assign it to a group, **Then** users in that group should be able to access the protected feature.
3. **Given** I create a permission with `applicable_user_type='client'`, **When** I attempt to assign it to a backoffice-only group, **Then** the system should allow it but the permission won't be effective for backoffice users.
4. **Given** I create a cross-company permission, **When** a backoffice user has this permission, **Then** they should be able to access resources across all companies for that specific resource.action.

---

### User Story 8 - Prevent Orphaned Permissions (Priority: P2) `❌ Not Implemented`

As a System, I must ensure that deleted groups don't leave users without necessary access.

**Why this priority**: Prevents accidental lockouts and maintains system integrity.

**Independent Test**: Delete a group and verify users are warned or permissions are reassigned.

**Acceptance Scenarios**:

1. **Given** a group "Temp Contractors" has users assigned, **When** I attempt to delete the group, **Then** the system should warn me about affected users.
2. **Given** I confirm deletion of a group, **When** the deletion completes, **Then** users formerly in that group should lose only those specific permissions.

---

### User Story 9 - Frontend Permission Integration (Priority: P1) `🔶 Partially Implemented (Back-Office)`

> **Note**: The back-office has `usePermissions`, `useGroups`, `useUIRoutePermissions` hooks connected to the API, and a `PermissionGuard` component for route guarding. The `PermissionProvider` context provides permission-aware UI rendering. However, the client admin portal (`ai-talent-analyst-system-admin-portal`) has not yet integrated these permission APIs.

As a Frontend Developer, I want efficient APIs to retrieve user permissions and metadata so that I can render permission-aware UI components (hide/disable buttons, conditional routes, masked fields) without making excessive backend calls.

**Why this priority**: Essential for good UX and security - users should only see actions they can perform, and frontend needs optimized permission checking to avoid performance issues.

**Independent Test**: Load the application as a restricted user and verify that unauthorized UI elements (buttons, menu items, routes) are hidden/disabled based on their permissions. Verify permission data is fetched efficiently (single API call on login).

**Acceptance Scenarios**:

1. **Given** I am a user with limited permissions, **When** I log in to the application, **Then** the system should return all my effective permissions in a single API call (e.g., `GET /api/v1/users/me/permissions`).
2. **Given** I need to check multiple permissions for a page, **When** I call the bulk check API (e.g., `POST /api/v1/permissions/check` with `["candidate.view", "interview.view"]`), **Then** I should receive a response indicating which permissions I have in a single request.
3. **Given** I lack `candidate.delete` permission, **When** I view a candidate profile page, **Then** the "Delete" button should be hidden or disabled in the UI.
4. **Given** I have `candidate.view` but not `salary.view`, **When** I view a candidate profile, **Then** the salary field should be masked or hidden.
5. **Given** I lack `admin.access` permission, **When** I attempt to navigate to `/admin` route, **Then** the frontend should redirect me to an unauthorized page before making API calls.
6. **Given** the frontend has cached my permissions, **When** an admin adds me to a new group, **Then** my permissions should refresh on next page load or via WebSocket/polling notification.
7. **Given** I am a frontend component, **When** I need to check if a user has a specific permission, **Then** I should use a permission guard/directive (e.g., `<Button permission="candidate.create">`) that reads from cached permission data.
8. **Given** I am building a dynamic admin UI, **When** I fetch permission metadata via `GET /api/v1/permissions/metadata`, **Then** I should receive permission descriptions, UI labels, and grouping information for all available permissions.
9. **Given** I am a backoffice user with `is_cross_company=True` permissions, **When** I view the permissions response, **Then** cross-company permissions should be clearly flagged so the UI can indicate their scope.

---

### User Story 10 - UI Route Permission Mapping (Priority: P2) `✅ Implemented (Back-Office)`

> **Note**: `route-permissions-page.tsx` (`RoutePermissionManager` component) provides full CRUD for `UIRoutePermission` records, including route path, service_type (client/bo), permission_mode (ALL/ANY), ui_component_type, and required_permissions (M2M).

As a System Administrator, I want to centrally define which permissions are required for each UI route/page so that frontend developers can fetch permission requirements declaratively and users are prevented from accessing pages they lack permissions for.

**Why this priority**: Improves maintainability by centralizing permission requirements instead of hardcoding them in frontend code. Enables dynamic access control without frontend deployments.

**Independent Test**: Create a UI route mapping with multiple required permissions, load the page as a user with partial permissions, and verify access is denied. Update the route's permission requirements without code changes and verify new rules are enforced.

**Acceptance Scenarios**:

1. **Given** I am a System Admin, **When** I create a UI route mapping for `/candidates/:id` requiring `["candidate.view", "interview.view"]` with `permission_mode='ALL'`, **Then** the route mapping should be saved and accessible via API.
2. **Given** a route `/candidates/:id` requires `["candidate.view", "interview.view"]` with mode `ALL`, **When** a user with only `candidate.view` accesses the route, **Then** the frontend should block access and show an unauthorized message.
3. **Given** a route `/admin/dashboard` requires `["admin.access", "analytics.view"]` with mode `ANY`, **When** a user has `analytics.view` but not `admin.access`, **Then** the user should be granted access (needs at least one permission).
4. **Given** I am a frontend developer, **When** I fetch `GET /api/v1/ui-routes/permissions`, **Then** I should receive all route permission mappings to cache locally for route guard implementation.
5. **Given** a route has `permission_mode='ALL'`, **When** the frontend uses the bulk check API to verify permissions, **Then** it should deny access only if the user lacks any required permission.
6. **Given** I update a route's required permissions from `["candidate.view"]` to `["candidate.view", "candidate.edit"]`, **When** users reload the application, **Then** the new permission requirements should be enforced immediately without frontend code changes.
7. **Given** a route is marked as `ui_component_type='modal'`, **When** I view route metadata, **Then** I should see this is a component-level permission check (not full page).
8. **Given** multiple routes share the same permission requirements, **When** I query route permissions, **Then** I should be able to filter by permission to see which routes require specific access.

---

### User Story 11 - Assign Permissions to Backoffice and Client Users Based on UI Route Mapping (Priority: P2) `🔶 Partially Implemented (Back-Office)`

> **Note**: Route permission definitions are manageable via the back-office. User-to-group assignment sync to API is still pending (US3 TODO). The enforcement logic (checking route permissions before rendering) is partially wired in the back-office via `PermissionGuard`, but not yet in the client admin portal.

As a System Administrator or Company Admin, I want to assign specific permissions to Backoffice and Client users based on the defined UI routes so that access control is tailored to user roles, ensuring that users can only access the functionalities relevant to their responsibilities.

**Why this priority**: This feature is crucial for maintaining security and operational efficiency by ensuring that users have access only to the parts of the system they need, reducing the risk of unauthorized actions and streamlining permission management.

**Independent Test**: Assign permissions to a Backoffice user and a Client user for specific UI routes, attempt to access those routes as different users, and verify that access is granted or denied based on the assigned permissions and route requirements.

**Acceptance Scenarios**:

1. **Given** I am a System Administrator, **When** I assign the permissions `["admin.access"]` to a Backoffice user for the route `/admin/dashboard`, **Then** the user should be able to access the dashboard when the route permission mapping requires `["admin.access"]`.
2. **Given** I have assigned the permissions `["candidate.view", "interview.view"]` to a Client user for the route `/candidates/:id`, **When** the Client user accesses the route with `permission_mode='ALL'`, **Then** they should be able to view the candidate details since they have both required permissions.
3. **Given** a Backoffice user has been assigned `["interview.view"]` for accessing the route `/interviews`, **When** they attempt to access the route requiring `["interview.view"]`, **Then** they should be granted access.
4. **Given** a Client user has only been assigned `["candidate.view"]` and the route `/admin/dashboard` requires `["admin.access"]`, **When** they attempt to access the route, **Then** they should be blocked and shown an unauthorized message.
5. **Given** I am a Company Admin, **When** I update the permissions for a Client user from `["candidate.view"]` to `["candidate.view", "interview.create"]`, **Then** the user should have immediate access to routes requiring either permission upon reloading the application.
6. **Given** a route is marked as `permission_mode='ANY'` and requires `["admin.access", "analytics.view"]`, **When** a Backoffice user has only `["analytics.view"]`, **Then** the user should be granted access to the route since they have at least one required permission.
7. **Given** I am assigning a Backoffice user cross-company permissions (e.g., `["ticket.view"]` with `is_cross_company=True`), **When** the user accesses routes requiring these permissions, **Then** they should be able to access data across all client companies as defined by the UI route mapping.
8. **Given** a Client user is assigned the same cross-company permission flag, **When** they access the system, **Then** the cross-company permission should be ignored or denied to maintain strict company isolation for client users.
9. **Given** I am a frontend developer building a dynamic permissions UI, **When** I fetch the UI route permission metadata via `GET /api/v1/ui-routes/permissions`, **Then** I should receive all routes with their required permissions, permission modes, and applicable user types to render appropriate access controls.
10. **Given** a route's required permissions are updated from `["candidate.view"]` to `["candidate.view", "candidate.edit"]`, **When** Backoffice users reload the application, **Then** the new permission requirements should be enforced without backend redeployment, and users lacking the new permissions should lose access.

---

### User Story 12 - Manage Temporary Permission Assignments (Priority: P2) `✅ Implemented (Back-Office)`

> **Note**: `UserGroupAssignment` model has `expires_at` (nullable) and `is_active` fields. The `AuthUser.get_groups()` query filters out expired assignments using `Q(expires_at__isnull=True) | Q(expires_at__gt=now)`. UI for setting expiry dates is available in `user-assignments-page.tsx`.

As an Admin, I want to assign users to groups with expiration dates so that I can grant temporary access (e.g., contractors, interns) that automatically revokes after a set period.

**Why this priority**: Important for security and compliance - temporary staff should automatically lose access when their contract ends without manual intervention.

**Independent Test**: Assign a user to a group with an expiration date, wait for expiration (or simulate time passing), and verify the user loses those permissions automatically.

**Acceptance Scenarios**:

1. **Given** I am adding user "Jane" to the "Contractors" group, **When** I set an expiration date of "2026-03-31", **Then** the assignment should be saved with `expires_at='2026-03-31 23:59:59'`.
2. **Given** a user has a group assignment that expires today, **When** the system's daily cleanup job runs, **Then** the assignment should be marked as `is_active=False` and the user should lose those permissions.
3. **Given** a user has a group assignment expiring in 7 days, **When** I view the group membership list, **Then** I should see a warning indicator "Expires in 7 days".
4. **Given** a user's group assignment has expired, **When** I view their user details, **Then** the expired assignment should be visible in history but marked as "Expired" with the expiration date.
5. **Given** an expired assignment exists, **When** I want to renew the user's access, **Then** I should be able to create a new assignment or update the existing one with a new expiration date.

---

### Edge Cases

- **Circular Dependencies**: What happens if permissions reference each other? → Not applicable; permissions are simple `resource.action` strings with no inter-permission dependencies.
- **Permission Conflicts**: If a user has `candidate.read` from one group but `candidate.deny` from another, which takes precedence? → Allow-first/union strategy applies; no deny rules exist in the system.
- **Super Admin Bypass**: Should Super Admins bypass all permission checks, or should they also be subject to groups? → Super Admins belong to the "Super Admin" system group with all permissions; they follow the same permission model.
- **Deletion of Critical Groups**: What happens if someone tries to delete the "Company Admin" group that all admins belong to? → System prevents deletion of any group where `is_system_critical=True`.
- **Empty Groups**: Should groups with no users be automatically deleted, or should they persist? → Groups persist even with zero users; admins must explicitly delete unused groups.
- **System Group Modifications**: Can system groups be modified? → Yes, permissions can be added/removed, but the group cannot be deleted, renamed, or have `is_system_critical` flag changed.
- **Expired Assignments**: What happens if a user is actively using the system when their assignment expires? → Permissions are checked per-request; expired assignments take effect on the next permission check (next page load or API call).
- **Timezone Handling**: How are expiration dates handled across timezones? → All expiration dates stored in UTC; evaluated at end-of-day UTC (23:59:59 UTC).
- **Concurrent Permission Updates**: What if two admins modify the same group simultaneously? → Last-write-wins at the database level; audit logs capture both changes with timestamps.

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
- **FR-011**: System MUST provide a bulk permission retrieval API endpoint (e.g., `GET /api/v1/users/me/permissions`) that returns all effective permissions, groups, and cross-company flags in a single response.
- **FR-012**: System MUST provide a permission metadata API endpoint that returns permission descriptions, UI labels, resource categorization, and user type applicability for building dynamic UIs.
- **FR-013**: Frontend components MUST enforce permission-based UI rendering (hide/disable elements for unauthorized actions) as an optimistic check, while backend MUST remain the authoritative enforcer.
- **FR-014**: System MUST support permission caching in the frontend with a refresh mechanism when group assignments change (e.g., on login, after group modification, or via real-time notification).
- **FR-015**: System MUST include permission scope indicators (company-scoped vs cross-company) in API responses to help frontend display appropriate context.
- **FR-016**: System MUST provide a bulk permission check API endpoint (e.g., `POST /api/v1/permissions/check`) that accepts multiple permissions and returns a boolean result for each permission in a single request.
- **FR-017**: System MUST support UI Route Permission Mapping that defines which permissions are required for each frontend route/page with configurable permission modes (ALL or ANY).
- **FR-018**: System MUST provide an API to retrieve all UI route permission mappings (e.g., `GET /api/v1/ui-routes/permissions`) for frontend route guard initialization.
- **FR-019**: System MUST retain audit logs for all permission-related changes (group creation/deletion, permission assignments, user assignments) for at least 2 years for compliance and security auditing.
- **FR-020**: System MUST restrict audit log access to users with `audit.view` permission, ensuring only authorized personnel can review permission change history.
- **FR-021**: System MUST automatically deactivate user group assignments when `expires_at` date is reached, removing those permissions from the user's effective permission set.
- **FR-022**: System MUST prevent modification of system-critical groups beyond permission assignments (name, `is_system_critical` flag, and deletion are prohibited).
- **FR-023**: System MUST send notification emails to users 7 days before their group assignment expires (if `expires_at` is set) to provide advance warning.
- **FR-024**: System MUST validate that `is_cross_company=True` permissions are only effective for backoffice users; client users must be denied even if assigned such permissions.

### Key Entities

- **AuthGroup** (`public.atas_auth_group`): `id`, `name`, `description`, `company` (FK, nullable for global groups), `applicable_user_type` (enum: `client`, `backoffice`, `both`, default: `client`), `is_system_critical` (boolean), `permissions` (M2M → `AuthPermission`), `created_by` (FK → `AuthUser`), `created_at`, `updated_at`. Managed via `groups-page.tsx`.
- **AuthPermission** (`public.atas_auth_permission`): `id`, `name` (unique), `resource` (e.g., `candidate`), `action` (e.g., `view`), `description`, `ui_label`, `category` (user/candidate/job/interview/report/company/admin), `api_type` (client/backoffice/both), `api_endpoint`, `applicable_user_type` (client/backoffice/both), `is_cross_company` (boolean), `is_active` (boolean), `updated_by`, `created_at`. Unique on `(resource, action)`. Managed via `permissions-page.tsx`.
- **UserGroupAssignment** (`public.atas_user_group_assignment`): `user` (FK → `AuthUser`), `group` (FK → `AuthGroup`), `assigned_by` (FK → `AuthUser`), `assigned_at`, `expires_at` (nullable — for temporary assignments), `is_active` (boolean), `notes`. Unique on `(user, group)`. Managed via `user-assignments-page.tsx`.
- **UIRoutePermission** (`public.atas_ui_route_permission`): `id`, `route` (e.g., `/candidates/:id`), `service_type` (enum: `client`, `bo`), `required_permissions` (M2M → `AuthPermission`), `permission_mode` (enum: `ALL`, `ANY`), `ui_component_type` (page/modal/tab/section), `description`, `is_active` (boolean), `created_by`, `created_at`, `updated_at`. Unique on `(route, service_type)`. Managed via `route-permissions-page.tsx`.
- **PermissionAuditLog** (`public.atas_permission_audit_log`): `id`, `action_type` (enum: group_created/group_updated/group_deleted/permission_created/permission_updated/user_assigned/user_unassigned/permission_added_to_group/permission_removed_from_group/ui_route_created/ui_route_updated/ui_route_deleted), `actor` (FK → `AuthUser`), `target_user` (FK, nullable), `target_group` (FK, nullable), `target_permission` (FK, nullable), `target_ui_route` (FK, nullable), `old_value` (JSON), `new_value` (JSON), `description`, `ip_address`, `user_agent`, `timestamp`.
- **AuthUser** (`public.auth_user`): `email`, `username`, `user_type` (client/backoffice), `first_name`, `last_name`, `is_active`, `is_superuser`, `joined_date`, `last_login`.
- **BackofficeUser** profile (`public.bo_user_profile`): `user` (OneToOne PK), `employee_id`, `role` (admin/manager/analyst/support/viewer), `department`, `access_level` (1–5), `status` (active/inactive/suspended/pending), `phone`, `notes`.
- **ClientUser** profile (`public.client_user`): `user` (OneToOne PK), `company` (FK), `permission_level` (owner/admin/manager/member/viewer), `status` (active/inactive/suspended/trial/expired), `subscription_tier`, `max_sessions`, `sessions_used`.

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
- **RESTRICTION**: This flag only applies to backoffice users. Client users always have company-scoped access regardless of this flag.
- Use case: Support agents viewing tickets across all client companies, analytics for all customers.

**Security Benefits:**

- Prevents client users from accessing other companies' data (strict isolation enforced)
- Prevents user type mismatches (e.g., client users in backoffice-only groups)
- Enables controlled cross-company access exclusively for backoffice staff
- Client users are always company-scoped regardless of permission flags
- Maintains additive permission model while enforcing isolation boundaries

## Assumptions

- **Additive Permissions**: Users accumulate permissions from all their groups (no deny rules by default).
- **Company Isolation**: Company Admins can only create/manage groups within their own company; Super Admins can manage global groups. Client users are strictly isolated to their own company data.
- **Cross-Company Access**: Only backoffice users can be granted cross-company permissions via `is_cross_company=True` flag. Client users cannot access other companies' data under any circumstances.
- **System Groups**: Some groups (e.g., "Super Admin", "Company Admin") are created by default and cannot be deleted.
- **Permission Format**: Permissions follow the format `resource.action` (e.g., `candidate.view`, `job.create`, `report.export`).

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Admins can create a new group and assign permissions in < 2 minutes.
- **SC-002**: Permission enforcement blocks 100% of unauthorized access attempts.
- **SC-003**: Effective permissions for a user are calculated in < 100ms.
- **SC-004**: Audit logs capture 100% of permission changes with accurate timestamps and actor identification.
- **SC-005**: System supports at least 50 custom groups per company without performance degradation.
- **SC-006**: Bulk permission API responds with complete user permissions in < 200ms (P95).
- **SC-007**: Frontend permission checks (button visibility, route guards) execute in < 10ms using cached permission data.
- **SC-008**: 100% of permission-protected UI elements (buttons, menus, routes, fields) correctly reflect user's effective permissions without requiring page refresh after login.
- **SC-009**: UI route permission mappings can be updated and take effect within 60 seconds without requiring frontend redeployment.
- **SC-010**: Permission checks for users with 10+ groups complete in < 50ms (P95), ensuring performance scales with complex permission hierarchies.
- **SC-011**: 100% of permission changes (assignments, removals, group modifications) are logged in audit trail with actor identification, timestamp, and old/new values.
- **SC-012**: Expired group assignments are automatically deactivated within 1 hour of expiration time.
- **SC-013**: System successfully prevents 100% of attempts to modify system-critical groups beyond permission changes.

---

## Appendix A: Cross-Feature Permission Catalog

This section documents the specific permissions required across all features in the AI-Talent-Analyst system. These permissions follow the `resource.action` format and should be seeded during system initialization.

### Feature 001: AI-HR Interview System

**Interview Management:**

- `interview.create` - Create new AI-led interviews
- `interview.view` - View interview details and responses
- `interview.edit` - Modify interview settings
- `interview.delete` - Remove interviews
- `interview.start` - Start/conduct an interview (for candidates)

**Candidate Management:**

- `candidate.create` - Add new candidates to the system
- `candidate.view` - View candidate profiles and data
- `candidate.edit` - Update candidate information
- `candidate.delete` - Remove candidates
- `candidate.invite` - Send interview invitations

**Job Management:**

- `job.create` - Create new job positions
- `job.view` - View job descriptions and details
- `job.edit` - Modify job postings
- `job.delete` - Remove job postings
- `job.publish` - Publish jobs for candidate applications

**Resume & Analysis:**

- `resume.view` - View uploaded resumes
- `resume.download` - Download resume files
- `resume.analyze` - Trigger AI resume analysis

**Reporting:**

- `report.view` - View interview analytics and reports
- `report.export` - Export reports to PDF/CSV
- `analytics.view` - Access analytics dashboards

**Sensitive Data:**

- `salary.view` - View candidate salary information (may be masked for some roles)

### Feature 002/007: Admin Portals (SaaS & Company Self-Service)

**Company Management (Super Admin):**

- `company.create` - Onboard new client companies
- `company.view` - View company details
- `company.edit` - Update company information
- `company.delete` - Remove companies (soft delete)
- `company.suspend` - Suspend company accounts

**User Management:**

- `user.create` - Create new user accounts
- `user.view` - View user profiles
- `user.edit` - Update user information
- `user.delete` - Remove users (soft delete)
- `user.invite` - Send user invitation emails
- `user.activate` - Activate/reactivate user accounts
- `user.deactivate` - Deactivate user accounts

**Settings:**

- `settings.view` - View company/system settings
- `settings.edit` - Modify settings

### Feature 003: Interview Status Tracking

**Status Management:**

- `candidate.status.view` - View candidate status in pipeline
- `candidate.status.update` - Change candidate status
- `status.history.view` - View status change audit trail

### Feature 004: Authorization (This Feature)

**Group Management:**

- `group.create` - Create authorization groups
- `group.view` - View group details
- `group.edit` - Modify groups and their permissions
- `group.delete` - Remove groups

**Permission Management:**

- `permission.create` - Define new permissions
- `permission.view` - View permission definitions
- `permission.edit` - Modify permission metadata
- `permission.delete` - Remove custom permissions
- `permission.assign` - Assign permissions to groups

**User Assignment:**

- `user.group.assign` - Add users to groups
- `user.group.remove` - Remove users from groups
- `user.permissions.view` - View user's effective permissions

**Audit:**

- `audit.view` - Access audit logs for permission changes
- `audit.export` - Export audit logs

### Feature 005: Credit & Subscription Management

**Billing:**

- `billing.view` - View billing history and invoices
- `billing.export` - Download invoices and receipts

**Subscription:**

- `subscription.view` - View subscription plan details
- `subscription.manage` - Upgrade/downgrade plans
- `subscription.cancel` - Cancel subscriptions

**Payment:**

- `payment.view` - View payment methods
- `payment.manage` - Add/remove payment methods
- `payment.process` - Process manual payments (backoffice)

**Credits:**

- `credits.view` - View credit balance and usage
- `credits.topup` - Add credits to account
- `credits.adjust` - Manually adjust credits (backoffice)

### Feature 006: System Management

**System Configuration:**

- `system.config.view` - View system-wide settings
- `system.config.edit` - Modify system settings

**AI Configuration:**

- `ai.model.switch` - Change AI models
- `ai.prompt.edit` - Modify system prompts
- `ai.prompt.history` - View prompt version history

**Monitoring:**

- `system.dashboard.view` - Access system health dashboard
- `system.metrics.view` - View system metrics
- `system.logs.view` - Access system logs

**Token Management:**

- `token.usage.view` - View token consumption analytics
- `token.audit.view` - Access detailed token usage logs

### Feature 008: Limit Enforcement

**Limits:**

- `limits.view` - View plan limits and current usage
- `limits.override` - Override limits for specific companies (backoffice)

### Cross-Company Permissions (Backoffice Only)

The following permissions should be marked with `is_cross_company=True` for backoffice staff:

- `ticket.view` (cross-company) - Support agents viewing tickets across all companies
- `analytics.export` (cross-company) - Generate reports across all companies
- `company.view` (cross-company) - View all company details
- `user.view` (cross-company) - View users across companies
- `billing.view` (cross-company) - View billing across all companies
- `token.usage.view` (cross-company) - View token usage across platform
- `audit.view` (cross-company) - View audit logs across all companies

### System Groups & Default Permissions

**Super Admin Group** (`is_system_critical=True`, `company=None`, `applicable_user_type='backoffice'`):\

- All permissions with `is_cross_company=True`

**Company Admin Group** (`is_system_critical=True`, `company=[specific]`, `applicable_user_type='client'`):\

- All company-scoped permissions within their company
- Cannot access other companies' data

**Support Agent Group** (`applicable_user_type='backoffice'`):\

- Read-only cross-company permissions
- `ticket.view` (cross-company), `company.view` (cross-company), `user.view` (cross-company)
- No delete or edit permissions

**Hiring Manager Group** (`applicable_user_type='client'`):\

- `job.create`, `job.edit`, `job.view`, `job.delete`
- `candidate.view`, `candidate.edit`, `candidate.invite`
- `interview.view`, `report.view`, `analytics.view`
- `salary.view`

**Interviewer Group** (`applicable_user_type='client'`):\

- `candidate.view` (without `salary.view`)
- `interview.view`, `interview.create`
- `report.view`

**Recruiter Group** (`applicable_user_type='client'`):\

- `candidate.create`, `candidate.view`, `candidate.edit`, `candidate.invite`
- `job.view`
- `interview.view`
- No `salary.view`
