# Feature Specification: Credit & Subscription Management

**Feature Branch**: `005-credit-subscription-management`  
**Created**: December 13, 2025  
**Status**: Draft  
**Input**: User description: "Tiered subscription packages (Gold Fish RM3600, Dolphin RM7200, Whale RM12000 per year) with per-interview billing. Individual interview purchase at RM15. Job Description creation is free. Subscription management, billing history, payment notifications, usage tracking."

> **Admin Portal Status**: Not Implemented in `ai-talent-analyst-system-admin-portal`. This feature is planned for the company admin portal but has not been built yet.

## User Scenarios & Testing _(mandatory)_

### User Story 1 - View Subscription Status & Interview Allocation (Priority: P1) `❌ Not Implemented`

Company Admins need to monitor their subscription tier, annual interview allocation, interviews used this year, account balance, and reserved quota to manage their recruitment capacity.

**Why this priority**: Core visibility into subscription status, interview quota, and account balance is fundamental for planning recruitment activities. Without this, users cannot track their usage or make timely upgrade decisions.

**Independent Test**: Can be fully tested by logging in as Company Admin, navigating to subscription dashboard, and verifying that subscription tier, annual interview allocation, interviews used this year, remaining interviews, reserved quota (pending invitations), available quota, and account balance are displayed accurately. Delivers immediate value by providing transparency into recruitment capacity and financial status.

**Acceptance Scenarios**:

1. **Given** I am logged in as a Company Admin with an active Gold Fish subscription (subscribed on Dec 1, 2025), account balance RM500, and 5 pending invitations, **When** I navigate to the subscription dashboard, **Then** I see subscription status "Active", subscription tier "Gold Fish (RM3600/year)", annual allocation "300 interviews", interviews used "12", remaining "288", reserved quota "5" (pending invitations), available quota "283", account balance "RM500", and subscription anniversary date "Dec 1, 2026"

2. **Given** I am logged in as a Company Admin with Dolphin subscription and have used 650 of 800 interviews this year, with account balance RM1200 and no reserved quota, **When** I view the subscription dashboard on December 13, 2025, **Then** I see "Remaining: 150 interviews", "Usage Rate: 81.25% of annual allocation", "Reserved Quota: 0", "Available Quota: 150", "Account Balance: RM1200"

3. **Given** I am logged in as a Company Admin with Gold Fish subscription and have used 280 of 300 interviews (93% used), **When** I view the subscription dashboard, **Then** I see a warning banner "⚠️ Running Low: 20 interviews remaining. Upgrade to continue interviewing after quota is exhausted. [Upgrade Now]"

4. **Given** my subscription interview quota has been fully used (300/300 interviews for Gold Fish) but I have RM500 available balance, **When** I navigate to the subscription dashboard, **Then** I see status banner "❌ Interview Quota Exhausted", "0 remaining interviews", account balance "RM500 available", and a prominent call-to-action "Upgrade your plan or purchase individual interviews at RM15 each" with [View Plans] button

5. **Given** I have available quota (200 remaining interviews) but only RM30 available balance (insufficient for 1 interview estimated at RM50), **When** I navigate to the subscription dashboard, **Then** I see warning banner "⚠️ Low Balance: RM30 available (insufficient for new interviews). Top up to continue interviewing. [Top Up Now]" alongside the quota status "200 interviews remaining"

---

### User Story 2 - View Interview Usage History & Analytics (Priority: P1) `❌ Not Implemented`

Company Admins need to analyze interview consumption patterns over time to track hiring progress and forecast remaining quota.

**Why this priority**: Usage analytics are essential for recruitment planning. Without historical data visualization, admins cannot identify hiring trends or make timely upgrade decisions.

**Independent Test**: Can be fully tested by creating sample interview records across different dates and statuses, then verifying that the dashboard displays accurate line graphs (daily/monthly views), date range filtering works, and pie charts show correct usage breakdowns by interview status (completed, cancelled, pending). Delivers standalone value by enabling data-driven hiring decisions.

**Acceptance Scenarios**:

1. **Given** my company has interview records for the past 3 months, **When** I view the interview usage chart in "Daily" mode with date range "Dec 1-13, 2025", **Then** I see a line graph with 13 data points showing daily interviews completed, with dates on X-axis and interview count on Y-axis

2. **Given** my company has interview data, **When** I switch to "Monthly" view with date range "Oct 2025 - Dec 2025", **Then** I see a line graph with 3 data points showing aggregate monthly interviews completed

3. **Given** my company completed: 50 interviews in "Completed" status, 8 in "Pending" status, 2 in "Cancelled" status in December 2025, **When** I view the usage breakdown pie chart for December 2025, **Then** I see 3 segments: "Completed" (86.2%), "Pending" (13.8%), "Cancelled" (3.4%)

4. **Given** I have interview usage data, **When** I view the combined dual-axis chart, **Then** I see a line graph for interviews over time overlaid with a bar chart showing monthly breakdown for the selected period

