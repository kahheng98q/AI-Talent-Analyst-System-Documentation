# Task Breakdown: User Authorization Groups & Permissions

**Source**: [004-user-authorization-groups-permissions (Phase 2)/spec.md](./spec.md)  
**Generated**: 2026-02-03  
**Status**: Planning

---

## Overview

This feature implements a comprehensive Role-Based Access Control (RBAC) system with groups, permissions, multi-tenant isolation, and cross-company access for backoffice users. It provides the authorization foundation for all platform features.

**Key Capabilities**:

- Create and manage authorization groups with custom permissions
- Assign users to multiple groups (additive permissions)
- Multi-tenant data isolation for client users
- Cross-company access for backoffice users
- UI route permission mapping with declarative access control
- Temporary permission assignments with automatic expiration
- Comprehensive audit logging of permission changes
- Frontend permission integration with optimized APIs

**Cross-Referencing System**:

This task breakdown uses **3 essential linkage types** to show relationships:

- **Depends On**: What must exist first (e.g., Database 1.1)
- **Related**: Connected tasks in other areas (e.g., Frontend 4.1, Backend 2.3)
- **Test Coverage**: Which test validates this task (e.g., Testing 5.1)

---

## Database

### Main Task 1: Authorization Schema Design

**Status**: Completed

**Description**: Design and implement core database schema for groups, permissions, user assignments, and audit trails with multi-tenant isolation.

**Spec Reference**: User Story 1, FR-001, FR-002, FR-003

#### Subtask 1.1: AuthGroup Table

- **Spec Reference**: Entity: AuthGroup, FR-001, FR-010
- **Status**: Completed
- **Description**: Create migration for `auth_groups` table to store authorization groups
- **Columns**:
  - `id` (PK, UUID)
  - `name` (VARCHAR, unique within company)
  - `description` (TEXT)
  - `company_id` (FK to companies, nullable for global groups, indexed)
  - `applicable_user_type` (ENUM: `client`, `backoffice`, `both`)
  - `is_system_critical` (BOOLEAN, default false)
  - `created_by` (FK to users), `created_at`, `updated_at`
- **Indexes**: `(company_id, name)` unique, `is_system_critical`, `applicable_user_type`
- **Constraints**: System groups cannot have `company_id` set
- **Related**: Backend 2.1, Backend 2.2, Backend 2.3, Backend 2.4, Frontend 4.1
- **Test Coverage**: Testing 5.1, Testing 5.2, Testing 5.3
- **Acceptance**: Migration runs successfully; unique constraint enforced; system groups validated

#### Subtask 1.2: AuthPermission Table

- **Spec Reference**: Entity: AuthPermission, FR-008
- **Status**: Completed
- **Description**: Create migration for `auth_permissions` table using resource.action format
- **Columns**:
  - `id` (PK, UUID)
  - `resource` (VARCHAR, e.g., "candidate")
  - `action` (VARCHAR, e.g., "view")
  - `description` (TEXT)
  - `applicable_user_type` (ENUM: `client`, `backoffice`, `both`)
  - `is_cross_company` (BOOLEAN, default false)
  - `created_at`
- **Indexes**: `(resource, action)` unique, `applicable_user_type`, `is_cross_company`
- **Related**: Backend 2.5, Backend 2.6, Backend 2.7, Frontend 4.2
- **Test Coverage**: Testing 5.4, Testing 5.5
- **Acceptance**: Migration runs; unique constraint on resource+action; seeded with Appendix A permissions

#### Subtask 1.3: GroupPermission Junction Table

- **Spec Reference**: FR-002
- **Status**: Completed
- **Description**: Create many-to-many relationship between groups and permissions
- **Columns**:
  - `id` (PK, UUID)
  - `group_id` (FK to auth_groups)
  - `permission_id` (FK to auth_permissions)
  - `assigned_by` (FK to users), `assigned_at`
- **Indexes**: `(group_id, permission_id)` unique, `assigned_by`, `assigned_at`
- **Related**: Backend 2.2, Backend 2.6, Backend 2.7
- **Test Coverage**: Testing 5.1, Testing 5.4
- **Acceptance**: Foreign keys validated; duplicate assignments prevented

#### Subtask 1.4: UserGroupAssignment Table

- **Spec Reference**: Entity: UserGroupAssignment, FR-003, User Story 12
- **Status**: Completed
- **Description**: Create migration for user-to-group assignments with expiration support
- **Columns**:
  - `id` (PK, UUID)
  - `user_id` (FK to users, indexed)
  - `group_id` (FK to auth_groups, indexed)
  - `assigned_by` (FK to users), `assigned_at`
  - `expires_at` (TIMESTAMP, nullable)
  - `is_active` (BOOLEAN, default true)
  - `notes` (TEXT, nullable)
- **Indexes**: `(user_id, group_id, is_active)`, `expires_at`, `is_active`, `assigned_at`
- **Constraints**: Check constraint: `expires_at IS NULL OR expires_at > assigned_at`
- **Related**: Backend 2.8, Backend 2.9, Backend 2.10, Frontend 4.3
- **Test Coverage**: Testing 5.6, Testing 5.7, Testing 5.8
- **Acceptance**: Migration runs; prevents duplicate active assignments; expiration validation works

#### Subtask 1.5: UIRoutePermission Table

- **Spec Reference**: Entity: UIRoutePermission, User Story 10, FR-017
- **Status**: Completed
- **Description**: Create migration for UI route permission mappings
- **Columns**:
  - `id` (PK, UUID)
  - `route` (VARCHAR, e.g., "/candidates/:id", unique)
  - `required_permissions` (JSON array of permission strings)
  - `permission_mode` (ENUM: `ALL`, `ANY`)
  - `description` (TEXT)
  - `ui_component_type` (ENUM: `page`, `modal`, `tab`, `section`)
  - `is_active` (BOOLEAN, default true)
  - `created_at`, `updated_at`, `created_by` (FK to users)
- **Indexes**: `route` unique, `is_active`, `ui_component_type`
- **Related**: Backend 3.1, Backend 3.2, Backend 3.3, Frontend 4.4
- **Test Coverage**: Testing 6.1, Testing 6.2, Testing 6.3
- **Acceptance**: Route uniqueness enforced; JSON validation for required_permissions

#### Subtask 1.6: PermissionAuditLog Table

- **Spec Reference**: Entity: PermissionAuditLog, FR-009, FR-019
- **Status**: Completed
- **Description**: Create audit log table for all permission-related changes
- **Columns**:
  - `id` (PK, UUID)
  - `action_type` (ENUM: `group_created`, `group_updated`, `group_deleted`, `permission_assigned`, `permission_removed`, `user_assigned`, `user_removed`, `user_expired`)
  - `actor_id` (FK to users, indexed)
  - `target_user_id` (FK to users, nullable, indexed)
  - `target_group_id` (FK to auth_groups, nullable, indexed)
  - `old_value` (JSON, nullable)
  - `new_value` (JSON, nullable)
  - `timestamp` (TIMESTAMP, indexed)
  - `ip_address` (VARCHAR, nullable)
  - `user_agent` (TEXT, nullable)
- **Indexes**: `(action_type, timestamp)`, `actor_id`, `target_user_id`, `target_group_id`, `timestamp` DESC
- **Retention**: Partitioned by month; 2-year retention policy
- **Related**: Backend 2.11, Backend 2.12
- **Test Coverage**: Testing 5.9, Testing 5.10
- **Acceptance**: All audit actions logged; retention policy configured; queryable by multiple dimensions

---

## Backend

### Main Task 2: Group & Permission Management Service

**Status**: Pending

**Description**: Implement core business logic and API endpoints for creating, managing, and querying authorization groups and permissions with multi-tenant isolation.

**Spec Reference**: User Story 1, User Story 7, FR-001, FR-002, FR-008

#### Subtask 2.1: AuthGroupService - Core Logic

