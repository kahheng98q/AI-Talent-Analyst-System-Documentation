# Feature Specification: System Behavior - Interview Quota Limit Enforcement

**Feature Branch**: `008-system-behavior-limit-enforcement`

**Created**: 2025-12-22
**Status**: Draft
**Input**: Extracted from Feature 005 - Interview quota and balance enforcement for interview invitations
**Note**: This feature focuses exclusively on interview quota limit enforcement. All interview subscription and billing logic is handled by Feature 005. Feature 008 provides the system-level enforcement layer for blocking actions that exceed interview quota.

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Block Interview Invitations When Quota Exhausted (Priority: P1)

As a System, I must block `Send Invitation` actions when a company exceeds their interview quota to prevent unauthorized interviews beyond their subscription limits.

**Why this priority**: Prevents revenue leakage by ensuring companies cannot send interviews beyond purchased quota. Core protection mechanism for subscription model.

**Independent Test**: Attempt to send interview invitation with zero remaining quota and verify system blocks the action with appropriate error message and CTA to upgrade/purchase.

**Acceptance Scenarios**:

1. **Given** a Company with Gold Fish subscription (300 interviews/year) has remaining available quota of 50 interviews and account balance RM100 (sufficient for estimated interview cost RM50), **When** they send an interview invitation, **Then** the system allows the action, reserves 1 interview quota, creates a QuotaReservation record, and the company's available quota becomes 49 interviews

2. **Given** a Company with mixed quota (plan quota exhausted but 5 individual interviews remaining) and sufficient account balance, **When** they send an invitation, **Then** the system allows the action and reserves 1 interview from individual interviews counter, reducing individual interviews remaining to 4

3. **Given** a Company with Gold Fish subscription (300 interviews/year) has used all 300 interviews and has zero reserved quota, **When** they attempt to send an interview invitation without purchasing additional interviews, **Then** the system blocks the action with message "Interview quota exhausted (300/300). Upgrade your plan or purchase individual interviews at RM15 each to continue." with [Upgrade] and [Purchase Now] buttons

4. **Given** a Company with available quota (50 remaining interviews) but insufficient account balance (RM10 available, estimated interview cost RM50), **When** they attempt to send an invitation, **Then** the system blocks the action with message "Insufficient account balance (RM10 available, RM50 required). Please purchase additional balance to continue." with [Purchase Balance] button

5. **Given** a Company that downgrades from Whale (2000/year) to Gold Fish (300/year) with current usage at 600 interviews, **When** they attempt to send new invitations after downgrade, **Then** the system blocks the action with message "Your new plan quota (300 interviews) is below current usage (600 interviews). New invitations are blocked until usage falls below 300. Upgrade plan or delete old interviews to continue."

6. **Given** a Company with subscription status "Cancelled" or "Suspended", **When** they attempt to send an interview invitation, **Then** the system blocks with message "No active subscription plan. Please subscribe to a plan to send invitations." with [View Plans] button

---

### Edge Cases

- **Race Conditions**: Multiple concurrent `Send Invitation` requests near quota limit are handled atomically; only requests within available quota succeed.
- **Quota Reservation and Release**: Quota is reserved at invitation send; released on expiry/cancellation, consumed on completion. System prevents double-booking through atomic transactions.
- **Downgrade with Over-Quota Usage**: If company downgrades plan reducing quota below current usage, existing interviews remain readable but new invitations blocked until usage falls below new quota.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: Enforce interview quota and balance at API/service layer before invitation send: When HR triggers `Send Invitation`, system MUST check: (1) company has active subscription plan, (2) company has remaining interview quota (subscription_interviews_remaining + individual_interviews_remaining - reserved_interviews_count > 0), (3) company has sufficient account balance to cover estimated interview cost (balance >= estimated_cost). If any check fails, block with error message. Prepaid model: company must pre-fund balance before sending invitations to prevent balance from going negative.

- **FR-002**: Provide clear, actionable error messages: Error messages MUST include current usage, limit, and CTA (Upgrade/Purchase/Top Up) matching Feature 005 messaging standards.

- **FR-003**: Prevent quota double-booking via atomic reservation: `Send Invitation` MUST atomically reserve one interview quota and create QuotaReservation record before accepting the invitation. If quota insufficient OR balance insufficient, block before any database writes. This prevents balance from going negative.

- **FR-004**: Handle race conditions on concurrent invitations: Multiple concurrent `Send Invitation` requests near quota limit MUST be handled atomically; only requests within available quota succeed. Failing requests receive clear error messages.

- **FR-005**: Enforce downgrade quota restrictions: After subscription downgrade, if current interview usage exceeds new quota, block new invitations with message directing user to Feature 005 downgrade workflow. Existing interviews remain accessible.

- **FR-006**: Integrate with Feature 005 data: System MUST read quota data from Feature 005 tables (CompanySubscription, QuotaReservation, BillingTransaction) to determine available quota and balance. No separate quota storage in Feature 008.

- **FR-007**: Block invitations with inactive plans: If company subscription status is "Cancelled" or "Suspended", block all invitations with message to reactivate subscription (Feature 005 responsibility).

### Key Entities

- **CompanySubscription** (from Feature 005): Provides current quota counters, balance, reservation count
- **QuotaReservation** (from Feature 005): Tracks reserved interviews for pending invitations
- **BillingTransaction** (from Feature 005): Audit log for all quota consumption
- **InterviewInvitation**: Local entity (created by Feature 001) that references quota reservation status

### Dependencies

- **Feature 001**: AI-HR Interview System (triggers `Send Invitation` action that requires quota check)
- **Feature 005**: Credit & Subscription Management (provides quota data, balance, reservation tracking, threshold alerts)

### Assumptions

- **Quota Source**: Interview quota comes from Feature 005's subscription plans (Gold Fish: 300, Dolphin: 800, Whale: 2000) + individual purchases (RM15 each)
- **Prepaid Model - No Negative Balance**: Company must pre-fund account with sufficient balance BEFORE sending invitations. Invitations are blocked if balance is insufficient. Balance is NOT deducted at invitation (reservation only), but actual settlement happens at completion (Feature 005 responsibility).
- **Check Timing**: Both quota AND balance enforced at `Send Invitation` time to prevent negative balance. Actual balance deduction at completion time.
- **Atomic Transactions**: All quota + balance checks and reservations use database transactions to prevent race conditions
- **No Grace Period**: No grace period after subscription expiration; service stops immediately when plan cancelled
- **Error Messages**: Align with Feature 005 messaging for consistency (e.g., "Interview quota exhausted (300/300)", "Insufficient account balance (RM10 available, RM50 required)")

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: 100% of quota-exceeding invitations blocked before QuotaReservation creation
- **SC-002**: Quota and balance enforcement check adds < 100ms latency to invitation send request (atomic database queries only)
- **SC-003**: Zero race conditions resulting in quota over-reservations (verified by audit trail)
- **SC-004**: Error messages include current quota, remaining quota, and next action CTA
- **SC-005**: Downgrade enforcement prevents new invitations when usage > new quota limit (100% accuracy)
- **SC-006**: All quota checks logged in BillingTransaction for audit trail with timestamp and attempting user