5. **Given** I select an invalid date range (end date before start date), **When** I apply the filter, **Then** I see an error message "Invalid date range: End date must be after start date" and the chart retains the previous valid filter

---

### User Story 3 - View Billing History & Invoices (Priority: P1) `❌ Not Implemented`

Company Admins need to access past invoices, receipts, and payment history for accounting, auditing, and expense tracking purposes.

**Why this priority**: Billing transparency and invoice access are mandatory for any B2B SaaS product. Finance departments require downloadable invoices for bookkeeping and tax compliance.

**Independent Test**: Can be fully tested by creating sample billing records with various transaction types and dates, then verifying that the billing history table displays correctly, filters work (date range, transaction type), and PDF downloads generate proper invoices. Delivers immediate value for financial record-keeping.

**Acceptance Scenarios**:

1. **Given** my company has 6 months of billing history, **When** I navigate to the billing history page, **Then** I see a table with columns: Invoice Number, Date, Description, Amount, Transaction Type (Subscription/Individual Purchase/Top-up), and Actions (View/Download PDF)

2. **Given** I have billing records from Jan 2025 to Dec 2025, **When** I filter by date range "Oct 2025 - Dec 2025" and transaction type "Subscription", **Then** I see only subscription invoices from October, November, and December 2025

3. **Given** I select an invoice from the billing history table, **When** I click "Download PDF", **Then** a PDF invoice is generated and downloaded containing: company name, invoice number, date, itemized charges (subscription fee, individual interview purchases, or balance top-up), total amount, payment method used, and payment completion timestamp

4. **Given** I have made a balance top-up via bank transfer for RM500 on November 15, 2025, **When** I view the billing history after the finance team processes my payment, **Then** I see an invoice record with transaction type "Balance Top-up", date "Nov 15, 2025", amount "RM500", and I can download a receipt showing the top-up confirmation and new account balance

5. **Given** I have no billing history (new account), **When** I navigate to the billing history page, **Then** I see an empty state message "No billing history available" with information about when the first invoice will be generated

---

### User Story 4 - Receive Interview Quota Alerts (Priority: P2) `❌ Not Implemented`

Company Admins need automated alerts when interview usage reaches critical thresholds (80%, 100%) to plan upgrade or purchase decisions.

**Why this priority**: Automated notifications prevent recruitment disruptions. While not MVP-critical (users can manually check dashboard), it significantly improves user experience and reduces support burden.

**Independent Test**: Can be fully tested by simulating interview usage that crosses 80% and 100% of annual quota thresholds AND balance that drops below RM50 threshold, then verifying that email notifications are sent and in-app banners/modals appear with correct messaging. Delivers value by enabling proactive quota and balance management.

**Acceptance Scenarios**:

1. **Given** I am subscribed to Gold Fish plan (300 interviews/year) and have used exactly 240 interviews (80%), **When** the system calculates remaining quota, **Then** I receive an email notification with subject "Interview Quota Alert: 80% Used" and body detailing current usage, remaining interviews, and link to upgrade plan

2. **Given** I am subscribed to Gold Fish plan and have used exactly 240 of 300 interviews (80%), **When** I next log in to the system, **Then** I see an in-app banner at the top of the dashboard: "⚠️ You have used 80% of interviews (240/300). 60 interviews remaining."

3. **Given** my company has used exactly 300 interviews (100%) of Gold Fish annual quota, **When** the system calculates remaining quota, **Then** I receive an email notification with subject "Interview Quota Exhausted - Upgrade or Purchase Now" and body with options to upgrade plan or purchase individual interviews at RM15 each

4. **Given** I have already received an 80% notification, **When** my usage increases from 240 to 250 interviews (still under 100%), **Then** I do NOT receive duplicate 80% notifications (notification sent once per threshold per billing year)

5. **Given** I have used 100% of quota, **When** a new subscription anniversary date arrives and quota resets, **Then** the notification flags are reset and I can receive new threshold notifications for the new year

6. **Given** my account available balance drops to RM45 (below RM50 threshold) after interview completion settlement, **When** the system processes the balance update, **Then** I receive an email notification with subject "Low Account Balance Alert" and body stating "Your available balance is RM45. Top up now to avoid interview disruptions." with [Top Up Now] link

7. **Given** my available balance is RM30 and I have remaining quota of 200 interviews, **When** I attempt to send an interview invitation (estimated cost RM50), **Then** the system blocks the action with message "Insufficient account balance (RM30 available, RM50 required). Please top up your account to continue." and a [Top Up Now] button

8. **Given** I have already received a low balance notification when balance dropped to RM45, **When** my balance increases to RM200 after top-up, and then later drops to RM40, **Then** I receive a new low balance notification (notifications are retriggered when balance crosses threshold again)

---

### User Story 5 - Purchase Individual Interviews (Priority: P1) `❌ Not Implemented`

Company Admins on any subscription tier can purchase individual interviews at RM15 each when they run out of annual quota or want to conduct additional interviews. Purchases add to quota counter and deduct from account balance.