- **Spec Reference**: FR-001, FR-010, User Story 1
- **Status**: Completed
- **Description**: Implement service layer for group CRUD operations with validation
- **Methods**:
  - `create_group(name, description, company_id, applicable_user_type, created_by)` → validates company isolation, user type, uniqueness
  - `update_group(group_id, updates, actor)` → validates system_critical restrictions
  - `delete_group(group_id, actor)` → prevents deletion of system groups, warns about affected users
  - `list_groups(company_id, user_type_filter)` → respects multi-tenant isolation
  - `can_assign_user(group, user)` → validates user type, company matching (User Story 3)
- **Depends On**: Database 1.1
- **Related**: Backend 2.2, Backend 2.3, Backend 2.4, Frontend 4.1
- **Test Coverage**: Testing 5.1, Testing 5.2, Testing 5.3
- **Acceptance**: All validation rules enforced; system groups protected; company isolation maintained

#### Subtask 2.2: API - GET /api/v1/groups

- **Spec Reference**: User Story 1, User Story 4, User Story 5, FR-006
- **Status**: Completed
- **Description**: List authorization groups with company filtering
- **Request**: Query params: `company_id` (optional for backoffice), `user_type`, `page`, `limit`
- **Response**: `{ items: [{ id, name, description, company_id, applicable_user_type, is_system_critical, member_count }], total, page, limit }`
- **Authorization**: Client users see only their company groups; backoffice sees all
- **Depends On**: Backend 2.1
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 5.2, Testing 7.1, Testing 7.2
- **Acceptance**: Returns filtered groups in <200ms; multi-tenant isolation verified; cross-company access for backoffice works

#### Subtask 2.3: API - POST /api/v1/groups

- **Spec Reference**: User Story 1, FR-001
- **Status**: Completed
- **Description**: Create new authorization group
- **Request**: `{ name, description, company_id, applicable_user_type }`
- **Response**: `{ id, name, description, company_id, applicable_user_type, is_system_critical, created_at }`
- **Validation**: Name uniqueness within company; user type valid; company access validated
- **Authorization**: Requires `group.create` permission; company admins restricted to own company
- **Depends On**: Backend 2.1
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 5.1, Testing 7.3
- **Acceptance**: Group created with audit log; 409 on duplicate name; 403 on cross-company attempt

#### Subtask 2.4: API - PUT /api/v1/groups/{id}

- **Spec Reference**: User Story 1, FR-022
- **Status**: Completed
- **Description**: Update group details (not permissions or members)
- **Request**: `{ name, description }`
- **Response**: Updated group object
- **Validation**: Cannot modify `is_system_critical`, `company_id`, or `applicable_user_type`
- **Authorization**: Requires `group.edit` permission; company isolation enforced
- **Depends On**: Backend 2.1
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 5.3, Testing 7.4
- **Acceptance**: Updates saved with audit log; system group restrictions enforced; 403 on unauthorized

#### Subtask 2.5: PermissionService - Core Logic

- **Spec Reference**: FR-008, User Story 7
- **Status**: Completed
- **Description**: Implement service layer for permission management
- **Methods**:
  - `create_permission(resource, action, description, applicable_user_type, is_cross_company)` → validates format, uniqueness
  - `list_permissions(user_type_filter, cross_company_filter)` → fetches available permissions
  - `get_permission_metadata()` → returns descriptions, UI labels, categorization (FR-012)
  - `check_permission(user, permission_string, company_id)` → validates effective permission (FR-005)
  - `bulk_check_permissions(user, permission_list, company_id)` → efficient multi-check (FR-016)
- **Depends On**: Database 1.2
- **Related**: Backend 2.6, Backend 2.7, Backend 2.13
- **Test Coverage**: Testing 5.4, Testing 5.5
- **Acceptance**: Permission format validated (resource.action); cross-company restricted to backoffice; metadata includes UI labels

#### Subtask 2.6: API - POST /api/v1/groups/{id}/permissions

- **Spec Reference**: User Story 1, FR-002
- **Status**: Completed
- **Description**: Assign permissions to a group
- **Request**: `{ permission_ids: [uuid, ...] }`
- **Response**: `{ group_id, permissions: [...], assigned_at }`
- **Authorization**: Requires `permission.assign`; validates user can modify this group
- **Depends On**: Backend 2.5, Database 1.3
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 5.1, Testing 7.5
- **Acceptance**: Permissions added; duplicate prevention; audit logged; 403 on unauthorized

#### Subtask 2.7: API - DELETE /api/v1/groups/{id}/permissions/{permission_id}

- **Spec Reference**: User Story 1, FR-002
- **Status**: Completed
- **Description**: Remove permission from a group
- **Response**: `{ success: true, removed_at }`
- **Authorization**: Requires `permission.assign`; validates user can modify this group
- **Depends On**: Backend 2.5, Database 1.3
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 5.1, Testing 7.5
- **Acceptance**: Permission removed; audit logged; users in group lose that permission immediately

#### Subtask 2.8: UserGroupAssignmentService - Core Logic

- **Spec Reference**: User Story 3, User Story 12, FR-003, FR-021
- **Status**: Completed
- **Description**: Implement service layer for user-to-group assignments
- **Methods**:
  - `assign_user_to_group(user_id, group_id, assigned_by, expires_at, notes)` → validates user type, company match (User Story 3 scenario 4, 5, 6)
  - `remove_user_from_group(user_id, group_id, actor)` → deactivates assignment
  - `list_user_groups(user_id)` → fetches active group memberships
  - `list_group_users(group_id)` → fetches users in group with expiration warnings
  - `renew_assignment(assignment_id, new_expires_at)` → extends expiration
  - `expire_assignments()` → batch job to deactivate expired assignments (FR-021)
- **Depends On**: Database 1.4, Backend 2.1
- **Related**: Backend 2.9, Backend 2.10, Frontend 4.3
- **Test Coverage**: Testing 5.6, Testing 5.7, Testing 5.8, Testing 7.6, Testing 7.7, Testing 7.8
- **Acceptance**: Company matching validated; user type validated; global group restrictions enforced; duplicate active assignments prevented

#### Subtask 2.9: API - POST /api/v1/groups/{id}/users

- **Spec Reference**: User Story 3, FR-007
- **Status**: Completed
- **Description**: Assign user to a group with optional expiration
- **Request**: `{ user_id, expires_at (optional), notes (optional) }`
- **Response**: `{ assignment_id, user_id, group_id, assigned_by, assigned_at, expires_at }`
- **Validation**: Validates company matching (User Story 3 scenario 4), user type compatibility (scenario 5), global group restrictions (scenario 6)
- **Authorization**: Requires `user.group.assign`; validates company access
- **Depends On**: Backend 2.8
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 5.6, Testing 7.6, Testing 7.7, Testing 7.8
- **Acceptance**: User assigned; permissions take effect immediately; 400 on company mismatch; 400 on user type mismatch; audit logged

#### Subtask 2.10: API - DELETE /api/v1/groups/{id}/users/{user_id}

- **Spec Reference**: User Story 3, FR-007
- **Status**: Completed
- **Description**: Remove user from a group
- **Response**: `{ success: true, removed_at }`
- **Authorization**: Requires `user.group.remove`; validates company access
- **Depends On**: Backend 2.8
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 5.6, Testing 7.6
- **Acceptance**: Assignment marked inactive; permissions revoked immediately; audit logged

#### Subtask 2.11: AuditLogService - Core Logic

- **Spec Reference**: FR-009, FR-019, FR-020
- **Status**: Pending
- **Description**: Implement service layer for audit logging and querying
- **Methods**:
  - `log_action(action_type, actor_id, target_user_id, target_group_id, old_value, new_value, ip_address, user_agent)` → creates audit entry
  - `query_logs(filters)` → searches by action type, actor, target, date range
  - `export_logs(filters, format)` → exports audit data (CSV/JSON)
- **Depends On**: Database 1.6
- **Related**: Backend 2.12, Backend 2.1, Backend 2.8, Frontend 4.5
- **Test Coverage**: Testing 5.9, Testing 5.10
- **Acceptance**: All changes logged automatically; query performance <500ms; retention policy enforced

#### Subtask 2.12: API - GET /api/v1/audit/permissions

