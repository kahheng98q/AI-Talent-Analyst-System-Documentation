# Feature Specification: Company Admin Self Service

**Feature Branch**: `007-company-admin-self-service`

**Created**: 2025-12-22
**Status**: Draft
**Input**: Extracted from Feature 002 - Company Admin self-service capabilities

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Manage HR Users (Priority: P2)

As a Company Admin, I want to manage my team's access.

**Why this priority**: Basic administration capability required for any B2B SaaS.

**Independent Test**: Add/remove a user as a Company Admin and verify access.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin, **When** I view my user list, **Then** I should see all users belonging to my company.
2. **Given** an employee has left, **When** I remove/deactivate their user account, **Then** they should no longer be able to log in.

### User Story 2 - Invite Users (Priority: P2)

As a Company Admin, I want to invite colleagues via email.

**Why this priority**: Streamlines user onboarding and reduces admin friction.

**Independent Test**: Send an invite and verify the email receipt and registration flow.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin, **When** I send an invitation to `colleague@example.com`, **Then** the user should receive an email with a unique registration link.
2. **Given** an uninvited user, **When** they try to register, **Then** they should be blocked (assuming closed registration).
3. **Given** a user with an invite link, **When** they complete the form, **Then** they should be able to set a password and join the company.

### User Story 3 - Manage Subscription (Priority: P1)

As a Company Admin, I want to upgrade my plan to access more features.

**Why this priority**: Directly drives revenue; essential for monetization.

**Independent Test**: Simulate an upgrade flow and verify plan limits change.

**Acceptance Scenarios**:

1. **Given** I am on a "Starter" plan, **When** I click "Upgrade" and complete the Stripe Checkout, **Then** my plan should be updated to "Pro".
2. **Given** the upgrade is successful, **When** I check my limits (e.g., max candidates), **Then** they should reflect the new plan's allowances immediately.

---

### Edge Cases

- **Self-Deletion**: Company Admins cannot delete their own account if they are the only admin; they must assign another admin first.
- **Stripe Failures**: Webhook failures trigger a retry; if ultimately failed, the subscription remains in the previous state until manual intervention.
- **Duplicate Invitations**: Sending multiple invites to the same email uses the latest token and invalidates previous ones.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST allow Company Admins to view and manage all users within their company.
- **FR-002**: System MUST support User Invitation via email with unique registration links.
- **FR-003**: System MUST handle Subscription management (Stripe integration) for plan upgrades/downgrades.
- **FR-004**: System MUST prevent Company Admins from deleting their own account if they are the only admin.
- **FR-005**: System MUST integrate with Feature 004 (User Authorization Groups & Permissions) for user role assignment.

### Key Entities

- **User**: System user with Role (Soft Delete required for audit trails).
- **Invitation**: Pending user access with unique token and expiration.
- **Subscription**: Link between Company and Plan for billing management.
- **Plan**: Subscription tier with Limits and features.

### Dependencies

- **Feature 002**: SaaS Admin Portal (for company and user data structures).
- **Feature 004**: User Authorization Groups & Permissions (for RBAC enforcement).
- **Feature 005**: Credit & Subscription Management (for billing and plan limits).

## Assumptions

- **Stripe**: Used for all billing and subscription management.
- **Invitation Validity**: Invitation links expire after 7 days.
- **Deletion**: "Removing" a user performs a soft delete to preserve historical data.
- **Email Service**: Transactional email service (e.g., SendGrid, AWS SES) is configured.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Company Admin can add/remove users in < 3 clicks.
- **SC-002**: Invitation emails delivered within 30 seconds of sending.
- **SC-003**: Subscription upgrades reflect new limits immediately (< 5 seconds).
- **SC-004**: 95% of user management actions complete in < 2 seconds.