**Why this priority**: Individual interview purchase provides flexibility for hiring spikes and is essential for companies that exceed their subscription quota. This directly enables revenue and prevents customer churn.

**Independent Test**: Can be fully tested by simulating quota exhaustion, initiating individual interview purchase, paying via bank transfer, and verifying that the purchase is credited to their quota counter AND deducted from balance, allowing new interview invitations. Delivers immediate value by enabling recruitment continuation.

**Acceptance Scenarios**:

1. **Given** I am a Company Admin with Gold Fish subscription (300 interviews/year) and have used all 300 interviews with account balance RM500, **When** I attempt to send a new invitation, **Then** the system blocks the action with message "Interview quota exhausted (300/300). Purchase individual interviews at RM15 each to continue." with a [Purchase Now] button

2. **Given** I click "Purchase Individual Interviews", **When** I select quantity "10 interviews", **Then** I see a checkout summary: "10 interviews × RM15 = RM150.00 total", payment instructions for bank transfer, unique reference code for this purchase, and note "RM150 will be deducted from your account balance (RM500 available) and 10 interviews will be added to your quota."

3. **Given** I make a bank transfer for RM150 for 10 individual interviews and email the receipt to billing@company.com with my account balance at RM500, **When** the finance team processes my receipt, **Then** my individual interview quota is credited with 10 interviews, my account balance is reduced to RM350, BillingTransaction is created (type: individual_purchase_debit, amount: RM150), and I receive email confirmation "10 additional interviews purchased. Account balance: RM350 remaining."

4. **Given** I have purchased 10 individual interviews and used 8 of them, **When** I view my subscription dashboard, **Then** I see "Plan Quota: 300/300 used", "Individual Interviews: 2 remaining", and "Total Available: 2 interviews"

5. **Given** I have account balance RM10 (insufficient for 1 interview at RM15), **When** I attempt to purchase individual interviews, **Then** the system blocks the purchase with message "Insufficient account balance (RM10 available, RM15 required per interview). Please top up your account first." with [Top Up Now] button

6. **Given** I have mixed quota (plan + individual interviews), **When** I send invitations, **Then** the system consumes plan quota first, and only uses individual interviews after plan quota is exhausted

---

### Edge Cases