- **Spec Reference**: User Story 6, FR-009, FR-020
- **Status**: Pending
- **Description**: Query permission audit logs
- **Request**: Query params: `action_type`, `actor_id`, `target_user_id`, `target_group_id`, `start_date`, `end_date`, `page`, `limit`
- **Response**: `{ items: [{ id, action_type, actor, target_user, target_group, old_value, new_value, timestamp }], total }`
- **Authorization**: Requires `audit.view` permission
- **Depends On**: Backend 2.11
- **Related**: Frontend 4.5
- **Test Coverage**: Testing 5.9, Testing 7.9
- **Acceptance**: Returns filtered logs in <500ms; all filters work; restricted to authorized users

#### Subtask 2.13: API - GET /api/v1/users/{id}/permissions

- **Spec Reference**: User Story 6, FR-011
- **Status**: Completed
- **Description**: Get user's effective permissions (union of all groups)
- **Response**: `{ user_id, groups: [{ id, name, permissions: [...] }], effective_permissions: [{ resource, action, is_cross_company, source_groups: [...] }] }`
- **Authorization**: Requires `user.permissions.view` or self-access
- **Depends On**: Backend 2.5, Backend 2.8, Database 1.3, Database 1.4
- **Related**: Frontend 4.2, Frontend 4.3
- **Test Coverage**: Testing 5.7, Testing 7.10
- **Acceptance**: Returns complete permission set in <100ms (FR-003); additive model applied; company-scoped for client users; cross-company flags included

---

### Main Task 3: UI Route Permission Mapping Service

**Status**: Pending

**Description**: Implement service and API endpoints for managing and querying UI route permission requirements with declarative access control.

**Spec Reference**: User Story 9, User Story 10, User Story 11, FR-017, FR-018

#### Subtask 3.1: UIRoutePermissionService - Core Logic

- **Spec Reference**: User Story 10, FR-017
- **Status**: Completed
- **Description**: Implement service layer for route permission management
- **Methods**:
  - `create_route_mapping(route, required_permissions, permission_mode, description, ui_component_type)` → validates route format, permission existence
  - `update_route_mapping(route_id, updates)` → modifies permission requirements
  - `delete_route_mapping(route_id)` → removes route mapping
  - `check_route_access(user, route)` → validates if user can access route based on requirements
  - `list_routes_by_permission(permission_string)` → finds which routes require specific permission
- **Depends On**: Database 1.5, Backend 2.5
- **Related**: Backend 3.2, Backend 3.3, Frontend 4.4
- **Test Coverage**: Testing 6.1, Testing 6.2, Testing 6.3
- **Acceptance**: Route validation works; permission mode (ALL/ANY) enforced; route patterns (/:id) supported

#### Subtask 3.2: API - GET /api/v1/ui-routes/permissions

- **Spec Reference**: User Story 9, User Story 10, FR-018
- **Status**: Completed
- **Description**: Retrieve all UI route permission mappings for frontend caching
- **Response**: `{ routes: [{ route, required_permissions, permission_mode, ui_component_type, is_active }] }`
- **Caching**: Response cached for 5 minutes; invalidated on route updates
- **Authorization**: Public endpoint (used by frontend route guards before authentication)
- **Depends On**: Backend 3.1
- **Related**: Frontend 4.4
- **Test Coverage**: Testing 6.1, Testing 8.1
- **Acceptance**: Returns all active routes in <200ms; frontend can cache locally; updates reflected within 60 seconds

#### Subtask 3.3: API - POST /api/v1/ui-routes/check-access

- **Spec Reference**: User Story 10, User Story 11
- **Status**: Pending
- **Description**: Bulk check if user can access multiple routes
- **Request**: `{ user_id, routes: ["/candidates", "/admin/dashboard"] }`
- **Response**: `{ access: [{ route, allowed: boolean, missing_permissions: [...] }] }`
- **Authorization**: Requires `user.permissions.view` or self-access
- **Depends On**: Backend 3.1, Backend 2.13
- **Related**: Frontend 4.4
- **Test Coverage**: Testing 6.2, Testing 8.2
- **Acceptance**: Checks multiple routes in single request; returns missing permissions for denied routes; <100ms response time

#### Subtask 3.4: API - POST /api/v1/ui-routes/permissions (Admin)

- **Spec Reference**: User Story 10, User Story 11
- **Status**: Completed
- **Description**: Create or update UI route permission mapping
- **Request**: `{ route, required_permissions, permission_mode, description, ui_component_type }`
- **Response**: Created/updated route mapping object
- **Authorization**: Requires `system.config.edit` permission (admin only)
- **Depends On**: Backend 3.1
- **Related**: Frontend 4.6
- **Test Coverage**: Testing 6.3, Testing 8.3
- **Acceptance**: Route mapping saved; validation enforced; cache invalidated; audit logged

---

### Main Task 4: Permission Enforcement Middleware

**Status**: Pending

**Description**: Implement middleware and decorators for enforcing permissions at API/controller level with caching and performance optimization.

**Spec Reference**: FR-005, FR-013, FR-014

#### Subtask 4.1: Permission Decorator Implementation

- **Spec Reference**: FR-005
- **Status**: Completed
- **Description**: Create `@require_permission` decorator for API endpoints
- **Functionality**: Checks if request.user has required permission(s) before allowing action
- **Parameters**: `permissions` (list), `mode` (`ALL` or `ANY`), `company_context` (optional for cross-company checks)
- **Error Handling**: Returns 403 with clear message indicating missing permissions
- **Depends On**: Backend 2.5, Backend 2.13
- **Related**: Backend 4.2, Backend 4.3
- **Test Coverage**: Testing 7.11, Testing 7.12
- **Acceptance**: Blocks unauthorized access; logs authorization failures; <10ms overhead per request

#### Subtask 4.2: Permission Caching Layer

- **Spec Reference**: FR-014
- **Status**: Pending
- **Description**: Implement Redis-based caching for user permissions
- **Functionality**:
  - Cache user's effective permissions on login (TTL: 1 hour)
  - Invalidate cache when user group assignments change
  - Fallback to database on cache miss
- **Cache Key**: `user_permissions:{user_id}:{company_id}`
- **Depends On**: Backend 2.13, Backend 4.1
- **Related**: Backend 2.9, Backend 2.10
- **Test Coverage**: Testing 7.13, Testing 7.14
- **Acceptance**: Permission checks <5ms with cache hit; cache invalidation within 1 second of assignment change

#### Subtask 4.3: Company Context Middleware

- **Spec Reference**: User Story 4, User Story 5, FR-005
- **Status**: Pending
- **Description**: Extract company context from request for multi-tenant filtering
- **Functionality**:
  - Detect if user is client (use user.company_id) or backoffice (allow cross-company)
  - Add `request.company_context` for downstream services
  - Validate cross-company permissions for backoffice users
- **Depends On**: Backend 2.5, Backend 4.1
- **Related**: Backend 2.2, Backend 3.2
- **Test Coverage**: Testing 7.1, Testing 7.2, Testing 7.15
- **Acceptance**: Client users always scoped to own company; backoffice cross-company access works; 100% isolation maintained

---

### Main Task 5: Background Jobs & Notifications

**Status**: Pending

**Description**: Implement scheduled jobs for assignment expiration, notifications, and audit log retention.

**Spec Reference**: User Story 12, FR-021, FR-023

#### Subtask 5.1: Assignment Expiration Job

- **Spec Reference**: FR-021, User Story 12
- **Status**: Pending
- **Description**: Scheduled job to deactivate expired group assignments
- **Schedule**: Runs every 1 hour
- **Functionality**:
  - Query assignments where `expires_at < NOW()` AND `is_active=true`
  - Set `is_active=false` for expired assignments
  - Log expiration events in audit log
  - Invalidate permission cache for affected users
- **Depends On**: Backend 2.8, Backend 2.11, Backend 4.2
- **Test Coverage**: Testing 5.8, Testing 7.16
- **Acceptance**: Expired assignments deactivated within 1 hour; users lose permissions immediately after deactivation; audit logged

#### Subtask 5.2: Expiration Warning Email Job

