# Feature Specification: Back-Office Subscription & Payment Management

**Feature Branch**: `011-back-office-subscription-management`  
**Created**: February 13, 2026  
**Status**: Draft  
**Input**: User description: "Back-office admin portal for managing company subscriptions, processing bank transfer payments, crediting accounts, viewing usage analytics, handling refunds, and monitoring system-wide billing operations."

## User Scenarios & Testing _(mandatory)_

### User Story 1 - View All Companies with Subscription Status (Priority: P1)

Back-office admins need a centralized dashboard to view all companies, their subscription tier, quota usage, account balance, and payment status to monitor system health and identify companies needing attention.

**Why this priority**: Core visibility into all companies' subscription and financial status is fundamental for support, billing operations, and business intelligence. Without this, admins cannot identify at-risk companies or process payment inquiries efficiently.

**Independent Test**: Can be fully tested by creating sample companies with different subscription tiers, usage levels, and balance statuses, then verifying that the admin dashboard displays accurate data with filtering by status, tier, quota usage, and balance. Delivers immediate value by providing operational transparency.

**Acceptance Scenarios**:

1. **Given** I am logged in as a Back-Office Admin, **When** I navigate to the Companies Dashboard, **Then** I see a table with columns: Company Name, Subscription Tier, Status (Active/Suspended/Cancelled), Annual Quota, Interviews Used, Remaining, Reserved, Available Balance, Last Payment Date, Anniversary Date, and Actions (View Details/Manage)

2. **Given** there are 50 companies in the system with various subscription statuses, **When** I filter by status "Active" and tier "Gold Fish", **Then** I see only active Gold Fish subscribers sorted by remaining quota (ascending) to identify companies approaching exhaustion

3. **Given** Company XYZ has: Gold Fish subscription (300 quota), 285 interviews used (95% consumed), RM50 balance (low), 5 reserved invitations, last payment Nov 1, 2025, **When** I view Company XYZ in the dashboard, **Then** I see all these details with warning indicators for "High Usage (95%)" and "Low Balance (RM50)" highlighted in amber

4. **Given** I am viewing the Companies Dashboard, **When** I apply filter "Balance < RM100", **Then** I see all companies with available balance below RM100 sorted by balance (ascending) to prioritize low-balance interventions

5. **Given** there are 200 companies in the system, **When** I search by company name "Acme Corp", **Then** I see only results matching "Acme" with all subscription details displayed

6. **Given** I need to export company data for financial reporting, **When** I click "Export to CSV", **Then** a CSV file is downloaded containing all visible columns for the filtered companies with accurate data as of export time

---

### User Story 2 - Process Bank Transfer Payments & Credit Accounts (Priority: P1)

Back-office admins need to manually process bank transfer receipts for subscription payments, balance top-ups, and individual interview purchases by crediting company accounts and updating quotas.