- What happens when a company exceeds their annual interview quota? (CLARIFIED: Block `Send Invitation` with message about upgrading plan or purchasing individual interviews. Interview cannot proceed without available quota.)
- What happens when an HR invites 10 candidates but only 5 complete interviews? (CLARIFIED: Quota is reserved on invitation send. On completion: reserved quota is consumed and actual cost is settled against account balance. On expiry: reserved quota is released, quota unchanged.)
- What happens when company has quota but insufficient balance? (NEW: Block `Send Invitation` with message "Insufficient balance (RM50 available, RM100 required). Please top up your account to continue.")
- What happens if estimated cost is RM100 but actual cost is RM120 (overage)? (NEW: System settles RM120, consuming RM20 more than reserved. If this causes negative balance, trigger alert to admin to top up.)
- What happens if estimated cost is RM100 but actual cost is RM80 (underrun)? (RM80 deducted from balance, RM20 released back to available balance)
- How does the system handle interview cancellations? (Cancelled interviews do NOT refund quota consumption. Once an interview is completed/submitted, the quota is permanently consumed regardless of outcomes. Reserved quota is released if interview cancelled before completion.)
- How are notifications handled for companies with multiple admins? (CLARIFIED: Send to all users with admin flag, future enhancement: notification preferences)
- What happens when a company cancels their subscription? (Set status to "Cancelled", prorated refund issued for unused quota period, service continues until renewal date)
- How to handle date range filters that span multiple months for usage analytics? (Show aggregated data across all selected months with monthly breakdown markers in charts)
- What happens if a Company Admin tries to download a PDF invoice still being generated? (Disable download button until generation complete, show "Processing..." status)
- How are timezone differences handled for usage tracking and transaction timestamps? (Use company's configured timezone for all billing operations, display times in that timezone)
- How is remaining interview estimate calculated if usage varies significantly? (Use rolling 7-day average usage to estimate remaining quota depletion, update estimate daily)
- What happens if a company upgrades mid-year from Gold to Whale? (NEW: Pro-rate the difference. If Gold cost RM3600 for remaining 200 days and Whale annual is RM12000, charge prorated difference, grant new annual quota immediately)
- Can companies downgrade plans mid-year? (Allowed with prorated credit/charge. If downgrading reduces quota below current usage, existing data is preserved but new invitations are blocked until usage falls below new quota)
- How are individual interviews tracked separately from subscription quota? (NEW: Use separate fields in CompanySubscription: `subscription_interviews_remaining` and `individual_interviews_purchased`. Both are checked during authorization.)
- What happens if the system processes an interview completion after 24 hours? (Quota consumption happens immediately, retroactively if necessary. All charges must be recorded with completion timestamp for audit purposes)

## Requirements _(mandatory)_

### Functional Requirements

- **FR-000**: Require active subscription plan: Company MUST have an active subscription plan to send any interview invitations. If a company has no plan or plan is cancelled/suspended, system MUST block `Send Invitation` with message "No active subscription plan. Please subscribe to a plan to send invitations." with [View Plans] CTA.

- **FR-000a**: Interview quota and balance check before HR invitation: When HR triggers `Send Invitation`, the system MUST verify: (1) company has active subscription plan, (2) company has remaining interview quota (subscription quota + individual purchases > 0), AND (3) company has sufficient account balance to cover estimated interview cost. If any check fails, block the action with appropriate error message. Log the attempt for audit.

- **FR-001**: Display subscription tier and interview allocation: System MUST display subscription tier (Gold Fish/Dolphin/Whale), annual interview allocation, interviews used this year, remaining interviews, and subscription anniversary date on the subscription dashboard.

- **FR-002**: Display interview usage analytics: System MUST provide interview usage visualization via line graphs with toggleable daily/monthly views and date range filtering.

- **FR-003**: Display interview usage breakdown: System MUST provide interview usage breakdown by status (completed, pending, cancelled) in pie chart format.

- **FR-004**: Display billing history with invoices: System MUST maintain billing history with records for all subscription invoices, individual interview purchases, and payments, with columns: Invoice Number, Date, Description, Amount, Status, and Actions.

- **FR-005**: Generate downloadable PDF invoices: System MUST generate downloadable PDF invoices containing: company name, invoice number, date, itemized charges (subscription fee OR individual interview purchases), total amount, payment method, and payment status.

- **FR-006**: Support billing history filtering: System MUST support billing history filtering by date range and invoice type (subscription/individual_purchase).

- **FR-007**: Announce subscription tier options and pricing: System MUST clearly display all three subscription tiers: Gold Fish (RM3600/year, 300 interviews), Dolphin (RM7200/year, 800 interviews), Whale (RM12000/year, 2000 interviews) with per-interview rates (RM12, RM9, RM6 respectively).

- **FR-008**: Enable individual interview purchases: System MUST allow Company Admins to purchase individual interviews at RM15 each when plan quota is exhausted, with payment via bank transfer.

- **FR-009**: Track individual interview quota separately: System MUST maintain separate quota counters for subscription interviews and purchased individual interviews, displaying both on the dashboard.

- **FR-010**: Consume plan quota before individual interviews: When candidate completes an interview, system MUST consume plan quota first, then use individual purchased interviews only after plan quota is exhausted.

- **FR-011**: Send interview quota alerts at 80% and 100%: System MUST send email and in-app notifications when company reaches 80% interview quota usage (warning to upgrade) and 100% quota usage (urgent message to purchase or upgrade).

- **FR-012**: Prevent duplicate quota alerts: System MUST prevent sending duplicate notifications for the same threshold within the same subscription year. Flags reset on subscription anniversary date.

- **FR-013**: Support subscription upgrades with prorated charges: System MUST allow Company Admins to upgrade plans mid-year, applying pro-rated charges for the difference and immediately granting new annual quota based on upgrade date.

- **FR-014**: Support subscription downgrades with refunds: System MUST allow Company Admins to downgrade plans mid-year, issuing pro-rated refunds or charges and warning if current interview usage exceeds new plan quota (existing data read-only, new invitations blocked).

- **FR-015**: Allow subscription cancellation: System MUST allow Company Admins to cancel subscriptions, issuing pro-rated refund for unused period and allowing service continuation until renewal date.

- **FR-016**: Process bank transfer payments for individual purchases: System MUST provide bank transfer payment instructions for individual interview purchases, with unique reference code for tracking.

- **FR-017**: Allow back-office admins to credit individual interviews: System MUST allow Super Admin/Finance team to process bank transfer receipts and credit individual interviews to company accounts after payment verification.

- **FR-018**: Send payment confirmation emails: System MUST send email confirmation when individual interviews are purchased, subscription is upgraded, or bank transfer payment is processed, detailing company name, items purchased, cost, and new quota balance.

- **FR-019**: Calculate annual billing on subscription anniversary: System MUST calculate all subscription and quota usage based on anniversary date (subscription start date to same date next year), not calendar month alignment.

- **FR-020**: Use company timezone for billing operations: System MUST use company's configured timezone for all billing operations, dashboard displays, quota resets, and notification scheduling.

- **FR-021**: Make Job Description creation free: System MUST NOT charge any interview quota or credits for creating Job Descriptions. Job Description is a free operation for all companies.

- **FR-022**: Calculate estimated quota depletion date: System MUST display estimated interview quota depletion date based on rolling 7-day average interview completion rate, updated daily on dashboard.

- **FR-023**: Block quota overage gracefully: When company exceeds quota and attempts `Send Invitation`, system MUST block with clear message showing: current plan quota exhausted, remaining individual interviews (if any), upgrade options, and purchase CTA.

- **FR-024**: Audit all quota consumption: Every interview completion MUST create a `BillingTransaction` record with type `interview_completion`, including company_id, interview_id, interview_cost (RM), timestamp, and quota source (plan_quota or individual_purchase).

- **FR-025**: Support bulk individual interview purchase: System MUST allow Company Admins to purchase multiple individual interviews in a single transaction (e.g., buy 10 interviews at RM150 total).

- **FR-026**: Display quota breakdown in invoice: PDF invoices for subscription billing MUST show: subscription tier, annual allocation, remaining quota, and link to purchase individual interviews if needed.

- **FR-027**: Reserve interview quota when sending invitation: When HR sends an interview invitation, system MUST reserve one interview from available quota (plan quota first, then individual purchases). If unreserved quota is insufficient, block the invitation with a clear error.

- **FR-028**: Convert reserved quota on completion: When a candidate completes an interview, system MUST convert the reserved quota to consumed quota, record a `BillingTransaction` for the settlement, and tag the quota source (plan or individual purchase).

- **FR-029**: Release reserved quota on expiry/cancellation: When an invitation expires (3 days) or is cancelled before completion, system MUST release the reserved quota back to the available pool and mark the invitation as expired/cancelled.

- **FR-030**: Track account balance and quota reservations: System MUST maintain company account balance and track reserved_interviews_count to derive available quota: (subscription_interviews_remaining + individual_interviews_remaining) - reserved_interviews_count.

- **FR-031**: Top-up account balance via bank transfer: System MUST allow Company Admins to top up account balance via bank transfer, with back-office admin approval to credit the account.

- **FR-032**: Block invitations on insufficient balance: If company has quota remaining but insufficient available balance for estimated interview cost, system MUST block invitation with message "Insufficient balance. Please top up your account."

- **FR-033**: Quota exhaustion flow options: When company exhausts subscription plan quota (interviews_remaining = 0), system MUST display three options on dashboard and in send-invitation error message:
  1. **Upgrade Plan**: [Upgrade] button → allows switching to higher tier (e.g., Gold Fish to Dolphin) with prorated charges, immediately granting new annual quota
  2. **Purchase Individual Interviews**: [Purchase] button → buy interviews at RM15 each, deducted from balance, added to quota pool
  3. **Top Up Balance**: [Top Up] button → deposit funds to enable individual interview purchases if balance is insufficient

### Key Entities

- **SubscriptionPlan**: Defines the three tiered subscription plans with attributes:
  - name (Gold Fish, Dolphin, Whale)
  - annual_price_rmb (RM3600, RM7200, RM12000)
  - annual_interview_quota (300, 800, 2000)
  - cost_per_interview_rmb (RM12, RM9, RM6)
  - description (e.g., "Best for small teams")
  - features (JSON/array of included features)

- **CompanySubscription**: Links companies to their subscription with attributes:
  - id, company_id (FK), subscription_plan_id (FK)
  - status (active/suspended/cancelled)
  - subscription_start_date (the anniversary date for annual billing)
  - subscription_anniversary_date (same as start_date, used for annual renewal)
  - interviews_remaining_this_year (decremented each time interview completes)
  - individual_interviews_purchased (total individual interviews bought, separate from plan)
  - individual_interviews_remaining (decremented when individual interviews are consumed)
  - reserved_interviews_count (interviews held for pending invitations)
  - available_interviews_remaining (calculated: subscription_interviews_remaining + individual_interviews_remaining - reserved_interviews_count)
  - account_balance_rmb (total account balance in RM)
  - last_quota_reset_date (timestamp of last anniversary reset)
  - service_end_date (nullable, for cancelled subscriptions)
  - created_at, updated_at

- **InterviewCompletion**: Records each completed interview for quota consumption tracking with attributes:
  - id, company_id (FK), interview_id (FK), candidate_id (FK)
  - interview_completed_at (timestamp when interview was submitted)
  - quota_consumed_from (enum: plan_quota or individual_purchase)
  - cost_rmb (RM12, RM9, RM6, or RM15 depending on source)
  - billing_transaction_id (FK, reference to BillingTransaction for audit)
  - created_at

- **BillingTransaction**: Stores all financial transactions (annual subscription billing, individual purchases, balance top-ups, interview settlements) with attributes:
  - id, company_id (FK)
  - transaction_type (annual_subscription / individual_interview_purchase / balance_topup / interview_settlement)
  - amount_rmb (positive for charges, based on plan or individual purchases)
  - transaction_date (when the charge was processed)
  - description (e.g., "Annual subscription - Gold Fish Plan (300 interviews)", "Individual interview purchase (5 interviews × RM15)", "Interview settlement - Interview #123", "Balance top-up via bank transfer")
  - subscription_plan_id (FK, nullable - which plan this charge is for)
  - interview_id (FK, nullable - for interview-related transactions)
  - interview_count_charged (number of interviews in this transaction)
  - payment_method (bank_transfer, etc.)
  - payment_receipt_url (for bank transfers)
  - processed_by_user_id (FK, for back-office processing)
  - status (pending/completed/failed)
  - invoice_url (link to generated PDF invoice)
  - created_at, updated_at

- **QuotaReservation**: Tracks reserved interview quota for pending invitations with attributes:
  - id, company_id (FK), interview_id (FK)
  - reserved_interview_count (typically 1 per invitation)
  - reserved_from (plan_quota or individual_purchase)
  - estimated_cost_rmb (optional, for cost forecasting only)
  - reserved_at (timestamp when reservation was created)
  - expires_at (timestamp when reservation expires - 3 days from creation)
  - status (active/consumed/released/cancelled)
  - consumed_at (timestamp when reservation was converted to consumption)
  - released_at (timestamp when reservation was released without consumption)
  - billing_transaction_id (FK, reference to settlement transaction)
  - created_at, updated_at

- **PaymentMethod**: Stores payment method references with attributes:
  - id, company_id (FK)
  - type (bank_transfer, credit_card, etc.)
  - is_active (boolean)
  - payment_details (bank account info, card last 4, etc. - encrypted)
  - created_at, updated_at

- **NotificationLog**: Tracks sent notifications to prevent duplicates with attributes:
  - id, company_id (FK)
  - notification_type (email/in_app_banner/in_app_modal)
  - trigger_event (quota_80_percent / quota_exhausted / purchase_confirmation / subscription_anniversary)
  - sent_at
  - acknowledged_at (for modals)
  - subscription_year (to track which year's quota this notification is for)
  - created_at

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Company Admins can view subscription tier, interview quota (plan vs individual), reserved quota (pending invitations), and account balance in under 2 clicks from the main dashboard, with page load time under 2 seconds
- **SC-002**: Interview quota calculations are 100% accurate and match the formula: interviews_used from CompanySubscription.interviews_remaining_this_year
- **SC-003**: Interview quota consumption is instantaneous when candidate completes interview, reflected on dashboard within 5 seconds
- **SC-004**: Quota reservation occurs within 1 second of invitation send, with estimated cost accuracy ±10%
- **SC-005**: Reservation conversion on interview completion occurs within 30 seconds, consuming reserved quota and settling actual cost against account balance
- **SC-006**: Expired invitation quota release occurs within 30 minutes of expiry (3-day timeout)
- **SC-007**: Email notifications for 80% and 100% quota alerts are delivered within 5 minutes of threshold being crossed
- **SC-008**: Email notifications for low balance (< RM50) are delivered within 5 minutes of dropping below threshold
- **SC-009**: In-app notifications appear immediately upon user login after quota/balance threshold is crossed (no delay beyond page load time)
- **SC-010**: Duplicate notifications for the same threshold within the same subscription year occur in 0% of cases
- **SC-011**: PDF invoices for subscriptions and individual purchases are generated within 10 seconds of request, with 100% accuracy in data (no missing/incorrect amounts or interview counts)
- **SC-012**: Billing history page loads and displays up to 3 years of transactions (including all quota reservations, settlements, releases) in under 3 seconds, with pagination for older records
- **SC-013**: Date range filtering on usage charts returns results in under 2 seconds for up to 3 years of data
- **SC-014**: Interview quota check is enforced within 1 second when `Send Invitation` is triggered, blocking if quota is exhausted (100% enforcement)
- **SC-015**: Account balance check is enforced within 1 second when `Send Invitation` is triggered, blocking if available balance insufficient (100% enforcement)
- **SC-016**: Manual individual interview purchase processing (bank transfer workflow) is completed within 1 business day from receipt submission to quota credit AND balance deduction
- **SC-017**: Subscription upgrade/downgrade with prorated charges is calculated accurately and credited/charged within 1 hour of approval
- **SC-018**: Estimated quota depletion date calculation is updated daily and uses rolling 7-day average accuracy (±1 day margin acceptable)
- **SC-019**: Chart visualizations (line graphs, pie charts) render correctly across all major browsers (Chrome, Firefox, Safari, Edge) with no data display errors
- **SC-020**: Quota reservation audit trail provides 100% traceability from reservation → settlement/release with no orphaned holds

---

## Clarifications & Open Questions

### Answered Clarifications

0. **Mandatory Subscription Plan**: ✅ **CLARIFIED** - Company MUST have an active subscription plan to send interview invitations. No interviews can be sent without an active plan. Cancelled or suspended plans block all interview operations until plan is reactive or new plan is selected.

1. **Interview Quota Consumption**: ✅ **CLARIFIED** - Interview quota is ONLY consumed when a candidate completes and submits an interview. Sending invitations reserves quota (so it cannot be double-booked) but does NOT consume it. Expired or cancelled invitations do NOT consume quota. Once consumed, quota is not refundable regardless of interview outcome.

2. **Individual Interview Purchases**: ✅ **CLARIFIED** - Individual interviews are purchased via bank transfer at RM15 each and are tracked separately from plan quota. System consumes plan quota first, then individual purchases when plan quota is exhausted. Requires active plan + sufficient balance.

3. **Payment Method**: ✅ **CLARIFIED** - Initial payment method is bank transfer (manual processing by finance team). Credit card integration (Stripe/PayPal) to be added in future phase based on business needs.

4. **Billing Cycle**: ✅ **CLARIFIED** - Billing cycles are anniversary-based (subscription start date to same date next year), not calendar month aligned. Subscription reset happens on subscription anniversary date each year.

5. **Job Description Pricing**: ✅ **CLARIFIED** - Job Description creation is completely FREE for all companies regardless of subscription tier. No quota consumption, no charges.

6. **Prorating on Upgrade/Downgrade**: ✅ **CLARIFIED** - Subscription changes are pro-rated based on days remaining in current year. Formula: (annual_price / 365) × days_remaining = prorated_amount. Difference is charged (upgrade) or credited (downgrade) immediately.

7. **Billing Notifications**: ✅ **CLARIFIED** - All billing notifications (subscription confirmations, quota alerts, purchase receipts) are sent ONLY to users with admin flag, not all company users.

8. **Bulk Individual Purchases**: ✅ **CLARIFIED** - Companies can purchase multiple individual interviews in a single transaction (e.g., 5 interviews = RM75). Single invoice generated with total cost.

9. **Subscription Cancellation**: ✅ **CLARIFIED** - Companies can cancel at any time. Pro-rated refund issued for unused period. Service continues until renewal date. After renewal date, all interview operations blocked.

10. **Timezone Handling**: ✅ **CLARIFIED** - All billing operations use company's configured timezone: quota resets on subscription anniversary at midnight in company timezone, notifications scheduled in company timezone.

11. **Billing Notifications**: ✅ **CLARIFIED** - Only users with admin flag (Company Admins) will receive billing notifications and invoices. Regular users will not receive billing emails.

12. **Billing Cycle Start Date**: ✅ **CLARIFIED** - Billing cycles are anniversary-based (subscription date to same date next year), NOT calendar month aligned. Example: Subscribe on Dec 13 → quota resets every Dec 13 annually.

13. **Interview Quota Exhaustion**: ✅ **CLARIFIED** - When a company exhausts their plan quota, system blocks `Send Invitation` and displays three options: (1) Upgrade plan to higher tier, (2) Purchase individual interviews at RM15 each (uses balance), (3) Top up balance to enable purchases.

14. **Prorating Calculation**: ✅ **CLARIFIED** - Pro-ration uses formula: (annual_plan_price / 365 days) × remaining_days_in_year = prorated_amount. Applied to upgrades, downgrades, and cancellations.

15. **Interview Cancellation Refunds**: ✅ **CLARIFIED** - If a candidate cancels or doesn't complete an interview, no quota is consumed (quota only consumed on completion). Once interview is completed/submitted, quota consumption is permanent (non-refundable).

16. **Multi-Quota Consumption**: ✅ **CLARIFIED** - System maintains two separate interview quotas per company: (1) subscription_interviews_remaining (from plan), (2) individual_interviews_remaining (from purchases). Subscriptions quota is consumed first, individual purchases only after plan quota reaches zero.

17. **Billing Model - Hybrid Quota + Balance**: ✅ **CLARIFIED - OPTION A HYBRID** - System uses HYBRID quota + balance model:
    - Subscription plans provide QUOTA (300/800/2000 interviews/year)
    - All companies maintain an ACCOUNT BALANCE (in RM) for pay-as-you-go costs
    - Individual interview purchases at RM15 ADD to quota counter (not just balance deduction)
    - Sending invitation: System checks (1) quota available, (2) balance sufficient for estimated cost, then RESERVES QUOTA (plan first, then individual purchases)
    - Interview completion: System CONSUMES reserved quota + SETTLES actual cost against account balance (no balance reservation)
    - Invitation expiry (3 days): RELEASES reserved quota, quota unchanged
    - Quota exhausted: HR can still send invitations if balance sufficient + purchase individual interviews (RM15 adds quota)
    - Balance insufficient: System blocks invitation regardless of quota availability

### Remaining Open Questions

11. **Revenue Reporting**: How will revenue be tracked for analytics? Should individual interview purchases be reported separately from subscription MRR in God Mode dashboard? [NEEDS CLARIFICATION]

12. **Audit Trail Depth**: Should all quota changes (resets, consumption, adjustments) be logged in a separate audit table for compliance purposes, or is BillingTransaction table sufficient? [NEEDS CLARIFICATION]

13. **Top-Up Mechanism**: How do users initiate top-ups? Is it manual (user clicks "Add Funds" and makes payment), automatic (auto-recharge when balance hits threshold), or both options available? [NEEDS CLARIFICATION]

14. **Minimum Top-Up Amount**: Is there a minimum top-up amount (e.g., minimum $50 deposit)? Maximum top-up amount? [NEEDS CLARIFICATION]

15. **Unused Balance Handling**: If a user cancels their subscription, what happens to unused prepaid balance? (Refunded, rolled over to next billing cycle, forfeited?) [NEEDS CLARIFICATION]

16. **Token Pricing Model**: How is token pricing calculated for the single pay-as-you-go plan? Fixed flat rate per million tokens regardless of volume, or tiered volume-based pricing (e.g., first 10M at $25/M, next 10M at $20/M, etc.)? [NEEDS CLARIFICATION]

---

## Integration Points

### Dependencies on Other Features

- **Feature 002 (SaaS Admin Portal)**: Requires Company entity and Company Admin role for subscription ownership and management. God Mode may be needed for debugging subscription issues.

- **Feature 004 (User Authorization)**: Requires permission checks to ensure only Company Admins (with `subscription.view`, `subscription.manage` permissions) can access subscription management features. Regular users should have read-only access to usage analytics only if granted.

- **Feature 001 (AI-HR Interview System)**: Token consumption originates from interview operations (question generation, response scoring). TokenUsage records must be created by Feature 001 and linked to interview activities.

### External Systems

- **Payment Gateway**: Future integration with Stripe/PayPal/Square for credit card processing (temporary workaround: manual bank transfer)

- **Email Service**: Required for sending threshold notifications, invoice delivery, payment confirmations (e.g., SendGrid, AWS SES)

- **PDF Generation Library**: Required for invoice PDF generation (e.g., wkhtmltopdf, Puppeteer, PDFKit)

### Data Consistency Requirements

- **Token Usage Accuracy**: All token consumption MUST be recorded in token_usages table immediately after usage occurs, with no data loss. This is critical for billing accuracy.

- **Real-Time Credit Calculation**: Remaining credits displayed on dashboard must reflect the most recent token_usages records (acceptable delay: under 5 seconds for eventual consistency)

- **Billing Cycle Integrity**: Billing cycle dates (start/end) must be immutable once set. Subscription changes create new billing records but do not alter historical cycle dates.

- **Idempotent Notifications**: Notification delivery system must be idempotent to prevent duplicate notifications due to retry logic or race conditions.

---

## Technical Notes _(optional implementation guidance)_

### Suggested Dashboard Layout

```
+--------------------------------------------------+
| [Company Logo]                        [User Menu] |
+--------------------------------------------------+
| Subscription Overview                             |
| Plan: Plan A (Monthly) | Status: Active           |
| Renewal: Jan 15, 2026  | Price: $500/month        |
| Remaining Credits: 5,000,000 / 20,000,000 (25%)  |
| [Progress Bar ===================>          ]     |
| Overage: None | Extra Charges: $0.00             |
+--------------------------------------------------+
| Token Usage Analytics                             |
| [Daily / Monthly Toggle] [Date Range Picker]     |
| [Line Graph: Usage Over Time                  ]   |
|                                                   |
| [Pie Chart: Usage by Purpose]  [Dual-Axis Chart] |
+--------------------------------------------------+
| Billing & Invoices                                |
| [Filter: Date Range] [Filter: Status]            |
| | Invoice # | Date       | Amount | Status | PDF ||
| | INV-001   | Dec 1 2025 | $500   | Paid   | [⬇] ||
| | INV-002   | Nov 1 2025 | $500   | Paid   | [⬇] ||
+--------------------------------------------------+
| Subscription Management                           |
| [Upgrade Plan] [Change Billing Cycle] [Cancel]   |
+--------------------------------------------------+
```

### Chart Implementation Recommendations

- **Line Graph**: Use Chart.js or Recharts with time-series data, support zoom/pan for large date ranges
- **Pie Chart**: Limit to top 5 categories, group others as "Other" if more than 5 purposes exist
- **Dual-Axis Chart**: Combine line (usage over time) on primary Y-axis, bar chart (usage by purpose) on secondary Y-axis, with shared X-axis (time periods)

### Performance Optimization

- **Caching**: Cache aggregated usage statistics per company per billing cycle, invalidate on new token_usages insert
- **Pagination**: Implement cursor-based pagination for billing history (load 12 months initially, lazy-load older records)
- **Database Indexes**: Index token_usages table on (company_id, timestamp) and (company_id, purpose, timestamp) for fast aggregation queries
- **Background Jobs**: Run threshold notification checks via scheduled job (every 15 minutes) rather than real-time on every token usage event to reduce load

### Security Considerations

- **Invoice Access Control**: Ensure users can only download invoices for their own company (validate company_id in JWT/session)
- **Payment Method Encryption**: Encrypt sensitive payment method data (card details) at rest using AES-256
- **Audit Logging**: Log all subscription changes, payment method changes, and manual back-office invoice updates for compliance
- **Rate Limiting**: Apply rate limits to PDF download endpoints to prevent abuse (max 10 downloads per minute per user)