- **Spec Reference**: FR-023, User Story 12
- **Status**: Pending
- **Description**: Scheduled job to notify users 7 days before assignment expiration
- **Schedule**: Runs daily at 9:00 AM UTC
- **Functionality**:
  - Query assignments where `expires_at BETWEEN NOW() AND NOW() + 7 days` AND `is_active=true`
  - Send email to user and assigning admin
  - Include group name, expiration date, renewal instructions
- **Depends On**: Backend 2.8
- **Test Coverage**: Testing 5.8, Testing 7.17
- **Acceptance**: Emails sent 7 days before expiration; includes actionable instructions; no duplicate sends

#### Subtask 5.3: Audit Log Retention Job

- **Spec Reference**: FR-019
- **Status**: Pending
- **Description**: Scheduled job to archive/delete audit logs older than 2 years
- **Schedule**: Runs monthly on 1st day at 2:00 AM UTC
- **Functionality**:
  - Archive logs older than 2 years to cold storage (S3)
  - Delete from primary database after successful archive
  - Maintain audit trail of archival process
- **Depends On**: Backend 2.11, Database 1.6
- **Test Coverage**: Testing 5.10, Testing 7.18
- **Acceptance**: Logs archived successfully; primary database size managed; archival process audited

---

## Frontend

### Main Task 4: Group & Permission Management UI

**Status**: Pending

**Description**: Build admin interface for creating and managing authorization groups, permissions, and user assignments.

**Spec Reference**: User Story 1, User Story 3, FR-006, FR-007

#### Subtask 4.1: Group Management Page

- **Spec Reference**: User Story 1, User Story 8, FR-006
- **Status**: Completed
- **Description**: Build UI for viewing, creating, editing, and deleting groups
- **Components**:
  - `GroupListView` - Table with search, filter by company/user type
  - `GroupCreateModal` - Form for new group creation
  - `GroupEditModal` - Update group details (name, description only for system groups)
  - `GroupDeleteConfirmation` - Warning about affected users (User Story 8)
- **Depends On**: Backend 2.2, Backend 2.3, Backend 2.4
- **Related**: Frontend 4.2, Frontend 4.3
- **Test Coverage**: Testing 8.4, Testing 8.5, Testing 8.6
- **Acceptance**: Groups displayed with company isolation; system groups marked clearly; delete warning shows user count; responsive on mobile

#### Subtask 4.2: Permission Assignment UI

- **Spec Reference**: User Story 1, User Story 7
- **Status**: Completed
- **Description**: Build interface for assigning permissions to groups
- **Components**:
  - `PermissionSelector` - Multi-select checkbox tree grouped by resource (candidate, job, interview)
  - `PermissionBadge` - Shows permission with cross-company indicator
  - `PermissionMetadataPanel` - Displays permission descriptions and UI labels (FR-012)
- **Depends On**: Backend 2.6, Backend 2.7
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 8.5, Testing 8.7
- **Acceptance**: Permissions grouped logically; cross-company flag visible; descriptions shown on hover; bulk assign/remove works
#### Subtask 4.3: User Assignment UI

- **Spec Reference**: User Story 3, User Story 6, User Story 12, FR-007
- **Status**: Pending
- **Description**: Build interface for assigning users to groups with expiration support
- **Components**:
  - `UserAssignmentTable` - List users in group with expiration dates
  - `AssignUserModal` - Search users, select group, set optional expiration (User Story 12 scenario 1)
  - `ExpirationWarningBadge` - Shows "Expires in 7 days" indicator (User Story 12 scenario 3)
  - `RemoveUserConfirmation` - Confirm removal with permission impact preview (User Story 3 scenario 2)
- **Depends On**: Backend 2.9, Backend 2.10
- **Related**: Frontend 4.1
- **Test Coverage**: Testing 8.8, Testing 8.9, Testing 8.10
- **Acceptance**: User search works; expiration date picker functional; warnings shown for near-expiring assignments; removal confirmed with impact

#### Subtask 4.4: Route Permission Guard Implementation

- **Spec Reference**: User Story 9, User Story 10, User Story 11, FR-013, FR-014, FR-018
- **Status**: Completed
- **Description**: Implement frontend route guards using cached permission data
- **Components**:
  - `PermissionGuard` - HOC/route wrapper checking route permissions
  - `usePermissions` hook - Access cached permission data
  - `PermissionDirective` - Declarative permission check for components (`v-permission="candidate.view"`)
  - Route configuration with permission requirements
- **Depends On**: Backend 3.2, Backend 3.3, Backend 2.13
- **Related**: Frontend 4.5, Frontend 4.2
- **Test Coverage**: Testing 8.1, Testing 8.2, Testing 8.11
- **Acceptance**: Unauthorized routes redirect to 403 page; permission checks <10ms; cache refresh on login; WebSocket updates for real-time permission changes

#### Subtask 4.5: User Permission Viewer

- **Spec Reference**: User Story 6, FR-011
- **Status**: Pending
- **Description**: Build UI to view user's effective permissions across all groups
- **Components**:
  - `UserPermissionPanel` - Shows all permissions with source groups
  - `PermissionSourceTree` - Visualizes which groups grant which permissions
  - `CompanyScopeIndicator` - Shows if permission is cross-company (backoffice only)
- **Depends On**: Backend 2.13
- **Related**: Frontend 4.3
- **Test Coverage**: Testing 8.12, Testing 8.13
- **Acceptance**: Effective permissions displayed; source groups shown; cross-company permissions flagged; updates in real-time

#### Subtask 4.6: UI Route Configuration Admin Panel

- **Spec Reference**: User Story 10, User Story 11
- **Status**: Completed
- **Description**: Build admin interface for managing UI route permission mappings
- **Components**:
  - `RoutePermissionTable` - List all routes with required permissions
  - `RoutePermissionEditor` - Edit route requirements (permissions, mode, type)
  - `RoutePermissionTester` - Test if specific users can access routes
- **Depends On**: Backend 3.2, Backend 3.4
- **Related**: Frontend 4.4
- **Test Coverage**: Testing 8.14, Testing 8.15
- **Acceptance**: All routes displayed; inline editing works; tester shows missing permissions; updates take effect in <60 seconds

---

### Main Task 5: Permission-Aware UI Components

**Status**: Pending

**Description**: Build reusable components that hide/disable UI elements based on user permissions.

**Spec Reference**: User Story 9, FR-013

#### Subtask 5.1: Permission Button Component

- **Spec Reference**: User Story 9, FR-013
- **Status**: Pending
- **Description**: Button component that auto-hides or disables based on permission
- **Props**: `permission` (string), `fallback` (hidden | disabled), `onClick`, standard button props
- **Depends On**: Frontend 4.4 (permission cache)
- **Test Coverage**: Testing 8.16
- **Acceptance**: Button hidden if no permission; disabled mode shows tooltip; <1ms render check

#### Subtask 5.2: Conditional Field Renderer

- **Spec Reference**: User Story 9 (scenario 4 - salary field masking)
- **Status**: Pending
- **Description**: Component that conditionally renders fields based on permissions
- **Props**: `permission` (string), `masked` (boolean), `maskValue` (default "\*\*\*"), children
- **Depends On**: Frontend 4.4
- **Test Coverage**: Testing 8.17
- **Acceptance**: Field masked without permission; visible with permission; works with forms

#### Subtask 5.3: Permission-Based Menu Items

- **Spec Reference**: User Story 9 (scenario 5 - admin route hiding), FR-013
- **Status**: Completed
- **Description**: Navigation menu items filtered by user permissions
- **Components**: `PermissionNav`, `PermissionMenuItem`
- **Depends On**: Frontend 4.4
- **Test Coverage**: Testing 8.18
- **Acceptance**: Unauthorized menu items hidden; nested menus supported; updates on permission change

---

## Testing

### Main Task 5: Backend Unit & Integration Tests

**Status**: Pending

**Description**: Comprehensive test coverage for authorization services, API endpoints, and permission enforcement.

**Spec Reference**: All User Stories, FR-001 through FR-024