**Why this priority**: Manual payment processing is the MVP payment method (per Feature 005's FR-016/FR-017). Without this capability, no payments can be processed and revenue cannot be recognized, making this a critical P1 function.

**Independent Test**: Can be fully tested by simulating a company submitting a bank transfer receipt via email, then using the admin portal to verify payment, credit the account, update balance/quota, and trigger confirmation email. Delivers immediate revenue recognition capability.

**Acceptance Scenarios**:

1. **Given** Company ABC submitted a bank transfer receipt for RM500 balance top-up on Feb 10, 2026, **When** I navigate to Pending Payments queue, **Then** I see an entry for "Company ABC - Balance Top-up - RM500 - Submitted Feb 10, 2026" with [View Receipt] [Approve] [Reject] actions

2. **Given** I select a pending payment for Company ABC (RM500 top-up), **When** I click [View Receipt], **Then** a modal opens displaying the uploaded receipt image/PDF with transaction reference, amount, date, and bank account details

3. **Given** I have verified the bank transfer receipt is valid for RM500, **When** I click [Approve] and confirm, **Then** the system:
   - Credits Company ABC's account balance by RM500 (e.g., from RM200 to RM700)
   - Creates a BillingTransaction record (type: balance_topup, amount: RM500, status: completed)
   - Sends email confirmation to all Company ABC admins: "Balance top-up of RM500 processed. New balance: RM700"
   - Moves payment from Pending to Completed queue with processor name and timestamp

4. **Given** Company DEF submitted a bank transfer receipt for 20 individual interviews (20 × RM15 = RM300) with current balance RM1000, **When** I approve the payment, **Then** the system:
   - Deducts RM300 from Company DEF's balance (RM1000 → RM700)
   - Adds 20 to `individual_interviews_remaining` field in CompanySubscription
   - Creates BillingTransaction (type: individual_purchase_debit, amount: RM300, interview_count: 20)
   - Sends email: "20 individual interviews purchased for RM300. Remaining balance: RM700. Individual quota: 20 interviews."

5. **Given** I select a pending payment with an invalid or unclear receipt, **When** I click [Reject] and enter reason "Receipt amount does not match claimed amount", **Then** the system sends email to company admins with rejection reason and instructions to resubmit, and moves payment to Rejected queue

6. **Given** Company GHI submitted a subscription payment for Gold Fish upgrade (RM3600) but has insufficient receipt verification, **When** I mark the payment as "Under Review" with note "Awaiting bank confirmation", **Then** the payment remains in Pending queue with status "Under Review" and note visible to all admins

7. **Given** I need to manually credit a refund of RM1200 to Company JKL for downgrade, **When** I navigate to Company JKL details → [Manual Transaction] → select type "Refund/Credit", enter amount RM1200, and note "Prorated refund for Whale → Gold Fish downgrade", **Then** the system adds RM1200 to balance, creates BillingTransaction (type: refund, amount: RM1200), and sends confirmation email

---

### User Story 3 - View & Manage Company Subscription Details (Priority: P1)

Back-office admins need detailed view of individual company subscription history, quota consumption timeline, balance transactions, and ability to perform administrative actions like manual adjustments or plan changes.

**Why this priority**: Detailed company view is essential for handling support requests, investigating billing discrepancies, and performing manual adjustments. Admins cannot effectively troubleshoot without granular visibility into company data.

**Independent Test**: Can be fully tested by selecting a company from the dashboard, viewing their complete subscription timeline, transaction history, quota usage breakdowns, and performing manual adjustment actions (add quota, adjust balance, change status). Delivers value for support operations.

**Acceptance Scenarios**:

1. **Given** I click "View Details" for Company XYZ from the Companies Dashboard, **When** the Company Details page loads, **Then** I see sections: Subscription Overview, Quota Status, Balance & Transactions, Usage Analytics, Interview History, and Administrative Actions

2. **Given** I am viewing Company XYZ Details → Subscription Overview section, **When** the page renders, **Then** I see: current tier (Gold Fish), status (Active), subscription start date (Dec 1, 2025), anniversary date (Dec 1, 2026), annual allocation (300), plan cost (RM3600/year), and [Change Plan] [Suspend] [Cancel] action buttons

3. **Given** I am viewing Company XYZ Details → Quota Status section, **When** the page renders, **Then** I see:
   - Plan Quota: 285/300 used (95%), 15 remaining
   - Individual Interviews: 10 purchased, 7 remaining
   - Reserved Quota: 5 (pending invitations with expiry dates)
   - Available Quota: 17 (15 plan + 7 individual - 5 reserved)
   - Visual quota breakdown chart (stacked bar: used, reserved, available)

4. **Given** I am viewing Company XYZ Details → Balance & Transactions section, **When** the page renders, **Then** I see:
   - Current Balance: RM450
   - Recent Transactions table (last 20) with: Date, Type, Amount, Description, Invoice, Processor
   - [View All Transactions] button to expand full history
   - [Manual Adjustment] button for admin actions

5. **Given** I click [Manual Adjustment] in Balance & Transactions section, **When** the adjustment modal opens, **Then** I can select adjustment type (Add Balance / Deduct Balance / Add Quota / Deduct Quota), enter amount/count, add admin note, and preview the result before confirming

6. **Given** I perform a manual adjustment: "Add 50 individual interviews" with note "Goodwill credit for system downtime Dec 5-6", **When** I confirm, **Then** the system:
   - Adds 50 to `individual_interviews_remaining`
   - Creates BillingTransaction (type: manual_credit, interview_count: 50, description: admin note, processed_by: admin_user_id)
   - Logs the action in audit trail
   - Sends email to company: "50 individual interviews credited to your account. Reason: Goodwill credit for system downtime Dec 5-6."

7. **Given** I am viewing Company XYZ Details → Usage Analytics section, **When** the page renders, **Then** I see:
   - Line graph of daily interview completions for last 90 days
   - Pie chart of quota consumption by source (plan vs individual)
   - Average interviews per week (rolling 4-week average)
   - Estimated depletion date based on current consumption rate

8. **Given** I am viewing Company XYZ Details → Interview History section, **When** I scroll to this section, **Then** I see a paginated table of all interviews with: Interview ID, Candidate Name, Job Title, Status (Completed/Pending/Cancelled), Date, Quota Source (Plan/Individual), Cost, and [View Details] link

---

### User Story 4 - View System-Wide Analytics & Revenue Reports (Priority: P1)

Back-office admins need aggregate analytics across all companies to track revenue, subscription distribution, quota consumption trends, and identify business insights for decision-making.

**Why this priority**: Business intelligence and revenue tracking are critical for financial planning and identifying growth opportunities. Without system-wide analytics, management cannot make data-driven decisions about pricing, capacity, or product strategy.

**Independent Test**: Can be fully tested by generating sample data across multiple companies with various subscription tiers and usage patterns, then verifying that the Analytics Dashboard displays accurate aggregate metrics, charts, and revenue totals. Delivers immediate business value.

**Acceptance Scenarios**:

1. **Given** I am logged in as Back-Office Admin, **When** I navigate to Analytics Dashboard, **Then** I see key metrics tiles: Total Active Subscriptions, Total MRR/ARR, Total Interviews Conducted (This Month), Total Account Balance Across All Companies, Average Quota Utilization Rate (%), Pending Payments Count

2. **Given** the Analytics Dashboard is loaded with data as of Feb 13, 2026, **When** I view the key metrics, **Then** I see:
   - Total Active Subscriptions: 47 companies (22 Gold Fish, 18 Dolphin, 7 Whale)
   - Total ARR (Annual Recurring Revenue): RM327,600
   - Total MRR (Monthly Recurring Revenue): RM27,300
   - Interviews Conducted This Month: 1,245 interviews
   - Total Account Balance: RM87,450 across all companies
   - Average Quota Utilization: 68.4% (system-wide)

3. **Given** I am viewing Analytics Dashboard → Subscription Distribution section, **When** the page renders, **Then** I see:
   - Pie chart showing subscription tier breakdown (Gold Fish: 47%, Dolphin: 38%, Whale: 15%)
   - Bar chart showing subscription growth over last 12 months (new subscriptions, cancellations, net change per month)

4. **Given** I am viewing Analytics Dashboard → Revenue section, **When** I select date range "Jan 2026 - Feb 2026", **Then** I see:
   - Line graph of daily revenue (subscription renewals, individual purchases, balance top-ups)
   - Revenue breakdown by source: Subscription Renewals (RM120,000), Individual Purchases (RM18,500), Balance Top-ups (RM45,200)
   - Total Revenue for period: RM183,700

5. **Given** I am viewing Analytics Dashboard → Quota Consumption Trends section, **When** the page renders, **Then** I see:
   - Line graph showing system-wide daily interview completions over last 90 days
   - Quota utilization heatmap by subscription tier (Gold Fish avg 71%, Dolphin avg 68%, Whale avg 62%)
   - Companies approaching exhaustion alert: "5 companies at >90% quota usage" with [View List] link

6. **Given** I am viewing Analytics Dashboard → Payment Operations section, **When** the page renders, **Then** I see:
   - Pending Payments Count: 12 (RM23,400 total)
   - Average Payment Processing Time: 1.3 business days
   - Failed/Rejected Payments This Month: 3 (RM4,200)
   - [View Pending Queue] button

7. **Given** I need to export revenue report for accounting, **When** I click [Export Revenue Report] and select date range "Jan 2026 - Dec 2026" and format "Excel", **Then** an Excel file is generated with sheets: Summary (total revenue by month), Subscription Renewals, Individual Purchases, Balance Top-ups, Refunds, with company details and transaction IDs for reconciliation

---

### User Story 5 - Manage Subscription Lifecycle (Upgrades, Downgrades, Suspensions, Cancellations) (Priority: P2)

Back-office admins need ability to manually process subscription plan changes, calculate prorated charges/refunds, suspend accounts for non-payment, and handle cancellation requests with proper quota and billing adjustments.

**Why this priority**: While companies can self-serve upgrades/downgrades in Feature 005, manual admin intervention is needed for special cases (billing disputes, goodwill gestures, suspended accounts). This enhances flexibility but is not MVP-critical since companies have self-service options.

**Independent Test**: Can be fully tested by selecting a company, initiating plan change action (upgrade/downgrade), verifying prorated calculation preview, confirming change, and validating that quota, billing records, and email notifications are accurate. Delivers value for complex billing scenarios.

**Acceptance Scenarios**:

1. **Given** I am viewing Company ABC (Gold Fish, subscribed Dec 1, 2025, 150/300 quota used) on Feb 13, 2026, **When** I click [Change Plan] → select "Upgrade to Whale", **Then** I see prorated calculation preview:
   - Remaining days: 292 days (until Dec 1, 2026)
   - Prorated Gold Fish credit: RM2,880 (RM3600 × 292/365)
   - Prorated Whale cost: RM9,600 (RM12,000 × 292/365)
   - Net charge: RM6,720 (RM9,600 - RM2,880)
   - New annual quota: 2,000 interviews (granted immediately)
   - [Confirm Upgrade] button

2. **Given** I confirm the upgrade from Gold Fish to Whale for Company ABC, **When** the system processes the change, **Then**:
   - CompanySubscription.subscription_plan_id updated to Whale
   - CompanySubscription.interviews_remaining_this_year set to 2,000 (new quota)
   - BillingTransaction created (type: subscription_upgrade, amount: RM6,720, description: "Prorated upgrade: Gold Fish → Whale")
   - Email sent to Company ABC: "Subscription upgraded to Whale. New quota: 2,000 interviews. Prorated charge: RM6,720. Anniversary date: Dec 1, 2026."
   - Invoice PDF generated with prorated calculation breakdown

3. **Given** I am viewing Company DEF (Whale, subscribed Jan 1, 2026, 1800/2000 quota used) on Feb 13, 2026, **When** I click [Change Plan] → select "Downgrade to Dolphin", **Then** I see warning: "⚠️ Current usage (1,800 interviews) exceeds Dolphin annual quota (800 interviews). Existing data will be preserved but new invitations will be blocked until anniversary reset." and prorated refund calculation: RM7,800 credit to account balance

4. **Given** I confirm the downgrade from Whale to Dolphin for Company DEF (usage > new quota), **When** the system processes the change, **Then**:
   - CompanySubscription.subscription_plan_id updated to Dolphin
   - CompanySubscription.interviews_remaining_this_year remains 200 (2000 - 1800 used)
   - System flags account with `overage_mode: true` to block new invitations
   - BillingTransaction created (type: refund, amount: RM7,800, credited to balance)
   - Email sent with warning about blocked invitations until next anniversary (Jan 1, 2027)

5. **Given** Company GHI has failed to pay subscription renewal for 30 days, **When** I click [Suspend Account], **Then** a modal opens asking for suspension reason ("Non-payment - 30 days overdue"), and preview shows: status changes to "Suspended", all pending invitations cancelled and quota released, new invitations blocked, service end date set to today

6. **Given** I confirm suspension of Company GHI for non-payment, **When** the system processes suspension, **Then**:
   - CompanySubscription.status updated to "Suspended"
   - All QuotaReservation records with status "active" are cancelled, reserved quota released
   - Email sent to Company GHI: "Your account has been suspended due to non-payment. Please contact billing@company.com to reactivate. Outstanding amount: RM7,200."
   - Dashboard access retained (read-only) but all interview operations blocked

7. **Given** Company JKL requests cancellation on Feb 13, 2026 (subscribed Nov 1, 2025 to Gold Fish, 120/300 quota used), **When** I click [Cancel Subscription] and confirm, **Then** I see refund calculation:
   - Remaining days: 262 days (until Nov 1, 2026)
   - Unused quota: 180 interviews (300 - 120)
   - Prorated refund: RM2,584 (RM3600 × 262/365)
   - Service continues until: Nov 1, 2026 (current anniversary date)
   - [Confirm Cancellation & Issue Refund] button

8. **Given** I confirm cancellation for Company JKL, **When** the system processes cancellation, **Then**:
   - CompanySubscription.status updated to "Cancelled"
   - CompanySubscription.service_end_date set to Nov 1, 2026
   - BillingTransaction created (type: refund, amount: RM2,584, credited to balance or issued via bank transfer)
   - Email sent: "Subscription cancelled. Service continues until Nov 1, 2026. Prorated refund: RM2,584 credited to account balance."
   - Account retains access until service_end_date, then becomes read-only

---

### User Story 6 - View & Search Billing Transactions Across All Companies (Priority: P2)

Back-office admins need a global billing transactions view to search, filter, and export transaction history across all companies for accounting reconciliation, audit purposes, and financial reporting.

**Why this priority**: Global transaction search enables financial reconciliation and audit compliance. While important for operational efficiency, individual company transaction views (User Story 3) provide sufficient MVP capability, making this P2.

**Independent Test**: Can be fully tested by creating transactions across multiple companies with various types and dates, then using the global search interface to filter by company, type, date range, amount, and verifying export functionality. Delivers value for financial operations.

**Acceptance Scenarios**:

1. **Given** I am logged in as Back-Office Admin, **When** I navigate to Global Billing Transactions page, **Then** I see a table with columns: Transaction ID, Date, Company Name, Type, Amount, Description, Status, Invoice, Processor, and [View Details] action

2. **Given** there are 3,000+ transactions in the system, **When** I filter by date range "Jan 1, 2026 - Feb 13, 2026", transaction type "Individual Purchase", and status "Completed", **Then** I see only individual interview purchase transactions completed in that period, paginated with 50 per page

3. **Given** I need to find all transactions for Company XYZ, **When** I search by company name "XYZ", **Then** I see all billing transactions (subscriptions, individual purchases, top-ups, refunds, settlements) for Company XYZ sorted by date descending

4. **Given** I am viewing a specific transaction (e.g., Transaction ID: TXN-2026-001234), **When** I click [View Details], **Then** a modal opens showing full transaction details: company, type, amount, date, description, invoice URL, payment receipt URL, processed by (admin), related interview IDs, quota changes (before/after), balance changes (before/after), and audit trail

5. **Given** I need to export transactions for accounting reconciliation, **When** I apply filters (date range, type, status) and click [Export to Excel], **Then** an Excel file is generated with all filtered transactions including: Transaction ID, Date, Company, Type, Amount, Description, Invoice Number, Processor, Status, and subtotals by transaction type

6. **Given** I need to find all refund transactions issued this month, **When** I filter by type "Refund" and date range "Feb 1-13, 2026", **Then** I see all refunds with total refunds issued: RM15,400 across 8 transactions

---

### User Story 7 - Monitor & Manage Quota Reservations (Priority: P3)

Back-office admins need visibility into active quota reservations system-wide to identify stale reservations, manually release stuck reservations, and monitor reservation-to-completion conversion rates for system health.

**Why this priority**: Quota reservation monitoring is valuable for troubleshooting edge cases but not critical for day-to-day operations. Automated expiry handling (Feature 005 FR-029) covers normal scenarios, making manual intervention a P3 capability.

**Independent Test**: Can be fully tested by creating sample active, expired, and stuck reservations across companies, then using the admin interface to view reservation status, filter by company/age, and manually release reservations. Delivers value for edge case handling.

**Acceptance Scenarios**:

1. **Given** I am logged in as Back-Office Admin, **When** I navigate to Quota Reservations Monitor, **Then** I see a table with: Company Name, Interview ID, Reserved Count, Reserved From (Plan/Individual), Estimated Cost, Reserved At, Expires At, Status (Active/Expired/Consumed/Released), and [Release] action for Active reservations

2. **Given** there are 150 active reservations system-wide, **When** the page loads, **Then** I see reservations sorted by expiry date (ascending) to prioritize those expiring soon, with aging indicators (Red: expires <6 hours, Amber: expires <24 hours, Green: >24 hours)

3. **Given** I filter reservations by status "Expired" and age ">3 days", **When** the results load, **Then** I see expired reservations that have not been automatically released (potential system issue), with [Force Release] action to manually recover quota

4. **Given** I select an expired reservation (Company ABC, Interview #456, expired 5 days ago, status still "Active"), **When** I click [Force Release] and confirm, **Then** the system:
   - Updates QuotaReservation status to "Released"
   - Decrements reserved_interviews_count in CompanySubscription
   - Logs manual release action in audit trail with admin user ID
   - Shows success message: "Reservation released. Company ABC quota released: 1 interview."

5. **Given** I am viewing Quota Reservations Monitor → Summary section, **When** the page renders, **Then** I see aggregate metrics:
   - Total Active Reservations: 150
   - Total Reserved Quota: 150 interviews (RM12,450 estimated value)
   - Average Reservation Age: 1.2 days
   - Conversion Rate (Last 30 days): 72% (reservations → completed interviews)
   - Expiry Rate: 18% (reservations expired without completion)
   - Cancellation Rate: 10% (manually cancelled before expiry)

6. **Given** I need to export reservation data for analysis, **When** I click [Export Reservations Report] with date range "Jan 2026 - Feb 2026", **Then** an Excel file is generated with all reservations in that period showing: company, interview ID, reservation lifecycle (created → consumed/expired/released), duration, and outcome

---

### Edge Cases

- What happens if admin approves a payment that was already refunded/cancelled by the company? (System checks transaction status before processing. If already cancelled, show warning and prevent duplicate credit. Log attempt in audit trail.)
- How to handle admin processing a bank transfer with wrong amount (company claims RM500, receipt shows RM300)? (Admin should reject with reason "Amount mismatch". Company must resubmit with correct details.)
- What happens if admin manually adds 100 quota to a company that is already at quota capacity? (System allows admin override. New quota is added to `individual_interviews_remaining` field. Display warning: "Company is already at 300/300. Adding 100 will result in 400 total available." Log as manual credit in BillingTransaction.)
- How to handle concurrent admin actions (two admins processing same payment)? (Implement optimistic locking. Second admin gets error: "Payment already processed by [Admin Name] at [timestamp]". Show reload button.)
- What happens if admin downgrades a plan but company has pending reservations that exceed new quota? (System shows warning: "5 pending reservations will be cancelled during downgrade." Admin must confirm. On confirmation, cancel all reservations, release quota, notify company.)
- How are refunds handled for bank transfer payments? (Admin marks refund amount, system creates BillingTransaction (type: refund), credits to company balance. Company can use balance for future purchases or request bank wire refund via support ticket.)
- What happens if admin suspends account with active interviews in progress? (Active interviews continue until completion/expiry. New invitations blocked immediately. Suspended status does NOT terminate in-progress interviews to avoid candidate disruption.)
- How to handle timezone differences for anniversary date calculations? (Use company's configured timezone for all date calculations. Prorated charge calculations use days remaining in company timezone. Display timezone info in admin interface.)
- What happens if admin tries to change plan for a company with cancelled status? (System blocks with message: "Cannot change plan for cancelled subscription. Reactivate first." Reactivation requires selecting new plan and processing payment.)
- How are duplicate bank transfers handled (company accidentally submits same receipt twice)? (System flags potential duplicates based on receipt hash, amount, date, company. Admin sees warning: "⚠️ Potential duplicate: similar payment received [date]". Admin must verify before approving.)
- What happens if admin credits quota but company balance is negative? (Quota credit is independent of balance. System allows, but blocks new invitations until balance is topped up to cover estimated costs. Display notice: "Quota credited but balance negative (RM-50). Company cannot send invitations until balance positive.")
- How to handle prorated calculations for leap years? (Use actual days in subscription year (365 or 366) for prorated calculations. Formula: prorated_amount = annual_price × days_remaining / days_in_year.)

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: Display all companies dashboard: System MUST provide back-office admin dashboard displaying all companies with columns: company name, subscription tier, status, annual quota, interviews used, remaining, reserved, available balance, last payment date, anniversary date, and actions.

- **FR-002**: Filter and search companies: System MUST support filtering companies by status (Active/Suspended/Cancelled), subscription tier, quota usage threshold (e.g., >90%), balance threshold (e.g., <RM100), and text search by company name.

- **FR-003**: Export company data: System MUST allow admins to export filtered company list to CSV/Excel format with all displayed columns for reporting and analysis.

- **FR-004**: View pending payments queue: System MUST maintain a pending payments queue showing all submitted bank transfer receipts awaiting approval, with columns: company name, transaction type, amount, submission date, receipt URL, and actions (View/Approve/Reject).

- **FR-005**: Process bank transfer payments: System MUST allow admins to view receipt, approve/reject payment, and on approval: credit account balance (for top-ups), deduct from balance and add quota (for individual purchases), or process subscription payment, create BillingTransaction record, send confirmation email, and move to completed queue.

- **FR-006**: Reject payments with reason: System MUST allow admins to reject payments with mandatory reason text, send rejection email to company with reason and resubmission instructions, and move to rejected queue.

- **FR-007**: Manual transaction adjustments: System MUST allow admins to manually add/deduct balance or quota for any company with mandatory admin note, create BillingTransaction record (type: manual_credit/manual_debit), log admin user ID and timestamp, and send notification email to company.

- **FR-008**: View company subscription details: System MUST provide detailed company view with sections: Subscription Overview (tier, status, dates, cost), Quota Status (plan/individual breakdown, reserved, available), Balance & Transactions (current balance, recent transactions), Usage Analytics (graphs), Interview History (table), and Administrative Actions (buttons).

- **FR-009**: Display quota breakdown chart: System MUST visualize company quota status as stacked bar chart showing: used (plan quota), used (individual), reserved, available (plan), available (individual), for clear visual understanding of quota allocation.

- **FR-010**: View company transaction history: System MUST display paginated transaction history for each company showing all billing transactions (subscriptions, purchases, top-ups, refunds, settlements) with date, type, amount, description, invoice, and processor.

- **FR-011**: System-wide analytics dashboard: System MUST provide aggregate analytics showing: total active subscriptions by tier, MRR/ARR, interviews conducted this month, total account balance, average quota utilization, pending payments count, with drill-down capability.

- **FR-012**: Revenue reports and exports: System MUST provide revenue visualization (line graphs, breakdowns by source) with date range filtering and export functionality (Excel) with sheets: Summary, Subscription Renewals, Individual Purchases, Balance Top-ups, Refunds, including transaction IDs for reconciliation.

- **FR-013**: Subscription distribution analytics: System MUST display subscription tier distribution (pie chart), subscription growth over time (bar chart showing new subscriptions, cancellations, net change per month), for business intelligence.

- **FR-014**: Quota consumption trends: System MUST show system-wide interview completion trends (line graph over 90 days), quota utilization by tier (average %), and alert for companies approaching exhaustion (>90% usage) with count and list view.

- **FR-015**: Manual subscription plan changes: System MUST allow admins to upgrade/downgrade company subscription plans with prorated charge/refund calculation preview showing: remaining days, old plan credit, new plan cost, net charge/refund, new quota, and confirmation flow.

- **FR-016**: Prorated calculation accuracy: System MUST calculate prorated charges/refunds using formula: `prorated_amount = annual_price × days_remaining / days_in_subscription_year` where days_in_subscription_year accounts for leap years (365 or 366 days).

- **FR-017**: Handle downgrade with overage: System MUST detect when downgrade would result in usage exceeding new quota, display warning to admin, and on confirmation: update plan, set `overage_mode: true`, block new invitations, preserve existing data, and notify company of blocked invitations until anniversary reset.

- **FR-018**: Suspend accounts: System MUST allow admins to suspend company accounts with mandatory reason, update status to "Suspended", cancel all active quota reservations and release quota, block new invitations (preserve in-progress interviews), and send suspension notification email with contact information and outstanding amount.

- **FR-019**: Cancel subscriptions: System MUST allow admins to cancel subscriptions with prorated refund calculation, update status to "Cancelled", set service_end_date to current anniversary date, issue refund (credit to balance or bank transfer), send cancellation confirmation, and maintain access until service_end_date.

- **FR-020**: Global billing transactions search: System MUST provide global billing transactions view searchable by company name, transaction type, date range, amount range, status, with paginated results and full transaction details modal showing: company, type, amount, date, invoice, receipt, processor, quota/balance changes, audit trail.

- **FR-021**: Export billing transactions: System MUST allow admins to export filtered billing transactions to Excel with all transaction details and subtotals by transaction type for accounting reconciliation.

- **FR-022**: Quota reservations monitor: System MUST provide quota reservations dashboard showing all active, expired, consumed, and released reservations with columns: company, interview ID, reserved count, reserved from, estimated cost, reserved at, expires at, status, and [Release] action for active reservations.

- **FR-023**: Manual quota release: System MUST allow admins to manually release active or expired reservations, decrement reserved_interviews_count in CompanySubscription, update QuotaReservation status to "Released", log admin action in audit trail with user ID, and show success confirmation.

- **FR-024**: Reservation analytics: System MUST display aggregate reservation metrics: total active reservations, total reserved quota value, average reservation age, conversion rate (reservations → completions), expiry rate, cancellation rate, for system health monitoring.

- **FR-025**: Audit trail for admin actions: System MUST log all administrative actions (payment processing, manual adjustments, plan changes, suspensions, cancellations, quota releases) with timestamp, admin user ID, action type, company affected, before/after state, and reason/note, viewable in Admin Audit Log section.

- **FR-026**: Invoice regeneration: System MUST allow admins to regenerate and resend invoice PDFs for any completed transaction, preserving original transaction data but with new generation timestamp noted on invoice.

- **FR-027**: Duplicate payment detection: System MUST flag potential duplicate bank transfer submissions based on receipt hash, amount, date, and company ID, displaying warning to admin: "⚠️ Potential duplicate: similar payment received [date]" requiring explicit confirmation to proceed.

- **FR-028**: Concurrent action prevention: System MUST implement optimistic locking for payment processing to prevent concurrent admin actions on same payment, displaying error to second admin: "Payment already processed by [Admin Name] at [timestamp]" with reload option.

- **FR-029**: Payment processing SLA tracking: System MUST track payment processing time from submission to approval/rejection, display average processing time in Analytics Dashboard (target: <1 business day), and flag payments pending >2 business days for admin attention.

- **FR-030**: Negative balance handling: System MUST allow quota credits even when company balance is negative, but block new invitations until balance is positive, displaying notice to admin: "Quota credited but balance negative (RM-X). Company cannot send invitations until balance positive."

- **FR-031**: Admin role permissions: System MUST enforce role-based access control within back-office portal: Super Admin (full access), Finance Admin (payment processing, view-only otherwise), Support Admin (view-only all, manual adjustments limited to quota credits).

- **FR-032**: Manual balance top-up: System MUST allow admins to initiate manual balance top-up for any company (e.g., after receiving bank wire for top-up request), specifying amount, payment method, reference number, and auto-generating BillingTransaction and confirmation email.

- **FR-033**: Payment rejection history: System MUST maintain rejected payments history with rejection reason, rejected by admin, rejection date, and allow companies to resubmit from client portal with reference to original submission for admin context.

### Key Entities

**Note**: Entity definitions are based on existing database schema in `ai-talent-analyst-system-api/atas/models/` and `authentication/models.py`. All entities use Django ORM models with PostgreSQL database (`atas` and `public` schemas).

#### Core Entities (Existing System)

- **Company** (Table: `atas.companies`)
  - id (PK), name, industry, location, phone, email
  - created_by (FK → AuthUser), updated_by (FK → AuthUser)
  - created_at, updated_at, deleted_at (soft delete support)

- **CompanySubscription** (Table: `atas.company_subscriptions`)
  - id (PK), company (FK → Company), plan (FK → SubscriptionPlan)
  - billing_period (FK → BillingPeriod), currency (FK → Currency)
  - price_snapshot (Decimal), quota_snapshot (Integer)
  - start_at (DateTime), end_at (DateTime, nullable)
  - auto_renew (Boolean), status (FK → Status)
  - is_enterprise (Boolean), overage_price_snapshot (Decimal, nullable)
  - created_at, updated_at, created_by (FK → AuthUser), updated_by (FK → AuthUser)

- **SubscriptionPlan** (Table: `atas.subscription_plans`)
  - id (PK), code (CharField, unique - e.g., "GOLDFISH", "DOLPHIN", "WHALE")
  - name (CharField), description (TextField)
  - base_interview_quota (Integer)
  - is_active (Boolean)
  - created_at, updated_at, created_by (FK → AuthUser), updated_by (FK → AuthUser)

- **SubscriptionPlanPrice** (Table: `atas.subscription_plan_prices`)
  - id (PK), plan (FK → SubscriptionPlan), currency (FK → Currency)
  - billing_period (FK → BillingPeriod)
  - subscription_price_amount (Decimal), overage_price_per_interview (Decimal, nullable)
  - created_at, updated_at, created_by (FK → AuthUser), updated_by (FK → AuthUser)
  - Unique together: (plan, currency, billing_period)

- **Account** (Table: `atas.accounts`)
  - id (PK), company (OneToOne → Company)
  - wallet_available_balance (Decimal) - Available after RESERVE deductions
  - wallet_held_balance (Decimal) - Currently RESERVED funds
  - wallet_total_balance (Decimal) - available + held
  - quota_remaining_cached (Integer), quota_used_cached (Integer)
  - currency (FK → Currency), last_recalculated_at (DateTime, nullable)
  - created_at, updated_at

- **BillingTransaction** (Table: `atas.billing_transactions`)
  - id (PK), resource_type (WALLET / QUOTA / TOKEN)
  - transaction_type (RESERVE / RELEASE / COMMIT / CREDIT / ADJUSTMENT)
  - amount (Decimal), resource_delta (Integer, for QUOTA type)
  - transaction_group_id (UUID, groups related transactions)
  - idempotency_key (CharField, unique, prevents double billing)
  - metadata (JSONField), account (FK → Account)
  - session (FK → Session, nullable), reference_txn (FK → self, nullable)
  - created_at, updated_at
  - **IMMUTABLE**: Cannot be updated or deleted after creation (ledger integrity)

- **TopupRequest** (Table: `atas.topup_requests`)
  - account (FK → Account), amount (Decimal), currency (FK → Currency)
  - receipt_file (FileField, upload_to="topup_receipts/")
  - bank_reference (CharField, unique)
  - status (FK → Status, nullable)
  - approved_by (FK → AuthUser, nullable), approved_at (DateTime, nullable)
  - rejected_by (FK → AuthUser, nullable), rejected_at (DateTime, nullable)
  - cancelled_by (FK → AuthUser, nullable), cancelled_at (DateTime, nullable)
  - created_at

- **Session** (Table: `atas.sessions`)
  - id (PK), chat_history (JSONField)
  - scheduled_at, started_at, ended_at (DateTime, nullable)
  - interview_score, learning_agility_score, jd_resume_score, salary_score, total_score (Float)
  - resume (FK → Resume), job_description (FK → JobDescription)
  - jd_version (FK → JobDescriptionVersion), jd_locked_at (DateTime)
  - qa_prompt (FK → Prompt), scoring_prompt (FK → Prompt)
  - candidate (FK → Candidate), company (FK → Company)
  - token_expires_at, token_hash (CharField, unique)
  - status (FK → Status), interview_analysis (JSONField)
  - created_at, updated_at, deleted_at (soft delete)
  - created_by (FK → AuthUser), updated_by (FK → AuthUser)

- **SubscriptionChangeLog** (Table: `atas.subscription_change_logs`)
  - id (PK), subscription (FK → CompanySubscription)
  - from_plan (FK → SubscriptionPlan, nullable), to_plan (FK → SubscriptionPlan, nullable)
  - proration_amount (Decimal), effective_at (DateTime)
  - created_at, updated_at
  - created_by (FK → AuthUser), updated_by (FK → AuthUser)

- **SubscriptionOverride** (Table: `atas.subscription_overrides`)
  - id (PK), subscription (FK → CompanySubscription)
  - override_price (Decimal, nullable), override_quota (Integer, nullable)
  - reason (TextField)
  - created_at, updated_at
  - created_by (FK → AuthUser), updated_by (FK → AuthUser)

- **Currency** (Table: `atas.currencies`)
  - id (PK), code (e.g., "MYR", "USD"), name, symbol
  - exchange_rate (Decimal), is_active (Boolean)
  - created_at, updated_at

- **BillingPeriod** (Table: `atas.billing_periods`)
  - id (PK), code (e.g., "ANNUAL", "MONTHLY"), name, duration_months (Integer)
  - created_at, updated_at

- **Status** (Table: `atas.statuses`)
  - id (PK), module (e.g., "SESSION", "SUBSCRIPTION"), code (e.g., "ACTIVE", "COMPLETED")
  - name, description
  - created_at, updated_at

#### Authentication & Authorization Entities (Existing System)

- **AuthUser** (Table: `public.auth_user`)
  - email (EmailField, unique), username (CharField, unique), password (CharField)
  - user_type ("client" / "backoffice"), joined_date, last_login
  - is_active, is_admin, is_superuser (Boolean)
  - first_name, last_name, status, attempt (Integer)
  - allowed_multiple (Boolean), company (FK → Company, nullable)

- **BackofficeUser** (Table: `public.bo_user_profile`)
  - user (OneToOne → AuthUser, PK)
  - employee_id (CharField, unique), role (admin / manager / analyst / support / viewer)
  - department (CharField), access_level (1-5), status (active / inactive / suspended / pending)
  - created_at, updated_at, created_by (FK → AuthUser)
  - last_activity (DateTime), phone, notes (TextField)

- **ClientUser** (Table: `public.client_user`)
  - user (OneToOne → AuthUser, PK), company (FK → Company)
  - permission_level (owner / admin / manager / member / viewer)
  - subscription_tier (free / basic / professional / enterprise)
  - status (active / inactive / suspended / trial / expired)
  - max_sessions, sessions_used, max_storage_mb, storage_used_mb
  - subscription_start_date, subscription_end_date, trial_end_date
  - created_at, updated_at, invited_by (FK → AuthUser)
  - last_activity, position, phone, notifications_enabled (Boolean)

- **AuthGroup** (Table: `public.atas_auth_group`)
  - name, description, applicable_user_type (client / backoffice / both)
  - company (FK → Company, nullable - global if null)
  - is_system_critical (Boolean, cannot be deleted)
  - permissions (ManyToMany → AuthPermission)
  - created_at, updated_at, created_by (FK → AuthUser)
  - Unique together: (company, name)

- **AuthPermission** (Table: `public.atas_auth_permission`)
  - resource (CharField), action (CharField), description
  - applicable_user_type (client / backoffice / both)
  - is_cross_company (Boolean, for backoffice cross-company access)
  - is_active (Boolean)
  - created_at, updated_at

- **UserGroupAssignment** (Table: `public.atas_user_group_assignment`)
  - user (FK → AuthUser), group (FK → AuthGroup)
  - assigned_by (FK → AuthUser), is_active (Boolean)
  - expires_at (DateTime, nullable)
  - created_at, updated_at

#### New Entities for Back-Office Management (To Be Implemented)

- **AdminAuditLog**: Logs all administrative actions for compliance with attributes:
  - id, admin_user (FK → AuthUser with user_type='backoffice')
  - action_type (approve_topup / reject_topup / manual_credit / manual_debit / plan_change / suspend_account / cancel_subscription / release_reservation / regenerate_invoice)
  - company (FK → Company), affected_entity_type (topup_request / subscription / billing_transaction / session)
  - affected_entity_id (Integer)
  - before_state (JSONField snapshot), after_state (JSONField snapshot)
  - reason_or_note (TextField)
  - ip_address (CharField), user_agent (TextField)
  - created_at

- **SystemAnalytics**: Pre-computed aggregate metrics for dashboard performance with attributes:
  - id, metric_date (Date, for daily aggregates)
  - total_active_subscriptions, mrr (Decimal), arr (Decimal)
  - total_interviews_this_month, total_account_balance (Decimal)
  - average_quota_utilization_percent (Float)
  - pending_topup_requests_count
  - goldfish_subscription_count, dolphin_subscription_count, whale_subscription_count
  - revenue_subscriptions (Decimal), revenue_individual_purchases (Decimal), revenue_topups (Decimal)
  - computed_at (DateTime)

- **PaymentProcessingSLA**: Tracks payment processing performance with attributes:
  - id, topup_request (FK → TopupRequest)
  - submitted_at, processed_at (DateTime)
  - processing_duration_hours (Float, calculated)
  - sla_met (Boolean, true if processed within 1 business day)
  - computed_at (DateTime)

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Back-office admins can view all companies with subscription and balance status in under 2 seconds page load time, with filters applied in <1 second

- **SC-002**: Payment approval/rejection workflow completes in under 30 seconds from admin action to account crediting/notification email sent, with 100% accuracy in balance/quota updates

- **SC-003**: Company subscription details page loads with all sections (subscription, quota, balance, transactions, usage analytics, interview history) in under 3 seconds

- **SC-004**: Prorated charge/refund calculations for plan changes are accurate to ±RM1 (accounting for rounding) and match formula: `prorated_amount = annual_price × days_remaining / days_in_subscription_year`

- **SC-005**: Manual transaction adjustments (add/deduct balance or quota) are reflected in company account within 5 seconds, with BillingTransaction record created and email notification sent

- **SC-006**: System-wide analytics dashboard displays aggregate metrics (MRR, ARR, subscriptions, quota utilization) with data accuracy 100% matching underlying transaction data, computed and cached daily

- **SC-007**: Revenue reports and exports generate Excel files with up to 10,000 transactions in under 10 seconds, with accurate subtotals by transaction type and no missing records

- **SC-008**: Admin audit trail logs 100% of administrative actions with timestamp, user ID, before/after state, and reason, with no data loss and searchable by action type, admin, company, or date range

- **SC-009**: Quota reservation monitoring dashboard displays all active reservations system-wide (up to 1,000 records) in under 2 seconds, with aging indicators accurate to current time

- **SC-010**: Manual quota release operations complete in under 5 seconds from admin action to QuotaReservation status update, reserved_interviews_count decrement, and audit log entry creation

- **SC-011**: Duplicate payment detection flags potential duplicates with 95%+ accuracy (based on receipt hash + amount + date similarity), preventing accidental double-crediting

- **SC-012**: Concurrent payment processing prevention works 100% of time, with second admin receiving immediate error notification and no double-processing incidents

- **SC-013**: Payment processing SLA tracking shows average processing time <1 business day for 90%+ of payments, with flagging for payments pending >2 business days

- **SC-014**: Global billing transactions search returns results for up to 50,000 transactions with date range and type filters applied in under 2 seconds, paginated at 50 records per page

- **SC-015**: Admin role permissions enforcement prevents unauthorized actions 100% of time (e.g., Support Admin cannot approve payments, Finance Admin cannot suspend accounts)

- **SC-016**: Subscription plan change operations (upgrade/downgrade) complete in under 1 minute from admin confirmation to CompanySubscription update, BillingTransaction creation, invoice generation, and email notification

- **SC-017**: Account suspension operations complete in under 30 seconds from admin action to status update, quota reservation cancellations, access blocking, and suspension email notification

- **SC-018**: Invoice regeneration and resend operations complete in under 10 seconds from admin action to PDF generation and email delivery

- **SC-019**: Company data exports (CSV/Excel) generate files with up to 1,000 companies in under 5 seconds with all filtered columns included and no data truncation

- **SC-020**: Usage analytics graphs (interview completions over time, quota breakdown by tier) render correctly in admin dashboard with data points for up to 90 days and accurate calculations

---

## Clarifications & Open Questions

### Answered Clarifications

1. **Admin Role Permissions**: ✅ **CLARIFIED** - Three admin roles: Super Admin (full access), Finance Admin (payment processing and view-only otherwise), Support Admin (view-only with limited quota credit capability). Role enforcement at API and UI levels.

2. **Payment Processing Workflow**: ✅ **CLARIFIED** - Bank transfer receipts submitted by companies are queued in PaymentQueue. Admins view receipt, verify against bank statement, approve/reject. On approval: system credits balance/quota, creates BillingTransaction, sends confirmation. Processing SLA target: <1 business day.

3. **Prorated Charge Calculation**: ✅ **CLARIFIED** - Use anniversary-based billing. Formula: `prorated_amount = annual_price × days_remaining / days_in_subscription_year` where days_in_subscription_year = 365 or 366 (for leap years). Days remaining calculated from action date to next anniversary date in company timezone.

4. **Refund Issuance Method**: ✅ **CLARIFIED** - Refunds from cancellations/downgrades are credited to company account balance by default. Companies can use balance for future purchases or request bank wire refund via support ticket (manual processing outside system scope for MVP).

5. **Duplicate Payment Handling**: ✅ **CLARIFIED** - System flags potential duplicates using receipt file hash, amount, date (±1 day), and company ID. Admin sees warning and must explicitly confirm "This is NOT a duplicate" before processing. Audit log records admin decision.

6. **Timezone Handling**: ✅ **CLARIFIED** - All date/time calculations use company's configured timezone stored in Company entity. Prorated calculations, anniversary dates, and SLA tracking respect company timezone. Admin interface displays timezone for clarity (e.g., "Anniversary: Dec 1, 2026 00:00 MYT").

7. **Invoice Regeneration**: ✅ **CLARIFIED** - Admins can regenerate invoices for any completed transaction (e.g., if company lost invoice or data correction needed). Regenerated invoice preserves original transaction data but notes "Regenerated on [date]" for audit trail. New PDF URL stored in BillingTransaction without creating duplicate transaction record.

8. **Manual Adjustment Notifications**: ✅ **CLARIFIED** - All manual adjustments (balance/quota credits or debits) trigger email notification to all company admins with details: adjustment type, amount/count, admin reason, new balance/quota, and contact info for questions.

9. **Concurrent Action Prevention**: ✅ **CLARIFIED** - Use optimistic locking on PaymentQueue records. When admin loads payment for processing, record version incremented. On save, system checks version. If mismatch (another admin processed), transaction fails with error message identifying first processor and timestamp.

10. **Suspended Account Access**: ✅ **CLARIFIED** - Suspended accounts retain read-only access to platform (view dashboard, past interviews, billing history) but cannot send new invitations or modify data. In-progress interviews continue to completion to avoid candidate disruption. Reactivation requires payment of outstanding balance and plan selection.

### Open Questions

1. **Credit Card Payment Integration**: When will credit card payment support (Stripe/PayPal) be added to reduce manual processing burden? (Planned for Phase 3 based on business needs and payment volume.)

2. **Automated Refund Processing**: Should refunds be automatically issued via bank transfer API integration or continue as manual process? (Future enhancement, manual process sufficient for MVP due to low refund volume expected.)

3. **Multi-Currency Support**: Will system need to support currencies other than RM (Malaysian Ringgit)? (Not in scope for Phase 2. All pricing in RM only.)

4. **Notification Preferences**: Should admins be able to configure notification preferences (email, SMS, in-app) for different alert types? (Future enhancement, email-only notifications sufficient for MVP.)

5. **Company Lifecycle Automation**: Should system automatically suspend accounts after X days of failed payment attempts? (Not in MVP scope. Manual suspension by admin after payment follow-up currently.)