#### Subtask 5.1: Group Management Tests

- **Spec Reference**: User Story 1, FR-001, FR-010
- **Status**: Pending
- **Description**: Test group CRUD operations and validation
- **Covers**: Backend 2.1, Backend 2.2, Backend 2.3, Backend 2.4
- **Type**: Unit + Integration
- **Scenarios**:
  - Create group with valid data succeeds
  - Duplicate group name within company fails
  - System groups cannot be deleted (User Story 1 scenario 1)
  - Non-system groups can be updated and deleted
  - Company isolation enforced (client admin cannot create groups in other company)
- **Acceptance**: All validation rules tested; edge cases covered; 90%+ code coverage

#### Subtask 5.2: Multi-Tenant Isolation Tests

- **Spec Reference**: User Story 4, FR-001
- **Status**: Pending
- **Description**: Test company data isolation for client users
- **Covers**: Backend 2.2, Backend 2.4, Backend 4.3
- **Type**: Integration
- **Scenarios**:
  - Client user only sees own company groups (User Story 4 scenario 2)
  - Client user cannot update other company groups (User Story 4 scenario 3)
  - Permissions filtered to own company (User Story 4 scenario 5)
- **Acceptance**: 100% isolation verified; no cross-company data leakage

#### Subtask 5.3: System Group Protection Tests

- **Spec Reference**: FR-010, FR-022
- **Status**: Pending
- **Description**: Test system-critical group restrictions
- **Covers**: Backend 2.1, Backend 2.4
- **Type**: Unit
- **Scenarios**:
  - System groups cannot be deleted
  - System group name cannot be changed
  - `is_system_critical` flag cannot be modified
  - Permissions can be added/removed from system groups
- **Acceptance**: All restrictions enforced; returns appropriate error codes

#### Subtask 5.4: Permission CRUD Tests

- **Spec Reference**: User Story 7, FR-008
- **Status**: Pending
- **Description**: Test permission creation and management
- **Covers**: Backend 2.5, Backend 2.6, Backend 2.7
- **Type**: Unit + Integration
- **Scenarios**:
  - Create permission with resource.action format succeeds (User Story 7 scenario 1)
  - Duplicate permission fails
  - Cross-company flag only effective for backoffice (User Story 7 scenario 4)
  - Permission assignment to group succeeds (User Story 7 scenario 2)
- **Acceptance**: All validation rules tested; format enforced; audit logged

#### Subtask 5.5: Cross-Company Permission Tests

- **Spec Reference**: User Story 5, FR-024
- **Status**: Pending
- **Description**: Test cross-company permissions for backoffice users
- **Covers**: Backend 2.5, Backend 4.3
- **Type**: Integration
- **Scenarios**:
  - Backoffice user with `is_cross_company=True` permission accesses all companies (User Story 5 scenario 1)
  - Client user with same permission is denied cross-company access (User Story 5 scenario 4, FR-024)
  - Standard permissions remain company-scoped for backoffice (User Story 5 scenario 2)
- **Acceptance**: Cross-company access works for backoffice only; client users always scoped

#### Subtask 5.6: User Assignment Tests

- **Spec Reference**: User Story 3, FR-003, FR-007
- **Status**: Pending
- **Description**: Test user-to-group assignment logic
- **Covers**: Backend 2.8, Backend 2.9, Backend 2.10
- **Type**: Integration
- **Scenarios**:
  - Assign user to group succeeds (User Story 3 scenario 1)
  - Remove user from group revokes permissions (User Story 3 scenario 2)
  - Multi-group user has union of permissions (User Story 3 scenario 3)
- **Acceptance**: Assignment persisted; permissions calculated correctly; audit logged

#### Subtask 5.7: Additive Permission Model Tests

- **Spec Reference**: User Story 3 scenario 3, FR-004
- **Status**: Pending
- **Description**: Test effective permissions calculation (union of all groups)
- **Covers**: Backend 2.13
- **Type**: Integration
- **Scenarios**:
  - User in multiple groups receives all permissions (User Story 3 scenario 3)
  - Removing one group only removes those permissions
  - Duplicate permissions across groups deduplicated
  - Effective permissions calculated in <100ms (SC-003)
- **Acceptance**: Additive model verified; performance target met; no permission conflicts

#### Subtask 5.8: Temporary Assignment Tests

- **Spec Reference**: User Story 12, FR-021, FR-023
- **Status**: Pending
- **Description**: Test expiration handling and notifications
- **Covers**: Backend 2.8, Backend 5.1, Backend 5.2
- **Type**: Integration
- **Scenarios**:
  - Create assignment with expiration date (User Story 12 scenario 1)
  - Expired assignment deactivated automatically (User Story 12 scenario 2)
  - Warning shown 7 days before expiration (User Story 12 scenario 3, FR-023)
  - Expired assignment visible in history (User Story 12 scenario 4)
- **Acceptance**: Expiration enforced within 1 hour (FR-021); notifications sent 7 days prior

#### Subtask 5.9: Audit Logging Tests

- **Spec Reference**: User Story 6, FR-009, FR-019
- **Status**: Pending
- **Description**: Test audit log capture and querying
- **Covers**: Backend 2.11, Backend 2.12
- **Type**: Integration
- **Scenarios**:
  - All permission changes logged (group create, permission assign, user assign, user remove)
  - Query logs by action type, actor, target user, date range
  - Audit log access restricted to `audit.view` permission (FR-020)
  - Performance: query returns <500ms (SC-004)
- **Acceptance**: 100% capture rate (SC-004, FR-009); query performance met; access restricted

#### Subtask 5.10: Audit Log Retention Tests

- **Spec Reference**: FR-019
- **Status**: Pending
- **Description**: Test audit log retention and archival
- **Covers**: Backend 5.3
- **Type**: Integration
- **Scenarios**:
  - Logs older than 2 years archived to S3
  - Archived logs deleted from primary DB
  - Archival process itself is audited
- **Acceptance**: Retention policy enforced; archival successful; primary DB size reduced

---

### Main Task 6: UI Route Permission Tests

**Status**: Pending

**Description**: Test route permission mapping and enforcement

**Spec Reference**: User Story 9, User Story 10, User Story 11

#### Subtask 6.1: Route Mapping CRUD Tests

- **Spec Reference**: User Story 10, FR-017
- **Status**: Pending
- **Description**: Test creation and management of route permission mappings
- **Covers**: Backend 3.1, Backend 3.2, Backend 3.4
- **Type**: Integration
- **Scenarios**:
  - Create route mapping with required permissions (User Story 10 scenario 1)
  - Update route permissions without code deployment (User Story 10 scenario 6)
  - Route uniqueness enforced
  - JSON validation for required_permissions array
- **Acceptance**: Mappings saved; validation enforced; updates reflected in <60 seconds (SC-009)

#### Subtask 6.2: Route Access Validation Tests

- **Spec Reference**: User Story 10, User Story 11, FR-018
- **Status**: Pending
- **Description**: Test route access checks with ALL/ANY modes
- **Covers**: Backend 3.1, Backend 3.3
- **Type**: Integration
- **Scenarios**:
  - User with all required permissions (mode=ALL) granted access (User Story 10 scenario 2)
  - User missing one permission (mode=ALL) denied access (User Story 11 scenario 2)
  - User with any required permission (mode=ANY) granted access (User Story 10 scenario 3, User Story 11 scenario 6)
  - Bulk check multiple routes in one request (User Story 10 scenario 4)
- **Acceptance**: Permission modes enforced correctly; bulk checks work; <100ms response

#### Subtask 6.3: Route Permission Update Tests

- **Spec Reference**: User Story 10 scenario 6, User Story 11 scenario 10
- **Status**: Pending
- **Description**: Test dynamic permission updates without frontend redeployment
- **Covers**: Backend 3.1, Backend 3.4
- **Type**: Integration
- **Scenarios**:
  - Update route from `[candidate.view]` to `[candidate.view, candidate.edit]`
  - Users without new permission lose access immediately
  - Frontend cache invalidated within 60 seconds (User Story 10 scenario 6, SC-009)
- **Acceptance**: Updates take effect immediately; no frontend deployment needed; cache invalidation works

---

### Main Task 7: API Security & Authorization Tests

**Status**: Pending

**Description**: Test permission enforcement at API level

**Spec Reference**: FR-005, FR-013

#### Subtask 7.1: Company Isolation API Tests

- **Spec Reference**: User Story 4, FR-005
- **Status**: Pending
- **Description**: Test API-level company isolation for client users
- **Covers**: Backend 2.2, Backend 4.3
- **Type**: Integration
- **Scenarios**:
  - Client user GET /api/v1/groups returns only own company (User Story 4 scenario 2)
  - Client user PUT /api/v1/groups/{other_company_group} returns 403 (User Story 4 scenario 3)
  - Client user permission check scoped to own company (User Story 4 scenario 5)
- **Acceptance**: 100% isolation enforced at API level; 403 errors returned correctly

#### Subtask 7.2: Cross-Company API Tests

- **Spec Reference**: User Story 5, FR-005
- **Status**: Pending
- **Description**: Test cross-company API access for backoffice users
- **Covers**: Backend 2.2, Backend 4.3
- **Type**: Integration
- **Scenarios**:
  - Backoffice user with cross-company permission accesses all companies (User Story 5 scenario 1)
  - Backoffice user GET /api/v1/groups returns all companies (User Story 5 scenario 3)
  - Backoffice assignment validation still enforces company matching (User Story 5 scenario 5)
- **Acceptance**: Cross-company access works; validation still enforced where appropriate

#### Subtask 7.3: Group Creation Authorization Tests

- **Spec Reference**: User Story 1, FR-005
- **Status**: Pending
- **Description**: Test authorization for group creation
- **Covers**: Backend 2.3, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `group.create` permission can create groups (User Story 1 scenario 1)
  - User without permission receives 403
  - Company admin restricted to own company
- **Acceptance**: Permission enforced; 403 on unauthorized; company isolation maintained

#### Subtask 7.4: Group Update Authorization Tests

- **Spec Reference**: User Story 1, FR-005, FR-022
- **Status**: Pending
- **Description**: Test authorization for group updates
- **Covers**: Backend 2.4, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `group.edit` can update groups
  - System group restrictions enforced (User Story 1 scenario 2)
  - Cross-company update blocked for client users
- **Acceptance**: Permission enforced; system groups protected; company isolation maintained

#### Subtask 7.5: Permission Assignment Authorization Tests

- **Spec Reference**: User Story 1, FR-005
- **Status**: Pending
- **Description**: Test authorization for assigning/removing permissions
- **Covers**: Backend 2.6, Backend 2.7, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `permission.assign` can add/remove permissions (User Story 1 scenario 1)
  - User without permission receives 403
  - Duplicate assignments prevented (User Story 1 scenario 1)
- **Acceptance**: Permission enforced; audit logged; duplicates blocked

#### Subtask 7.6: User Assignment Authorization Tests

- **Spec Reference**: User Story 3, FR-005, FR-007
- **Status**: Pending
- **Description**: Test authorization for user-to-group assignments
- **Covers**: Backend 2.9, Backend 2.10, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `user.group.assign` can assign users (User Story 3 scenario 1)
  - User with `user.group.remove` can remove users (User Story 3 scenario 2)
  - Company matching validated (User Story 3 scenario 4)
  - User type validated (User Story 3 scenario 5)
  - Global group restrictions enforced (User Story 3 scenario 6)
- **Acceptance**: Permission enforced; validation rules work; 400 on invalid assignments

#### Subtask 7.7: User Type Validation Tests

- **Spec Reference**: User Story 3 scenarios 5, 6, FR-007
- **Status**: Pending
- **Description**: Test user type and global group restrictions
- **Covers**: Backend 2.8, Backend 2.9
- **Type**: Integration
- **Scenarios**:
  - Client user cannot be assigned to `applicable_user_type='backoffice'` group (User Story 3 scenario 5)
  - Client user cannot be assigned to global groups (User Story 3 scenario 6)
  - Backoffice user can be assigned to backoffice or both types
- **Acceptance**: User type validation enforced; global group restriction enforced; 400 error with clear message

#### Subtask 7.8: Company Matching Validation Tests

- **Spec Reference**: User Story 3 scenario 4, FR-007
- **Status**: Pending
- **Description**: Test company matching during user assignment
- **Covers**: Backend 2.8, Backend 2.9
- **Type**: Integration
- **Scenarios**:
  - Company Admin cannot assign users from other companies (User Story 3 scenario 4)
  - User and group must belong to same company
  - Global groups (company=None) handled correctly
- **Acceptance**: Company matching enforced; 400 error with company mismatch message

#### Subtask 7.9: Audit Log Access Tests

- **Spec Reference**: FR-020
- **Status**: Pending
- **Description**: Test audit log access restrictions
- **Covers**: Backend 2.12, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `audit.view` can query logs (User Story 6)
  - User without permission receives 403
  - Cross-company audit logs accessible to backoffice with cross-company permission
- **Acceptance**: Permission enforced; 403 on unauthorized; cross-company access works

#### Subtask 7.10: Effective Permission API Tests

- **Spec Reference**: User Story 6, FR-011
- **Status**: Pending
- **Description**: Test user effective permissions endpoint
- **Covers**: Backend 2.13, Backend 4.1
- **Type**: Integration
- **Scenarios**:
  - User with `user.permissions.view` can view any user's permissions (User Story 6 scenario 1)
  - User can view own permissions without special permission
  - Response includes all groups and effective permissions (User Story 6 scenario 1)
  - Company scope indicators present (User Story 6 scenario 4)
- **Acceptance**: Permission enforced; complete data returned; <100ms response time (FR-003, SC-003)

#### Subtask 7.11: Permission Decorator Tests

- **Spec Reference**: FR-005
- **Status**: Pending
- **Description**: Test `@require_permission` decorator functionality
- **Covers**: Backend 4.1
- **Type**: Unit
- **Scenarios**:
  - Decorator allows access when permission present
  - Decorator blocks access when permission missing (returns 403)
  - Mode `ALL` requires all permissions
  - Mode `ANY` requires at least one permission
- **Acceptance**: Decorator works correctly; 403 on unauthorized; <10ms overhead (SC-010)

#### Subtask 7.12: Permission Enforcement Performance Tests

- **Spec Reference**: FR-005, SC-002, SC-010
- **Status**: Pending
- **Description**: Test performance of permission checks under load
- **Covers**: Backend 4.1, Backend 4.2
- **Type**: Performance
- **Scenarios**:
  - Permission check completes in <10ms (SC-010)
  - 100% of unauthorized requests blocked (SC-002)
  - Performance maintained with 10+ groups per user
- **Acceptance**: Performance targets met; no authorization bypasses; scales with group count

#### Subtask 7.13: Permission Cache Tests

- **Spec Reference**: FR-014
- **Status**: Pending
- **Description**: Test permission caching layer
- **Covers**: Backend 4.2
- **Type**: Integration
- **Scenarios**:
  - User permissions cached on login
  - Cache hit reduces permission check to <5ms
  - Cache invalidated on group assignment change
  - Cache miss falls back to database
- **Acceptance**: Cache hit rate >90%; invalidation within 1 second; fallback works

#### Subtask 7.14: Cache Invalidation Tests

- **Spec Reference**: FR-014
- **Status**: Pending
- **Description**: Test cache invalidation on permission changes
- **Covers**: Backend 4.2, Backend 2.9, Backend 2.10
- **Type**: Integration
- **Scenarios**:
  - Adding user to group invalidates cache
  - Removing user from group invalidates cache
  - Modifying group permissions invalidates all member caches
  - TTL expiration refreshes stale data
- **Acceptance**: Invalidation within 1 second; users see new permissions on next request

#### Subtask 7.15: Company Context Middleware Tests

- **Spec Reference**: User Story 4, User Story 5, FR-005
- **Status**: Pending
- **Description**: Test company context extraction and enforcement
- **Covers**: Backend 4.3
- **Type**: Integration
- **Scenarios**:
  - Client user context set to own company
  - Backoffice user context allows cross-company
  - Cross-company permission flag validated
- **Acceptance**: Context correctly set; client isolation maintained; backoffice cross-company works

#### Subtask 7.16: Assignment Expiration Job Tests

- **Spec Reference**: FR-021, User Story 12
- **Status**: Pending
- **Description**: Test scheduled expiration job
- **Covers**: Backend 5.1
- **Type**: Integration
- **Scenarios**:
  - Job runs every hour
  - Expired assignments deactivated (User Story 12 scenario 2)
  - Audit log created for expiration
  - Cache invalidated for affected users
- **Acceptance**: Job completes successfully; expiration within 1 hour (SC-012); audit logged

#### Subtask 7.17: Expiration Notification Tests

- **Spec Reference**: FR-023, User Story 12
- **Status**: Pending
- **Description**: Test expiration warning email job
- **Covers**: Backend 5.2
- **Type**: Integration
- **Scenarios**:
  - Job runs daily
  - Emails sent 7 days before expiration (User Story 12 scenario 3)
  - No duplicate sends
  - Email includes renewal instructions
- **Acceptance**: Emails sent correctly; timing accurate; no duplicates

#### Subtask 7.18: Audit Log Retention Job Tests

- **Spec Reference**: FR-019
- **Status**: Pending
- **Description**: Test audit log archival job
- **Covers**: Backend 5.3
- **Type**: Integration
- **Scenarios**:
  - Logs older than 2 years archived to S3
  - Archived logs deleted from primary DB
  - Archival process audited
- **Acceptance**: Archival successful; retention policy enforced; process logged

---

### Main Task 8: Frontend Permission Tests

**Status**: Pending

**Description**: Test frontend permission integration and UI behavior

**Spec Reference**: User Story 9, FR-013

#### Subtask 8.1: Route Guard Tests

- **Spec Reference**: User Story 9, User Story 10, FR-013
- **Status**: Pending
- **Description**: Test frontend route guards based on permissions
- **Covers**: Frontend 4.4
- **Type**: E2E
- **Scenarios**:
  - User without `admin.access` redirected from `/admin` route (User Story 9 scenario 5)
  - User with permission can access protected route
  - Permission data fetched efficiently on login (User Story 9 scenario 1)
  - Route mappings cached locally (User Story 10 scenario 4)
- **Acceptance**: Unauthorized routes blocked; <10ms check time (SC-007); cache works

#### Subtask 8.2: Bulk Permission Check Tests

- **Spec Reference**: User Story 9 scenario 2, FR-016
- **Status**: Pending
- **Description**: Test bulk permission check API usage from frontend
- **Covers**: Frontend 4.4, Backend 3.3
- **Type**: Integration
- **Scenarios**:
  - Frontend checks multiple permissions in single request (User Story 9 scenario 2)
  - Response indicates which permissions user has
  - <200ms response time (SC-006)
- **Acceptance**: Bulk check works; single request; performance target met

#### Subtask 8.3: Route Configuration Admin Tests

- **Spec Reference**: User Story 10, User Story 11
- **Status**: Pending
- **Description**: Test UI route configuration admin panel
- **Covers**: Frontend 4.6, Backend 3.4
- **Type**: E2E
- **Scenarios**:
  - Admin creates route mapping (User Story 10 scenario 1)
  - Admin updates route permissions (User Story 10 scenario 6, User Story 11 scenario 10)
  - Route tester shows missing permissions
  - Changes take effect within 60 seconds (User Story 10 scenario 6)
- **Acceptance**: Admin panel functional; updates reflected; performance target met (SC-009)

#### Subtask 8.4: Group List Tests

- **Spec Reference**: User Story 1, FR-006
- **Status**: Pending
- **Description**: Test group management page
- **Covers**: Frontend 4.1
- **Type**: E2E
- **Scenarios**:
  - Client user sees only own company groups (User Story 4)
  - Backoffice user sees all companies (User Story 5)
  - System groups marked clearly
  - Search and filter work
- **Acceptance**: Groups displayed correctly; isolation enforced; UI responsive

#### Subtask 8.5: Group Creation Tests

- **Spec Reference**: User Story 1, FR-006
- **Status**: Pending
- **Description**: Test group creation workflow
- **Covers**: Frontend 4.1, Backend 2.3
- **Type**: E2E
- **Scenarios**:
  - Admin creates group with permissions (User Story 1 scenario 1)
  - Duplicate name rejected
  - Company auto-set for client users
- **Acceptance**: Group created in <2 minutes (SC-001); validation works; success feedback shown

#### Subtask 8.6: Group Deletion Tests

- **Spec Reference**: User Story 8, FR-010
- **Status**: Pending
- **Description**: Test group deletion with warnings
- **Covers**: Frontend 4.1, Backend 2.4
- **Type**: E2E
- **Scenarios**:
  - Deleting group shows affected user count (User Story 8 scenario 1)
  - System groups cannot be deleted
  - Confirmation required before deletion
- **Acceptance**: Warning displayed; system groups protected; confirmation enforced

#### Subtask 8.7: Permission Assignment Tests

- **Spec Reference**: User Story 1, User Story 7
- **Status**: Pending
- **Description**: Test permission assignment UI
- **Covers**: Frontend 4.2, Backend 2.6
- **Type**: E2E
- **Scenarios**:
  - Admin assigns permissions to group (User Story 1 scenario 1)
  - Cross-company permissions flagged (User Story 7 scenario 4)
  - Permission descriptions shown
- **Acceptance**: Assignment works; cross-company indicator visible; descriptions helpful

#### Subtask 8.8: User Assignment Tests

- **Spec Reference**: User Story 3, FR-007
- **Status**: Pending
- **Description**: Test user-to-group assignment UI
- **Covers**: Frontend 4.3, Backend 2.9
- **Type**: E2E
- **Scenarios**:
  - Admin assigns user to group (User Story 3 scenario 1)
  - User search works
  - Permissions take effect immediately (User Story 3 scenario 1)
- **Acceptance**: Assignment succeeds; search functional; immediate effect verified

#### Subtask 8.9: User Removal Tests

- **Spec Reference**: User Story 3, FR-007
- **Status**: Pending
- **Description**: Test removing user from group
- **Covers**: Frontend 4.3, Backend 2.10
- **Type**: E2E
- **Scenarios**:
  - Admin removes user from group (User Story 3 scenario 2)
  - Permissions revoked immediately (User Story 3 scenario 2)
  - Confirmation shown
- **Acceptance**: Removal succeeds; permissions lost immediately; confirmation required

#### Subtask 8.10: Temporary Assignment Tests

- **Spec Reference**: User Story 12
- **Status**: Pending
- **Description**: Test temporary assignment UI
- **Covers**: Frontend 4.3, Backend 2.9
- **Type**: E2E
- **Scenarios**:
  - Admin sets expiration date (User Story 12 scenario 1)
  - Warning shown for near-expiring assignments (User Story 12 scenario 3)
  - Expired assignments visible in history (User Story 12 scenario 4)
- **Acceptance**: Expiration date picker works; warnings shown; history accessible

#### Subtask 8.11: Permission Button Tests

- **Spec Reference**: User Story 9, FR-013
- **Status**: Pending
- **Description**: Test permission-aware button component
- **Covers**: Frontend 5.1
- **Type**: Unit + E2E
- **Scenarios**:
  - Button hidden when permission missing (User Story 9 scenario 3)
  - Button visible when permission present
  - Disabled mode shows tooltip
- **Acceptance**: Button behavior correct; <1ms render check; tooltip informative

#### Subtask 8.12: Multi-Group Permission Display Tests

- **Spec Reference**: User Story 3 scenario 3, User Story 6
- **Status**: Pending
- **Description**: Test effective permissions display for multi-group users
- **Covers**: Frontend 4.5, Backend 2.13
- **Type**: E2E
- **Scenarios**:
  - User in multiple groups sees union of permissions (User Story 3 scenario 3, User Story 6 scenario 1)
  - Source groups displayed (User Story 6 scenario 2)
  - No duplicate permissions shown
- **Acceptance**: Effective permissions displayed; source groups shown; deduplication works

#### Subtask 8.13: Permission Viewer Tests

- **Spec Reference**: User Story 6
- **Status**: Pending
- **Description**: Test user permission viewer UI
- **Covers**: Frontend 4.5, Backend 2.13
- **Type**: E2E
- **Scenarios**:
  - Admin views user's effective permissions (User Story 6 scenario 1)
  - Cross-company permissions flagged (User Story 6 scenario 4)
  - Group membership shown (User Story 6 scenario 2)
- **Acceptance**: Complete permissions displayed; cross-company indicator works; group membership visible

#### Subtask 8.14: Route Permission Table Tests

- **Spec Reference**: User Story 10
- **Status**: Pending
- **Description**: Test route permission configuration table
- **Covers**: Frontend 4.6
- **Type**: E2E
- **Scenarios**:
  - All routes displayed with required permissions
  - Filter by permission to find routes
  - Inline editing works
- **Acceptance**: All routes visible; filter functional; editing smooth

#### Subtask 8.15: Route Permission Tester Tests

- **Spec Reference**: User Story 10
- **Status**: Pending
- **Description**: Test route permission tester component
- **Covers**: Frontend 4.6, Backend 3.3
- **Type**: E2E
- **Scenarios**:
  - Tester shows if user can access route
  - Missing permissions displayed
  - Multiple users can be tested
- **Acceptance**: Tester accurate; missing permissions shown; multiple tests possible

#### Subtask 8.16: Conditional Field Tests

- **Spec Reference**: User Story 9 scenario 4
- **Status**: Pending
- **Description**: Test field masking based on permissions
- **Covers**: Frontend 5.2
- **Type**: E2E
- **Scenarios**:
  - Salary field masked without `salary.view` (User Story 1 scenario 2, User Story 9 scenario 4)
  - Field visible with permission
  - Custom mask values work
- **Acceptance**: Masking works; field visible with permission; customization supported

#### Subtask 8.17: Field Masking Tests

- **Spec Reference**: User Story 1 scenario 2, User Story 9 scenario 4
- **Status**: Pending
- **Description**: Test conditional field renderer component
- **Covers**: Frontend 5.2
- **Type**: Unit + E2E
- **Scenarios**:
  - Field masked when permission missing (User Story 1 scenario 2)
  - Field visible when permission present
  - Works with forms and inputs
- **Acceptance**: Masking correct; visibility correct; form integration works

#### Subtask 8.18: Permission Menu Tests

- **Spec Reference**: User Story 9 scenario 5, FR-013
- **Status**: Pending
- **Description**: Test permission-based navigation menu
- **Covers**: Frontend 5.3
- **Type**: E2E
- **Scenarios**:
  - Unauthorized menu items hidden (User Story 9 scenario 5)
  - Nested menus filtered correctly
  - Menu updates on permission change (User Story 9 scenario 6)
- **Acceptance**: Menu filtered correctly; nested menus work; real-time updates

---

## Task Summary

| Area      | Main Tasks | Subtasks |
| --------- | ---------- | -------- |
| Database  | 1          | 6        |
| Backend   | 5          | 26       |
| Frontend  | 5          | 12       |
| Testing   | 3          | 31       |
| **Total** | **14**     | **75**   |

**Priority Distribution**:

- P1 (MVP): User Stories 1-5, 9 = ~55 tasks (core RBAC, multi-tenant isolation, permission enforcement, frontend integration)
- P2 (Enhancement): User Stories 6-8, 10-12 = ~20 tasks (audit viewer, custom permissions, route mapping, temporary assignments)

---

## Dependencies & Sequencing

### Phase 1: Foundation (Week 1-2)

**Blockers**: Database schema must complete first

- Database 1.1-1.6 (all tables) → blocks all backend work
- Backend 2.1, 2.5, 2.8 (core service logic)
- Backend 4.1 (permission decorator)

**Why**: Service logic depends on database; permission enforcement needed before APIs

### Phase 2: Core APIs (Week 3-4)

**Can Parallelize**: Backend APIs and caching

- Backend 2.2-2.4 (group APIs)
- Backend 2.6-2.7 (permission APIs)
- Backend 2.9-2.10 (assignment APIs)
- Backend 4.2-4.3 (caching and middleware)

**Depends On**: Phase 1 service logic

### Phase 3: Frontend Core (Week 4-5)

**Can Parallelize**: UI components after API completion

- Frontend 4.1-4.3 (group/permission/assignment UIs)
- Frontend 4.4 (route guards)
- Frontend 5.1-5.3 (permission-aware components)
- Backend 2.11-2.13 (audit and permission query APIs)

**Depends On**: Phase 2 APIs available

### Phase 4: Advanced Features (Week 6)

**Enhancement Features**: Can be done after MVP

- Backend 3.1-3.4 (UI route mapping)
- Frontend 4.5-4.6 (permission viewer and route admin)
- Backend 5.1-5.3 (background jobs)

**Depends On**: Phase 3 core features working

### Phase 5: Testing & Polish (Week 7-8)

**Comprehensive Testing**: Can parallelize test categories

- Testing 5.1-5.10 (backend unit/integration tests)
- Testing 6.1-6.3 (route permission tests)
- Testing 7.1-7.18 (API security tests)
- Testing 8.1-8.18 (frontend E2E tests)

**Depends On**: Implementation complete for test coverage

---

## Notes

### System Groups Seeding

During initial migration or system setup, the following system groups must be created with `is_system_critical=True`:

1. **Super Admin** (global, `applicable_user_type='backoffice'`)
   - All permissions with `is_cross_company=True`
2. **Company Admin** (per-company, `applicable_user_type='client'`)
   - All company-scoped permissions for their company
3. **Support Agent** (global, `applicable_user_type='backoffice'`)
   - Read-only cross-company permissions: `ticket.view`, `company.view`, `user.view`

### Permission Seeding

All permissions from **Appendix A** in the spec must be seeded during initial deployment. Use a Django management command or database migration to insert these records into `auth_permissions` table.

### Performance Optimization Notes

- **Database Indexes**: All foreign keys indexed; composite indexes on frequently queried columns (`user_id + is_active`, `company_id + created_at`)
- **Permission Caching**: Redis cache with 1-hour TTL; invalidation on assignment change
- **Query Optimization**: Use `select_related()` and `prefetch_related()` for group/permission queries to avoid N+1 problems
- **Audit Log Partitioning**: Partition by month for faster queries and easier archival

### Multi-Tenant Security Considerations

- **Client User Restrictions**:
  - ALWAYS filter by `user.company_id` in service layer
  - Cannot access global groups (`company=None`)
  - Cannot be assigned to backoffice-only groups
  - Cross-company flags ignored (always company-scoped)
- **Backoffice User Capabilities**:
  - Can have cross-company permissions via `is_cross_company=True` flag
  - Can view all companies when granted appropriate permissions
  - Still subject to permission checks (not bypass)

### Open Questions

- **Q1**: Should we implement a "God Mode" login-as feature for Super Admins to debug permission issues?
  - **Answer**: Feature 006 addresses this; coordinate implementation
- **Q2**: Should expired assignments be soft-deleted or remain in the table forever?
  - **Answer**: Remain in table with `is_active=False` for audit trail; no automatic deletion
- **Q3**: Should we support hierarchical groups (groups containing other groups)?
  - **Answer**: Out of scope for Phase 2; revisit in Phase 3 if needed

---

## Validation Summary

✅ All task references point to existing tasks  
✅ Dependency flow is correct (Database → Backend → Frontend → Testing)  
✅ P1 tasks have comprehensive test coverage (Testing 5-8 cover all core functionality)  
✅ Multi-tenant isolation requirements addressed in User Stories 4, 5 and tested in Testing 7.1, 7.2, 7.15  
✅ Cross-company permissions restricted to backoffice only (FR-024, User Story 5, Testing 5.5, 7.2)  
✅ All acceptance criteria from spec referenced in test scenarios
