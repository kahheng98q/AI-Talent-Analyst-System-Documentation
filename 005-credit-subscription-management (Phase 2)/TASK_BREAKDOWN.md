# Task Breakdown: Credit & Subscription Management

**Source**: [005-credit-subscription-management/spec.md](spec.md)  
**Generated**: January 28, 2026  
**Updated**: February 1, 2026 (Added cross-area linkages and reorganized API under Backend)  
**Status**: Planning

---

## Overview

Feature 005 implements a comprehensive subscription and billing system with three tiered plans (Gold Fish, Dolphin, Whale), interview quota management with reservation system, account balance tracking, individual interview purchases, usage analytics, billing history, and automated notifications. The system uses a hybrid quota + balance model where subscriptions provide annual interview quotas and all companies maintain account balances for pay-as-you-go costs.

**Key Capabilities**:

- Subscription tier management with annual interview quotas
- Interview quota reservation (on invitation send) and settlement (on completion/expiry)
- Account balance tracking with top-up support
- Individual interview purchases at RM15 each
- Usage analytics with charts (daily/monthly views)
- Billing history with PDF invoice generation
- Automated quota and balance alerts (80%, 100%, low balance)
- Mid-year subscription upgrades/downgrades with prorated charges

**Cross-Referencing System**:

This task breakdown uses explicit cross-area linkages to show dependencies and relationships:

- **Frontend tasks** include **"Uses"** field referencing Backend API endpoints/services they call
- **Backend tasks** include **"Called By"** field showing which Frontend/other Backend tasks consume them
- **Backend API endpoints** include **"Service Method"** field showing which service layer methods they delegate to
- **Database tasks** include **"Used By"** field showing which Backend services depend on them
- **All implementation tasks** include **"Tested By"** field referencing test tasks that validate them
- **Task numbering**: Use format "Frontend 1.1", "Backend 2.3", "Database 1.2", "Testing 3.1" for cross-references

**Note**: API endpoints are organized under Backend as they represent the backend's interface layer.

---

## Frontend

### Main Task 1: Subscription Dashboard Overview

**Description**: Create the main subscription dashboard page that displays subscription tier, interview quota breakdown (plan + individual), reserved quota, account balance, and quick action buttons.

**User Story Reference**: P1 User Story 1 - View Subscription Status & Interview Allocation

#### Subtask 1.1: Create Subscription Status Card Component

- **Description**: Build a card component displaying subscription tier (Gold Fish/Dolphin/Whale), status (Active/Suspended/Cancelled), annual allocation, interviews used, remaining, reserved quota, available quota, account balance, and anniversary date
- **Uses**: Backend Task 1.8 (GET /api/subscriptions/status endpoint)
- **Tested By**: Testing Task 3.1 (Subscription Dashboard E2E Test)
- **Acceptance**: Card displays all fields accurately, updates in real-time when quota changes, shows color-coded status badges (green=active, yellow=warning, red=suspended)
- **Files**: `components/SubscriptionStatusCard.jsx`, `styles/subscription-status.css`
- **User Story**: P1 User Story 1, Scenario 1

#### Subtask 1.2: Implement Low Quota Warning Banner

- **Description**: Create a warning banner component that appears at top of dashboard when quota reaches 80% (yellow) or 100% (red) usage thresholds
- **Uses**: Backend Task 1.1 (SubscriptionService.getSubscriptionStatus method)
- **Tested By**: Testing Task 3.1 (Subscription Dashboard E2E Test)
- **Acceptance**: Banner appears automatically when threshold crossed, includes remaining count, call-to-action buttons ([Upgrade Now], [Purchase Interviews]), dismissible but reappears on next login
- **Files**: `components/QuotaWarningBanner.jsx`
- **User Story**: P1 User Story 1, Scenarios 3-4

#### Subtask 1.3: Implement Low Balance Warning Banner

- **Description**: Create a warning banner that appears when account balance drops below RM50 threshold, blocking new invitations if balance insufficient
- **Uses**: Backend Task 1.6 (AccountBalanceService methods)
- **Tested By**: Testing Task 3.1 (Subscription Dashboard E2E Test)
- **Acceptance**: Banner displays current balance, minimum required amount, includes [Top Up Now] button, shows real-time balance updates
- **Files**: `components/BalanceWarningBanner.jsx`
- **User Story**: P1 User Story 1, Scenario 5

#### Subtask 1.4: Create Quick Action Buttons Section

- **Description**: Build action button group for common subscription operations: [Upgrade Plan], [Purchase Interviews], [Top Up Balance], [View Billing History]
- **Acceptance**: Buttons trigger appropriate modals/pages, disabled states when action not available (e.g., Upgrade disabled if already on Whale tier)
- **Files**: `components/SubscriptionActions.jsx`
- **User Story**: P1 User Story 1

#### Subtask 1.5: Implement Subscription Tier Comparison Modal

- **Description**: Create modal/overlay showing all three subscription tiers with pricing, annual allocation, cost per interview, and feature comparison table
- **Acceptance**: Modal opens from dashboard, displays Gold Fish (RM3600/300), Dolphin (RM7200/800), Whale (RM12000/2000), highlights current tier, [Select Plan] buttons
- **Files**: `components/SubscriptionTierModal.jsx`
- **User Story**: P1 User Story 1, User Story 5

---

### Main Task 2: Usage Analytics Dashboard

**Description**: Build interactive charts and visualizations for interview usage tracking with daily/monthly toggles, date range filtering, and usage breakdown by status.

**User Story Reference**: P1 User Story 2 - View Interview Usage History & Analytics

#### Subtask 2.1: Create Line Graph for Usage Over Time

- **Description**: Implement line graph using Chart.js/Recharts showing interview completions over time with toggleable daily/monthly views and date range picker
- **Uses**: Backend Task 5.1 (GET /api/usage/timeseries endpoint)
- **Tested By**: Testing Task 3.2 (Usage Analytics E2E Test)
- **Acceptance**: Chart renders with time on X-axis, interview count on Y-axis, supports toggle between daily (last 30 days) and monthly (last 12 months) views, date range picker filters data accurately
- **Files**: `components/UsageLineChart.jsx`, `utils/chartConfig.js`
- **User Story**: P1 User Story 2, Scenarios 1-2

#### Subtask 2.2: Create Pie Chart for Usage Breakdown by Status

- **Description**: Build pie chart showing interview distribution by status (Completed, Pending, Cancelled) with percentages and legend
- **Uses**: Backend Task 5.2 (GET /api/usage/breakdown endpoint)
- **Tested By**: Testing Task 3.2 (Usage Analytics E2E Test)
- **Acceptance**: Pie chart displays correct percentages, legend shows color coding, tooltips show absolute counts, updates based on date range filter
- **Files**: `components/UsageBreakdownPieChart.jsx`
- **User Story**: P1 User Story 2, Scenario 3

#### Subtask 2.3: Create Dual-Axis Combined Chart

- **Description**: Implement combined chart with line graph (primary Y-axis) for usage over time and bar chart (secondary Y-axis) for monthly breakdown
- **Acceptance**: Both visualizations share X-axis (time), scales appropriately, legend differentiates line vs bars, responsive to window resizing
- **Files**: `components/UsageDualAxisChart.jsx`
- **User Story**: P1 User Story 2, Scenario 4

#### Subtask 2.4: Implement Date Range Picker Component

- **Description**: Create reusable date range picker with presets (Last 7 Days, Last 30 Days, Last 3 Months, Custom Range) and validation
- **Acceptance**: Picker validates end date > start date, shows error message for invalid ranges, applies filter to all charts simultaneously, supports up to 3 years of historical data
- **Files**: `components/DateRangePicker.jsx`
- **User Story**: P1 User Story 2, Scenario 5

#### Subtask 2.5: Create Usage Analytics Summary Cards

- **Description**: Build summary card components showing key metrics: total interviews this period, average per day/month, estimated depletion date, usage trend (↑/↓)
- **Acceptance**: Cards display accurate calculations based on filtered date range, trend indicators use rolling 7-day average, depletion date uses linear projection
- **Files**: `components/UsageSummaryCards.jsx`
- **User Story**: P1 User Story 2

---

### Main Task 3: Billing History & Invoices Interface

**Description**: Create billing history table with filtering, pagination, and PDF invoice download functionality.

**User Story Reference**: P1 User Story 3 - View Billing History & Invoices

#### Subtask 3.1: Create Billing History Table Component

- **Description**: Build data table displaying billing transactions with columns: Invoice Number, Date, Description, Amount, Transaction Type, Status, Actions (View/Download PDF)
- **Acceptance**: Table displays paginated data (20 per page), sortable by date/amount, status badges color-coded (Completed=green, Pending=yellow, Failed=red)
- **Files**: `components/BillingHistoryTable.jsx`, `styles/billing-table.css`
- **User Story**: P1 User Story 3, Scenario 1

#### Subtask 3.2: Implement Billing History Filters

- **Description**: Create filter panel for date range selection and transaction type dropdown (All/Subscription/Individual Purchase/Balance Top-up/Interview Settlement)
- **Acceptance**: Filters apply immediately on selection, persist in URL query params, clear filters button resets to defaults, filter count badge shows active filters
- **Files**: `components/BillingHistoryFilters.jsx`
- **User Story**: P1 User Story 3, Scenario 2

#### Subtask 3.3: Create PDF Invoice Download Handler

- **Description**: Implement client-side handler to trigger PDF invoice generation and download with loading state and error handling
- **Acceptance**: Click [Download PDF] button triggers API call, shows loading spinner, downloads PDF on success, displays error toast on failure, disables button during generation
- **Files**: `components/InvoiceDownloadButton.jsx`, `services/billingService.js`
- **User Story**: P1 User Story 3, Scenario 3

#### Subtask 3.4: Design PDF Invoice Template

- **Description**: Create PDF invoice template layout including company logo, invoice number, date, itemized charges table, subtotal/total, payment method, payment status, footer with company details
- **Acceptance**: Template renders correctly in PDF, includes all required fields from spec, supports multi-currency (RM), page breaks for long item lists, company branding
- **Files**: `templates/invoice-template.html`, `styles/invoice-pdf.css`
- **User Story**: P1 User Story 3, Scenario 3

#### Subtask 3.5: Implement Empty State for Billing History

- **Description**: Create empty state component shown when no billing history exists (new accounts) with informative message and next steps
- **Acceptance**: Displays message "No billing history available", explains when first invoice will appear (after first subscription or purchase), includes [View Plans] CTA
- **Files**: `components/BillingHistoryEmptyState.jsx`
- **User Story**: P1 User Story 3, Scenario 5

---

### Main Task 4: Individual Interview Purchase Flow

**Description**: Build UI flow for purchasing individual interviews including quantity selection, checkout summary, payment instructions, and confirmation.

**User Story Reference**: P1 User Story 5 - Purchase Individual Interviews

#### Subtask 4.1: Create Purchase Interview Modal

- **Description**: Build modal with quantity input (spinner or dropdown for 1-50 interviews), total cost calculation (quantity × RM15), available balance display, checkout button
- **Acceptance**: Modal opens from quota exhausted banner or dashboard action button, validates quantity > 0, calculates total in real-time, shows available balance vs required amount
- **Files**: `components/PurchaseInterviewModal.jsx`
- **User Story**: P1 User Story 5, Scenarios 1-2

#### Subtask 4.2: Implement Insufficient Balance Warning

- **Description**: Add validation and warning message when available balance is insufficient to cover purchase cost with [Top Up Now] CTA
- **Acceptance**: Warning appears immediately if balance < (quantity × RM15), disables checkout button, shows exact shortfall amount, link to top-up page
- **Files**: `components/PurchaseInterviewModal.jsx` (validation logic)
- **User Story**: P1 User Story 5, Scenario 5

#### Subtask 4.3: Create Bank Transfer Payment Instructions Page

- **Description**: Build page displaying bank transfer details (bank name, account number, account name), unique reference code for this purchase, amount to transfer, upload receipt button
- **Acceptance**: Page shows all payment details, generates unique reference code (format: COMP-[company_id]-INT-[timestamp]), includes file upload for receipt, instructions for email submission
- **Files**: `pages/PaymentInstructions.jsx`, `components/ReceiptUpload.jsx`
- **User Story**: P1 User Story 5, Scenario 2

#### Subtask 4.4: Implement Purchase Confirmation Page

- **Description**: Create confirmation page shown after purchase request submitted displaying purchase summary, pending status message, expected processing time (1 business day)
- **Acceptance**: Page displays purchase details (quantity, total cost), estimated credit date, instructions to check email for confirmation, [Return to Dashboard] button
- **Files**: `pages/PurchaseConfirmation.jsx`
- **User Story**: P1 User Story 5, Scenario 3

#### Subtask 4.5: Create Mixed Quota Display Component

- **Description**: Build component showing separate counters for plan quota and individual interviews quota with visual distinction
- **Acceptance**: Component displays "Plan Quota: X/Y used" and "Individual Interviews: Z remaining", total available calculation shown, color-coded (plan=blue, individual=green)
- **Files**: `components/MixedQuotaDisplay.jsx`
- **User Story**: P1 User Story 5, Scenario 4

---

### Main Task 5: Subscription Upgrade/Downgrade Flow

**Description**: Implement UI for subscription plan changes with prorated charge calculations and confirmation dialogs.

**Requirement Reference**: FR-013, FR-014, FR-015

#### Subtask 5.1: Create Plan Upgrade Modal

- **Description**: Build modal for selecting higher tier plan with side-by-side comparison of current vs new plan, prorated charge calculation, confirmation checkbox
- **Acceptance**: Modal shows current plan on left, available upgrade options on right, calculates prorated cost using (new_annual - current_annual) / 365 × days_remaining, requires confirmation checkbox before upgrade
- **Files**: `components/PlanUpgradeModal.jsx`, `utils/prorateCalculator.js`
- **User Story**: P1 User Story 5 (quota exhaustion options)

#### Subtask 5.2: Create Plan Downgrade Warning Dialog

- **Description**: Build warning dialog for downgrades explaining quota reduction, existing interview data preservation, new invitation blocking if usage > new quota
- **Acceptance**: Dialog shows current usage vs new quota limit, warning if usage exceeds new limit, explains read-only data access, requires explicit "I Understand" confirmation
- **Files**: `components/PlanDowngradeWarning.jsx`
- **Requirement**: FR-014

#### Subtask 5.3: Implement Subscription Cancellation Flow

- **Description**: Create multi-step cancellation flow with reason selection, prorated refund calculation, service end date display, final confirmation
- **Acceptance**: Flow collects cancellation reason (dropdown), shows prorated refund amount, displays service end date (current anniversary date), requires typing "CANCEL" to confirm
- **Files**: `components/SubscriptionCancellation.jsx`
- **Requirement**: FR-015

---

### Main Task 6: Account Balance Top-Up Interface

**Description**: Build interface for account balance top-up via bank transfer with amount selection, payment instructions, and confirmation.

**Requirement Reference**: FR-031, FR-032

#### Subtask 6.1: Create Top-Up Amount Selection Page

- **Description**: Build page with preset amount buttons (RM100, RM250, RM500, RM1000) and custom amount input, shows current balance, calculates new balance
- **Acceptance**: Page displays current balance prominently, preset buttons auto-populate amount field, custom input validates minimum RM50, calculates "New Balance = Current + Top-up"
- **Files**: `pages/AccountTopUp.jsx`, `components/TopUpAmountSelector.jsx`
- **Requirement**: FR-031

#### Subtask 6.2: Create Top-Up Payment Instructions

- **Description**: Display bank transfer instructions specifically for balance top-up with unique reference code format (COMP-[id]-BAL-[timestamp])
- **Acceptance**: Shows bank details, top-up amount, unique reference for balance top-up, receipt upload, email submission instructions
- **Files**: `pages/TopUpPaymentInstructions.jsx`
- **Requirement**: FR-031

#### Subtask 6.3: Implement Balance Top-Up Confirmation

- **Description**: Create confirmation page after top-up request submitted showing pending status, processing time (1 business day), return to dashboard link
- **Acceptance**: Displays top-up amount, current balance (unchanged), expected credit date, email confirmation instructions
- **Files**: `pages/TopUpConfirmation.jsx`
- **Requirement**: FR-031

---

## Backend

### Main Task 1: Subscription & Quota Management Service

**Description**: Implement core business logic for subscription management, quota calculations, reservation system, and balance tracking.

**Requirement Reference**: FR-001, FR-027, FR-028, FR-029, FR-030, FR-033

#### Subtask 1.1: Create Subscription Service Class

- **Description**: Build service class with methods for subscription CRUD operations, quota calculations, status checks, anniversary date calculations
- **Called By**: Backend Task 1.8 (GET /api/subscriptions/status endpoint), Frontend Task 1.1 (Subscription Status Card)
- **Tested By**: Testing Task 1.1 (Subscription Service Unit Tests)
- **Acceptance**: Service provides methods: `getSubscriptionStatus()`, `calculateRemainingQuota()`, `checkQuotaAvailability()`, `getAnniversaryDate()`, all methods return accurate data based on CompanySubscription entity
- **Files**: `services/SubscriptionService.js`, `models/CompanySubscription.js`
- **Requirement**: FR-001

#### Subtask 1.2: Implement Quota Reservation Logic

- **Description**: Create method to reserve interview quota when HR sends invitation, checking quota availability (plan + individual) and balance sufficiency
- **Depends On**: Database Task 1.2 (CompanySubscription table), Database Task 1.3 (QuotaReservation table)
- **Called By**: Backend Task 1.9 (POST /api/quota/reserve endpoint)
- **Tested By**: Testing Task 1.2 (Quota Reservation Unit Tests), Testing Task 2.2 (Quota Management API Integration Tests)
- **Acceptance**: Method `reserveInterviewQuota(companyId, estimatedCost)` creates QuotaReservation record, decrements available quota, validates balance ≥ estimatedCost, returns reservation object or error
- **Files**: `services/QuotaReservationService.js`, `models/QuotaReservation.js`
- **Requirement**: FR-027, FR-030

#### Subtask 1.3: Implement Quota Settlement on Interview Completion

- **Description**: Create method to convert reserved quota to consumed quota when candidate completes interview, settle actual cost against balance, create billing transaction
- **Acceptance**: Method `settleInterviewCompletion(reservationId, actualCost)` updates reservation status to 'consumed', decrements subscription or individual quota (plan first), deducts actualCost from balance, creates InterviewCompletion + BillingTransaction records
- **Files**: `services/QuotaReservationService.js`, `models/InterviewCompletion.js`
- **Requirement**: FR-028
- **Dependencies**: QuotaReservation must exist

#### Subtask 1.4: Implement Quota Release on Invitation Expiry

- **Description**: Create scheduled job to release reserved quota for expired invitations (3-day timeout), update reservation status to 'released'
- **Acceptance**: Background job runs hourly, finds QuotaReservation records with expires_at < now AND status='active', updates status to 'released', increments available quota, logs release timestamp
- **Files**: `jobs/ExpiredReservationCleanup.js`, `services/QuotaReservationService.js`
- **Requirement**: FR-029
- **Dependencies**: QuotaReservation entity

#### Subtask 1.5: Implement Mixed Quota Consumption Logic

- **Description**: Create method to determine quota source (plan vs individual) when consuming quota, prioritizing plan quota first
- **Acceptance**: Method `determineQuotaSource(companyId)` returns 'plan_quota' if subscription_interviews_remaining > 0, else returns 'individual_purchase', used by settlement logic
- **Files**: `services/SubscriptionService.js`
- **Requirement**: FR-010
- **User Story**: P1 User Story 5, Scenario 6

#### Subtask 1.6: Implement Account Balance Tracking

- **Description**: Create methods for balance operations: check balance, reserve balance (for estimated cost), deduct balance (on settlement), release reserved balance (on expiry)
- **Acceptance**: Methods maintain available_balance calculation (total_balance - reserved_balance), validate sufficient funds before operations, create audit trail for all balance changes
- **Files**: `services/AccountBalanceService.js`
- **Requirement**: FR-030, FR-032

#### Subtask 1.7: Implement Quota Exhaustion Blocking

- **Description**: Create validation method to check quota availability and balance sufficiency before allowing invitation send, return structured error with options
- **Called By**: Backend Task 1.12 (GET /api/quota/validate-invitation endpoint), Feature 001 Interview Module
- **Tested By**: Testing Task 1.2 (Quota Reservation Unit Tests), Testing Task 3.5 (Quota Exhaustion E2E Test)
- **Acceptance**: Method `validateInvitationEligibility(companyId)` checks: (1) active subscription exists, (2) available_quota > 0, (3) balance ≥ estimatedCost, returns error object with failure reasons and suggested actions ([Upgrade], [Purchase], [Top Up])
- **Files**: `services/SubscriptionService.js`
- **Requirement**: FR-000, FR-000a, FR-023, FR-032, FR-033
- **User Story**: P1 User Story 5, Scenario 1

#### Subtask 1.8: GET /api/subscriptions/status

- **Description**: API endpoint to retrieve current subscription status for authenticated company with quota, balance, and anniversary details
- **Service Method**: Calls SubscriptionService.getSubscriptionStatus() from Backend Task 1.1
- **Called By**: Frontend Task 1.1 (Subscription Status Card)
- **Tested By**: Testing Task 2.1 (Subscription API Integration Tests), Testing Task 3.1 (Subscription Dashboard E2E Test)
- **Request**: Headers: Authorization (JWT with company_id)
- **Response**: `{ status: "active", tier: "Gold Fish", annual_allocation: 300, interviews_used: 12, remaining: 288, reserved: 5, available: 283, account_balance: 500, anniversary_date: "2026-12-01" }`
- **Acceptance**: Returns 200 with subscription details, 404 if no subscription exists, enforces company_id from JWT
- **User Story**: P1 User Story 1, Scenario 1

#### Subtask 1.9: POST /api/quota/reserve

- **Description**: API endpoint to reserve interview quota when HR sends invitation
- **Service Method**: Calls QuotaReservationService.reserveInterviewQuota() from Backend Task 1.2
- **Called By**: Feature 001 Interview Module (on candidate invitation send)
- **Tested By**: Testing Task 2.2 (Quota Management API Integration Tests)
- **Request**: `{ estimated_cost: 12, interview_id: 456 }`
- **Response**: `{ success: true, reservation_id: 789, reserved_from: "plan_quota", expires_at: "2026-01-31T00:00:00Z" }`
- **Acceptance**: Returns 201 on success with reservation details, 400 if insufficient quota or balance, creates QuotaReservation record with status='active'
- **Requirement**: FR-027, FR-030

---

### Main Task 2: Individual Interview Purchase Processing

**Description**: Implement backend logic for individual interview purchases via bank transfer with admin approval workflow.

**Requirement Reference**: FR-008, FR-009, FR-016, FR-017, FR-025

#### Subtask 2.1: Create Purchase Request Handler

- **Description**: Build API handler to create purchase request record with quantity, total cost, payment instructions generation, unique reference code
- **Acceptance**: Handler validates quantity > 0, calculates total = quantity × RM15, checks balance sufficiency, generates unique reference code (format: COMP-{id}-INT-{timestamp}), creates PurchaseRequest record with status 'pending'
- **Files**: `controllers/PurchaseController.js`, `services/PurchaseService.js`, `models/PurchaseRequest.js`
- **Requirement**: FR-008, FR-016
- **User Story**: P1 User Story 5, Scenario 2

#### Subtask 2.2: Implement Admin Purchase Approval Workflow

- **Description**: Create admin/finance interface to review pending purchase requests, verify payment receipt, approve/reject with quota credit and balance deduction
- **Acceptance**: Admin can view list of pending purchases, upload/attach payment receipt, click [Approve] to credit interviews to company, deduct from balance, update purchase status to 'completed', create BillingTransaction
- **Files**: `controllers/AdminPurchaseController.js`, `services/PurchaseService.js`
- **Requirement**: FR-017
- **User Story**: P1 User Story 5, Scenario 3

#### Subtask 2.3: Implement Bulk Purchase Support

- **Description**: Extend purchase request handler to support bulk quantities (1-50 interviews), calculate total cost, generate single invoice
- **Acceptance**: Handler accepts quantity parameter, calculates total = quantity × RM15, validates balance ≥ total, creates single BillingTransaction with interview_count_charged = quantity
- **Files**: `services/PurchaseService.js`
- **Requirement**: FR-025

#### Subtask 2.4: Implement Individual Quota Counter Management

- **Description**: Create methods to manage individual_interviews_remaining counter: increment on purchase approval, decrement on consumption, separate from plan quota
- **Acceptance**: Methods maintain separate counter in CompanySubscription entity, provides getter for total_available_quota = subscription_remaining + individual_remaining - reserved
- **Files**: `services/SubscriptionService.js`
- **Requirement**: FR-009
- **User Story**: P1 User Story 5, Scenario 4

---

### Main Task 3: Subscription Upgrade/Downgrade Processing

**Description**: Implement backend logic for mid-year subscription changes with prorated charge calculations and quota adjustments.

**Requirement Reference**: FR-013, FR-014, FR-015

#### Subtask 3.1: Create Prorated Charge Calculator

- **Description**: Build utility function to calculate prorated charges/refunds for subscription changes based on remaining days in subscription year
- **Acceptance**: Function `calculateProration(currentPlanPrice, newPlanPrice, daysRemaining)` returns prorated_amount = (newPrice - currentPrice) / 365 × daysRemaining, handles upgrades (positive) and downgrades (negative)
- **Files**: `utils/prorationCalculator.js`
- **Requirement**: FR-013, FR-014

#### Subtask 3.2: Implement Subscription Upgrade Handler

- **Description**: Create handler to process subscription upgrades with prorated charge calculation, balance deduction, immediate quota increase
- **Acceptance**: Handler validates new tier > current tier, calculates prorated charge, deducts from balance (or creates invoice), updates subscription_plan_id, resets interviews_remaining_this_year to new plan's annual quota, creates BillingTransaction (type: subscription_upgrade)
- **Files**: `controllers/SubscriptionController.js`, `services/SubscriptionService.js`
- **Requirement**: FR-013
- **User Story**: P1 User Story 5 (quota exhaustion option 1)

#### Subtask 3.3: Implement Subscription Downgrade Handler

- **Description**: Create handler to process subscription downgrades with prorated refund calculation, quota reduction, usage validation
- **Acceptance**: Handler validates new tier < current tier, calculates prorated refund, checks if current usage > new plan quota (if yes, block new invitations), updates subscription_plan_id, adjusts interviews_remaining_this_year, credits balance if refund
- **Files**: `controllers/SubscriptionController.js`, `services/SubscriptionService.js`
- **Requirement**: FR-014

#### Subtask 3.4: Implement Subscription Cancellation Handler

- **Description**: Create handler to process subscription cancellations with prorated refund, service end date calculation, status update
- **Acceptance**: Handler calculates prorated refund for unused period, sets status='cancelled', service_end_date=current_anniversary_date, creates refund transaction, sends confirmation email
- **Files**: `controllers/SubscriptionController.js`, `services/SubscriptionService.js`
- **Requirement**: FR-015

---

### Main Task 4: Billing & Transaction Management

**Description**: Implement billing transaction recording, audit trail, invoice generation, and payment processing backend.

**Requirement Reference**: FR-004, FR-005, FR-024, FR-026

#### Subtask 4.1: Create Billing Transaction Service

- **Description**: Build service class for creating, querying, and managing billing transactions with transaction types: annual_subscription, individual_purchase, balance_topup, interview_settlement
- **Acceptance**: Service provides methods: `createTransaction()`, `getCompanyTransactions()`, `generateInvoiceNumber()`, validates required fields, creates audit trail
- **Files**: `services/BillingService.js`, `models/BillingTransaction.js`
- **Requirement**: FR-024

#### Subtask 4.2: Implement Interview Completion Billing

- **Description**: Create method to record billing transaction when interview completes, linking to interview_id, recording actual cost, quota source
- **Acceptance**: Method `recordInterviewBilling(interviewId, companyId, actualCost, quotaSource)` creates BillingTransaction with type='interview_settlement', stores interview_cost, quota_consumed_from, creates InterviewCompletion record
- **Files**: `services/BillingService.js`
- **Requirement**: FR-024

#### Subtask 4.3: Implement Invoice Number Generation

- **Description**: Create function to generate unique, sequential invoice numbers with format: INV-YYYY-MM-{sequence}
- **Acceptance**: Function generates unique invoice numbers, increments sequence per month, zero-padded sequence (e.g., INV-2026-01-0001), stores in BillingTransaction.invoice_number
- **Files**: `utils/invoiceNumberGenerator.js`
- **Requirement**: FR-004

#### Subtask 4.4: Implement Billing History Query Service

- **Description**: Create service methods to query billing history with filtering by date range and transaction type, pagination support
- **Acceptance**: Method `getBillingHistory(companyId, filters)` returns paginated results, supports date range filter, transaction type filter, sorts by date descending, includes invoice_url links
- **Files**: `services/BillingService.js`
- **Requirement**: FR-004, FR-006

---

### Main Task 5: Usage Analytics & Reporting

**Description**: Implement backend services for aggregating interview usage data, calculating metrics, and providing chart data endpoints.

**User Story Reference**: P1 User Story 2 - View Interview Usage History & Analytics

#### Subtask 5.1: Create Usage Aggregation Service

- **Description**: Build service to aggregate interview completion data by date (daily/monthly) with date range filtering
- **Acceptance**: Service method `getUsageTimeSeries(companyId, granularity, startDate, endDate)` returns array of {date, count} objects, supports 'daily' and 'monthly' granularity, filters by date range
- **Files**: `services/UsageAnalyticsService.js`
- **User Story**: P1 User Story 2, Scenarios 1-2

#### Subtask 5.2: Implement Usage Breakdown Calculator

- **Description**: Create method to calculate interview usage breakdown by status (completed, pending, cancelled) with percentage calculations
- **Acceptance**: Method `getUsageBreakdown(companyId, startDate, endDate)` returns {completed: {count, percentage}, pending: {count, percentage}, cancelled: {count, percentage}}, percentages sum to 100%
- **Files**: `services/UsageAnalyticsService.js`
- **User Story**: P1 User Story 2, Scenario 3

#### Subtask 5.3: Implement Quota Depletion Estimator

- **Description**: Create method to estimate quota depletion date based on rolling 7-day average usage rate
- **Acceptance**: Method `estimateDepletionDate(companyId)` calculates rolling 7-day average interview completions, divides remaining_quota by average, returns estimated date (±1 day acceptable), returns null if usage rate = 0
- **Files**: `services/UsageAnalyticsService.js`
- **Requirement**: FR-022

#### Subtask 5.4: Implement Combined Chart Data Endpoint

- **Description**: Create endpoint to return combined data for dual-axis chart (line + bar) in single response
- **Acceptance**: Endpoint `/api/usage/combined-chart` returns {timeSeries: [], breakdown: []} in single response, reduces client API calls from 2 to 1
- **Files**: `controllers/UsageController.js`, `services/UsageAnalyticsService.js`
- **User Story**: P1 User Story 2, Scenario 4

---

### Main Task 6: Notification System

**Description**: Implement notification triggers, delivery logic, and duplicate prevention for quota alerts, balance alerts, and payment confirmations.

**Requirement Reference**: FR-011, FR-012, FR-018

**User Story Reference**: P2 User Story 4 - Receive Interview Quota Alerts

#### Subtask 6.1: Create Notification Trigger Service

- **Description**: Build service to check quota and balance thresholds, trigger notifications when crossed, prevent duplicates using NotificationLog
- **Acceptance**: Service method `checkAndTriggerAlerts(companyId)` calculates usage percentage, checks if 80% or 100% threshold crossed AND not already notified this subscription year, creates notification records
- **Files**: `services/NotificationService.js`, `models/NotificationLog.js`
- **Requirement**: FR-011, FR-012
- **User Story**: P2 User Story 4, Scenarios 1-2, 4-5

#### Subtask 6.2: Implement Email Notification Sender

- **Description**: Create email sending service with templates for quota alerts (80%, 100%), balance alerts, purchase confirmations, upgrade confirmations
- **Acceptance**: Service sends emails to all users with admin flag for the company, uses email templates with dynamic data, logs sent emails in NotificationLog, integrates with email provider (SendGrid/AWS SES)
- **Files**: `services/EmailService.js`, `templates/email-quota-alert-80.html`, `templates/email-quota-alert-100.html`, `templates/email-balance-alert.html`
- **Requirement**: FR-011, FR-018
- **User Story**: P2 User Story 4, Scenarios 1, 3, 6

#### Subtask 6.3: Implement In-App Banner Notification System

- **Description**: Create service to generate in-app notification banners/modals, store in notification queue, mark as displayed/acknowledged
- **Acceptance**: Service creates notification records with type='in_app_banner', provides API endpoint to fetch unacknowledged notifications for current user, supports dismiss action
- **Files**: `services/NotificationService.js`, `controllers/NotificationController.js`
- **User Story**: P2 User Story 4, Scenario 2

#### Subtask 6.4: Implement Low Balance Alert Trigger

- **Description**: Create method to detect when account balance drops below RM50 threshold, trigger low balance notifications
- **Acceptance**: Method `checkBalanceThreshold(companyId)` checks if available_balance < RM50 AND has not been notified since last balance increase above threshold, sends email + in-app notification with [Top Up Now] link
- **Files**: `services/NotificationService.js`
- **User Story**: P2 User Story 4, Scenarios 6-8

#### Subtask 6.5: Implement Notification Flag Reset on Anniversary

- **Description**: Create scheduled job to reset notification flags (80%, 100% quota alerts) on subscription anniversary date for new billing year
- **Acceptance**: Job runs daily, finds companies with subscription_anniversary_date = today, resets notification flags for quota thresholds, allows re-triggering in new year
- **Files**: `jobs/NotificationFlagReset.js`, `services/NotificationService.js`
- **User Story**: P2 User Story 4, Scenario 5

---

### Main Task 7: Account Balance Top-Up Processing

**Description**: Implement backend logic for balance top-up via bank transfer with admin approval workflow.

**Requirement Reference**: FR-031

#### Subtask 7.1: Create Top-Up Request Handler

- **Description**: Build API handler to create top-up request record with amount, payment instructions, unique reference code
- **Acceptance**: Handler validates amount ≥ RM50 (minimum), generates unique reference code (format: COMP-{id}-BAL-{timestamp}), creates TopUpRequest record with status 'pending', returns payment instructions
- **Files**: `controllers/TopUpController.js`, `services/TopUpService.js`, `models/TopUpRequest.js`
- **Requirement**: FR-031

#### Subtask 7.2: Implement Admin Top-Up Approval Workflow

- **Description**: Create admin interface to review pending top-up requests, verify payment receipt, approve/reject with balance credit
- **Acceptance**: Admin can view list of pending top-ups, upload payment receipt, click [Approve] to credit balance to company account, update top-up status to 'completed', create BillingTransaction (type: balance_topup)
- **Files**: `controllers/AdminTopUpController.js`, `services/TopUpService.js`
- **Requirement**: FR-031

#### Subtask 7.3: Implement Balance Credit Logic

- **Description**: Create method to credit top-up amount to company account balance, create billing transaction, send confirmation email
- **Acceptance**: Method `creditBalance(companyId, amount)` increments account_balance_rmb, creates BillingTransaction with type='balance_topup', triggers email notification with new balance
- **Files**: `services/AccountBalanceService.js`
- **Requirement**: FR-031

---

## Database

### Main Task 1: Core Subscription Schema Design

**Description**: Design and implement database schema for subscription plans, company subscriptions, and quota tracking.

**Entities**: SubscriptionPlan, CompanySubscription, QuotaReservation, InterviewCompletion

#### Subtask 1.1: Create SubscriptionPlan Table

- **Description**: Create table to store the three predefined subscription plans
- **Columns**:
  - `id` (PK, INT)
  - `name` (VARCHAR, unique: "Gold Fish", "Dolphin", "Whale")
  - `annual_price_rmb` (DECIMAL: 3600, 7200, 12000)
  - `annual_interview_quota` (INT: 300, 800, 2000)
  - `cost_per_interview_rmb` (DECIMAL: 12, 9, 6)
  - `description` (TEXT)
  - `features` (JSON array)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_name` on name
- **Acceptance**: Migration runs successfully, seeds 3 initial plans, unique constraint on name enforced

#### Subtask 1.2: Create CompanySubscription Table

- **Description**: Create table linking companies to their subscriptions with quota tracking
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies, unique)
  - `subscription_plan_id` (FK to subscription_plans)
  - `status` (ENUM: active, suspended, cancelled)
  - `subscription_start_date` (DATE)
  - `subscription_anniversary_date` (DATE)
  - `interviews_remaining_this_year` (INT, default: plan's annual_quota)
  - `individual_interviews_purchased` (INT, default: 0)
  - `individual_interviews_remaining` (INT, default: 0)
  - `reserved_interviews_count` (INT, default: 0)
  - `account_balance_rmb` (DECIMAL, default: 0)
  - `last_quota_reset_date` (DATE)
  - `service_end_date` (DATE, nullable)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_id` on company_id, `idx_status` on status, `idx_anniversary_date` on subscription_anniversary_date
- **Used By**: Backend Task 1.1 (SubscriptionService), Backend Task 1.2 (QuotaReservationService), Backend Task 1.6 (AccountBalanceService)
- **Tested By**: Testing Task 1.1 (Subscription Service Unit Tests)
- **Acceptance**: Migration runs successfully, foreign key constraints enforced, unique constraint on company_id (one subscription per company)
- **Requirement**: FR-001, FR-030

#### Subtask 1.3: Create QuotaReservation Table

- **Description**: Create table to track reserved interview quota for pending invitations
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `interview_id` (FK to interviews, nullable)
  - `reserved_interview_count` (INT, default: 1)
  - `reserved_from` (ENUM: plan_quota, individual_purchase)
  - `estimated_cost_rmb` (DECIMAL, nullable)
  - `reserved_at` (TIMESTAMP)
  - `expires_at` (TIMESTAMP, calculated: reserved_at + 3 days)
  - `status` (ENUM: active, consumed, released, cancelled)
  - `consumed_at` (TIMESTAMP, nullable)
  - `released_at` (TIMESTAMP, nullable)
  - `billing_transaction_id` (FK to billing_transactions, nullable)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_status` on (company_id, status), `idx_expires_at` on expires_at, `idx_interview_id` on interview_id
- **Acceptance**: Migration runs successfully, composite indexes created for query performance, expires_at automatically calculated
- **Requirement**: FR-027, FR-029, FR-030

#### Subtask 1.4: Create InterviewCompletion Table

- **Description**: Create table recording each completed interview for quota consumption audit
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `interview_id` (FK to interviews)
  - `candidate_id` (FK to candidates)
  - `interview_completed_at` (TIMESTAMP)
  - `quota_consumed_from` (ENUM: plan_quota, individual_purchase)
  - `cost_rmb` (DECIMAL)
  - `billing_transaction_id` (FK to billing_transactions)
  - `created_at` (TIMESTAMP)
- **Indexes**: `idx_company_completed_at` on (company_id, interview_completed_at), `idx_interview_id` on interview_id
- **Acceptance**: Migration runs successfully, foreign key constraints enforced, composite index for time-based queries
- **Requirement**: FR-024, FR-028

---

### Main Task 2: Billing & Transaction Schema Design

**Description**: Design and implement database schema for billing transactions, payment tracking, and invoices.

**Entities**: BillingTransaction, PurchaseRequest, TopUpRequest, PaymentMethod

#### Subtask 2.1: Create BillingTransaction Table

- **Description**: Create table to store all financial transactions with comprehensive audit trail
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `transaction_type` (ENUM: annual_subscription, individual_interview_purchase, balance_topup, interview_settlement, subscription_upgrade, subscription_downgrade, subscription_cancellation)
  - `amount_rmb` (DECIMAL, positive for charges, negative for refunds)
  - `transaction_date` (TIMESTAMP)
  - `description` (TEXT)
  - `subscription_plan_id` (FK to subscription_plans, nullable)
  - `interview_id` (FK to interviews, nullable)
  - `interview_count_charged` (INT, nullable)
  - `payment_method` (ENUM: bank_transfer, credit_card, etc.)
  - `payment_receipt_url` (VARCHAR, nullable)
  - `processed_by_user_id` (FK to users, nullable for admin actions)
  - `status` (ENUM: pending, completed, failed)
  - `invoice_number` (VARCHAR, unique)
  - `invoice_url` (VARCHAR, nullable)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_date` on (company_id, transaction_date), `idx_invoice_number` on invoice_number (unique), `idx_type_status` on (transaction_type, status)
- **Acceptance**: Migration runs successfully, composite indexes for filtering queries, unique constraint on invoice_number
- **Requirement**: FR-004, FR-024
- **User Story**: P1 User Story 3

#### Subtask 2.2: Create PurchaseRequest Table

- **Description**: Create table to track individual interview purchase requests and approval workflow
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `quantity` (INT)
  - `total_cost_rmb` (DECIMAL, calculated: quantity × 15)
  - `reference_code` (VARCHAR, unique)
  - `payment_receipt_url` (VARCHAR, nullable)
  - `status` (ENUM: pending, completed, rejected)
  - `approved_by_user_id` (FK to users, nullable)
  - `approved_at` (TIMESTAMP, nullable)
  - `rejection_reason` (TEXT, nullable)
  - `billing_transaction_id` (FK to billing_transactions, nullable)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_status` on (company_id, status), `idx_reference_code` on reference_code (unique)
- **Acceptance**: Migration runs successfully, unique constraint on reference_code enforced
- **Requirement**: FR-008, FR-017
- **User Story**: P1 User Story 5

#### Subtask 2.3: Create TopUpRequest Table

- **Description**: Create table to track account balance top-up requests and approval workflow
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `amount_rmb` (DECIMAL)
  - `reference_code` (VARCHAR, unique)
  - `payment_receipt_url` (VARCHAR, nullable)
  - `status` (ENUM: pending, completed, rejected)
  - `approved_by_user_id` (FK to users, nullable)
  - `approved_at` (TIMESTAMP, nullable)
  - `rejection_reason` (TEXT, nullable)
  - `billing_transaction_id` (FK to billing_transactions, nullable)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_status` on (company_id, status), `idx_reference_code` on reference_code (unique)
- **Acceptance**: Migration runs successfully, unique constraint on reference_code enforced
- **Requirement**: FR-031

#### Subtask 2.4: Create PaymentMethod Table

- **Description**: Create table to store company payment method details (encrypted)
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `type` (ENUM: bank_transfer, credit_card, etc.)
  - `is_active` (BOOLEAN, default: true)
  - `payment_details` (TEXT, encrypted JSON)
  - `created_at`, `updated_at` (TIMESTAMP)
- **Indexes**: `idx_company_active` on (company_id, is_active)
- **Acceptance**: Migration runs successfully, payment_details field uses encryption at rest

---

### Main Task 3: Notification Schema Design

**Description**: Design and implement database schema for notification logging and duplicate prevention.

**Entities**: NotificationLog

#### Subtask 3.1: Create NotificationLog Table

- **Description**: Create table to log all sent notifications and prevent duplicates
- **Columns**:
  - `id` (PK, INT)
  - `company_id` (FK to companies)
  - `notification_type` (ENUM: email, in_app_banner, in_app_modal)
  - `trigger_event` (ENUM: quota_80_percent, quota_100_percent, balance_low, purchase_confirmation, subscription_anniversary, subscription_upgrade, subscription_downgrade, subscription_cancellation)
  - `sent_at` (TIMESTAMP)
  - `acknowledged_at` (TIMESTAMP, nullable, for in-app notifications)
  - `subscription_year` (VARCHAR, format: "2025-2026", for tracking quota alert duplicates)
  - `message_content` (TEXT, nullable)
  - `recipient_user_ids` (JSON array of user IDs who received this notification)
  - `created_at` (TIMESTAMP)
- **Indexes**: `idx_company_trigger_year` on (company_id, trigger_event, subscription_year), `idx_acknowledged` on acknowledged_at
- **Acceptance**: Migration runs successfully, composite index for duplicate detection queries, supports querying unacknowledged notifications
- **Requirement**: FR-011, FR-012
- **User Story**: P2 User Story 4

---

### Main Task 4: Database Constraints & Relationships

**Description**: Define foreign key constraints, cascading rules, and referential integrity across all tables.

#### Subtask 4.1: Define Foreign Key Constraints

- **Description**: Add foreign key constraints with appropriate cascading rules for all relationships
- **Constraints**:
  - CompanySubscription.company_id → companies.id (ON DELETE CASCADE)
  - CompanySubscription.subscription_plan_id → subscription_plans.id (ON DELETE RESTRICT)
  - QuotaReservation.company_id → companies.id (ON DELETE CASCADE)
  - QuotaReservation.interview_id → interviews.id (ON DELETE SET NULL)
  - QuotaReservation.billing_transaction_id → billing_transactions.id (ON DELETE SET NULL)
  - InterviewCompletion.company_id → companies.id (ON DELETE CASCADE)
  - InterviewCompletion.interview_id → interviews.id (ON DELETE RESTRICT)
  - InterviewCompletion.billing_transaction_id → billing_transactions.id (ON DELETE RESTRICT)
  - BillingTransaction.company_id → companies.id (ON DELETE CASCADE)
  - BillingTransaction.subscription_plan_id → subscription_plans.id (ON DELETE SET NULL)
  - BillingTransaction.interview_id → interviews.id (ON DELETE SET NULL)
  - PurchaseRequest.company_id → companies.id (ON DELETE CASCADE)
  - TopUpRequest.company_id → companies.id (ON DELETE CASCADE)
  - NotificationLog.company_id → companies.id (ON DELETE CASCADE)
- **Acceptance**: All foreign key constraints created, cascading rules tested (deleting company cascades to related records), RESTRICT rules prevent invalid deletions

#### Subtask 4.2: Add Check Constraints for Business Rules

- **Description**: Add database-level check constraints to enforce business rules
- **Constraints**:
  - CompanySubscription.interviews_remaining_this_year ≥ 0
  - CompanySubscription.individual_interviews_remaining ≥ 0
  - CompanySubscription.reserved_interviews_count ≥ 0
  - CompanySubscription.account_balance_rmb ≥ 0 (or allow negative with overdraft logic)
  - BillingTransaction.amount_rmb ≠ 0
  - PurchaseRequest.quantity > 0
  - TopUpRequest.amount_rmb ≥ 50 (minimum top-up)
- **Acceptance**: Check constraints enforced at database level, INSERT/UPDATE violations raise errors

---

### Main Task 5: Database Indexes & Performance Optimization

**Description**: Create additional indexes for query performance optimization based on expected access patterns.

#### Subtask 5.1: Create Composite Indexes for Common Queries

- **Description**: Add composite indexes to optimize frequently used query patterns
- **Indexes**:
  - `idx_interview_completion_company_date` on InterviewCompletion(company_id, interview_completed_at) for usage analytics
  - `idx_billing_company_type_date` on BillingTransaction(company_id, transaction_type, transaction_date) for billing history filtering
  - `idx_quota_reservation_expiry` on QuotaReservation(expires_at, status) for expired reservation cleanup job
  - `idx_notification_company_acknowledged` on NotificationLog(company_id, acknowledged_at) for unread notification queries
- **Acceptance**: Indexes created successfully, query execution plans show index usage, query performance improvement measurable (< 2s for 3 years of data)

#### Subtask 5.2: Create Partial Indexes for Active Records

- **Description**: Add partial indexes for frequently queried active records to reduce index size
- **Indexes**:
  - `idx_active_subscriptions` on CompanySubscription(company_id) WHERE status = 'active'
  - `idx_active_reservations` on QuotaReservation(company_id, expires_at) WHERE status = 'active'
  - `idx_pending_purchases` on PurchaseRequest(company_id) WHERE status = 'pending'
- **Acceptance**: Partial indexes created, significantly smaller than full indexes, query planner uses them for filtered queries

---

### Main Task 6: Data Seeding & Initial Setup

**Description**: Create database seed scripts for initial data population (subscription plans, test companies).

#### Subtask 6.1: Seed Subscription Plans

- **Description**: Create seed script to populate SubscriptionPlan table with three predefined plans
- **Data**:
  - Gold Fish: RM3600/year, 300 interviews, RM12 per interview
  - Dolphin: RM7200/year, 800 interviews, RM9 per interview
  - Whale: RM12000/year, 2000 interviews, RM6 per interview
- **Acceptance**: Seed script is idempotent (can run multiple times safely), inserts 3 plans if not exist, includes feature arrays for each plan
- **Files**: `seeds/001_subscription_plans.sql`

#### Subtask 6.2: Create Test Company Subscriptions (Development Only)

- **Description**: Create seed script to set up test company subscriptions for development/testing
- **Data**: Create 5 test companies with different subscription tiers, various usage levels (20%, 50%, 85%, 100%), different balance amounts
- **Acceptance**: Seed script only runs in development/test environments, creates realistic test data for all user stories
- **Files**: `seeds/dev_002_test_subscriptions.sql`

---

## Billing & Payments

### Main Task 1: Invoice PDF Generation

**Description**: Implement PDF invoice generation using template engine and PDF library.

**Requirement Reference**: FR-005, FR-026

#### Subtask 1.1: Set Up PDF Generation Library

- **Description**: Install and configure PDF generation library (Puppeteer/wkhtmltopdf/PDFKit)
- **Acceptance**: Library installed, basic "Hello World" PDF generation works, supports HTML/CSS templates
- **Files**: `package.json`, `config/pdf.js`

#### Subtask 1.2: Create Invoice HTML Template

- **Description**: Design HTML/CSS template for invoice with company branding, itemized charges, totals
- **Acceptance**: Template renders correctly in browser, includes all required fields (company name, invoice number, date, items table, subtotal, total, payment info), responsive layout for A4 paper size
- **Files**: `templates/invoice-template.html`, `styles/invoice.css`
- **User Story**: P1 User Story 3, Scenario 3

#### Subtask 1.3: Implement PDF Generation Service

- **Description**: Create service to generate PDF from HTML template with dynamic data injection
- **Acceptance**: Service method `generateInvoicePDF(transactionId)` fetches transaction data, injects into template, generates PDF, stores in cloud storage, returns download URL
- **Files**: `services/PDFGenerationService.js`
- **Requirement**: FR-005

#### Subtask 1.4: Implement PDF Caching Strategy

- **Description**: Cache generated PDFs to avoid regenerating on every download request
- **Acceptance**: First download generates and caches PDF, subsequent downloads serve cached version, cache key = invoice_number, TTL = indefinite (invoices immutable)
- **Files**: `services/PDFGenerationService.js`, `utils/cacheManager.js`

#### Subtask 1.5: Add Quota Breakdown to Subscription Invoices

- **Description**: Extend invoice template to show quota allocation, usage, remaining quota for subscription invoices
- **Acceptance**: Subscription invoices include section: "Annual Allocation: 300 interviews", "Remaining: 288 interviews", link to purchase individual interviews
- **Files**: `templates/invoice-template.html`
- **Requirement**: FR-026

---

### Main Task 2: Bank Transfer Payment Processing

**Description**: Implement manual bank transfer payment processing workflow for finance team.

**Requirement Reference**: FR-016, FR-017, FR-031

#### Subtask 2.1: Generate Unique Reference Codes

- **Description**: Create utility to generate unique payment reference codes for tracking bank transfers
- **Acceptance**: Function `generateReferenceCode(companyId, type)` generates format: COMP-{id}-{type}-{timestamp}, where type = 'INT' (individual) or 'BAL' (balance), guaranteed unique
- **Files**: `utils/referenceCodeGenerator.js`
- **Requirement**: FR-016

#### Subtask 2.2: Create Payment Instructions Generator

- **Description**: Build service to generate bank transfer instructions with bank details and reference code
- **Acceptance**: Service method `generatePaymentInstructions(companyId, amount, type)` returns object with bank name, account number, account name, amount, reference code, submission instructions
- **Files**: `services/PaymentInstructionService.js`
- **Requirement**: FR-016

#### Subtask 2.3: Implement Receipt Upload Handler

- **Description**: Create file upload handler for payment receipt submission with validation and storage
- **Acceptance**: Handler validates file type (PDF, JPG, PNG), max size 5MB, uploads to cloud storage (S3/Azure Blob), stores URL in PurchaseRequest/TopUpRequest, returns success response
- **Files**: `controllers/ReceiptUploadController.js`, `services/FileStorageService.js`
- **Requirement**: FR-016

#### Subtask 2.4: Create Admin Payment Verification Interface (Backend)

- **Description**: Build backend endpoints for admin to view pending payments, approve/reject with notes
- **Acceptance**: Endpoints support: (1) GET /admin/payments/pending (list all pending purchase/top-up requests), (2) POST /admin/payments/:id/approve (approve and credit), (3) POST /admin/payments/:id/reject (reject with reason)
- **Files**: `controllers/AdminPaymentController.js`, `services/PaymentVerificationService.js`
- **Requirement**: FR-017

#### Subtask 2.5: Implement Payment Approval Webhook (Future Enhancement)

- **Description**: Prepare webhook endpoint for future automated payment verification via bank API integration
- **Acceptance**: Endpoint receives payment notification from bank, validates signature, matches reference code, auto-approves payment, credits quota/balance
- **Files**: `controllers/PaymentWebhookController.js`
- **Note**: Implementation deferred pending bank API access, endpoint stubbed for future use

---

## Notifications

### Main Task 1: Email Notification System

**Description**: Implement email sending infrastructure with templates for quota alerts, balance alerts, and payment confirmations.

**Requirement Reference**: FR-011, FR-018

**User Story Reference**: P2 User Story 4 - Receive Interview Quota Alerts

#### Subtask 1.1: Set Up Email Service Provider Integration

- **Description**: Integrate with email service provider (SendGrid/AWS SES) and configure API credentials
- **Acceptance**: Email provider API configured, test email sends successfully, credentials stored securely in environment variables
- **Files**: `config/email.js`, `.env`

#### Subtask 1.2: Create Email Template Engine

- **Description**: Set up template engine (Handlebars/EJS) for dynamic email content generation
- **Acceptance**: Template engine renders dynamic variables, supports partials (header, footer), preview in browser during development
- **Files**: `services/EmailTemplateService.js`, `templates/emails/layout.hbs`

#### Subtask 1.3: Create Quota Alert Email Templates

- **Description**: Design email templates for 80% and 100% quota alerts with clear messaging and CTAs
- **Acceptance**: Templates include: (1) quota-alert-80.html with warning message, remaining count, [Upgrade Now] button, (2) quota-alert-100.html with urgent message, options to upgrade/purchase, both mobile-responsive
- **Files**: `templates/emails/quota-alert-80.hbs`, `templates/emails/quota-alert-100.hbs`
- **User Story**: P2 User Story 4, Scenarios 1, 3

#### Subtask 1.4: Create Balance Alert Email Template

- **Description**: Design email template for low balance alert (< RM50) with top-up CTA
- **Acceptance**: Template includes current balance, minimum required, [Top Up Now] button, explanation of balance requirement, mobile-responsive
- **Files**: `templates/emails/balance-alert.hbs`
- **User Story**: P2 User Story 4, Scenario 6

#### Subtask 1.5: Create Payment Confirmation Email Templates

- **Description**: Design email templates for purchase confirmations (individual interviews, balance top-ups, subscription changes)
- **Acceptance**: Templates include: (1) purchase-confirmation.html with quantity, cost, new quota, (2) topup-confirmation.html with amount, new balance, (3) subscription-change.html with new tier, quota, all with invoice PDF attachment links
- **Files**: `templates/emails/purchase-confirmation.hbs`, `templates/emails/topup-confirmation.hbs`, `templates/emails/subscription-change.hbs`
- **Requirement**: FR-018

#### Subtask 1.6: Implement Batch Email Sending for Company Admins

- **Description**: Create method to send emails to all admin users for a company in single batch
- **Acceptance**: Method `sendToCompanyAdmins(companyId, templateName, data)` queries users with admin flag, sends email to all, logs each send in NotificationLog, handles individual send failures gracefully
- **Files**: `services/EmailService.js`
- **Requirement**: FR-011 (clarification: send to all admins)

---

### Main Task 2: In-App Notification System

**Description**: Implement in-app banner and modal notification system for real-time alerts.

**User Story Reference**: P2 User Story 4 - Receive Interview Quota Alerts

#### Subtask 2.1: Create Notification Queue Service

- **Description**: Build service to create in-app notification records for display in UI
- **Acceptance**: Service method `createInAppNotification(companyId, triggerEvent, message)` creates NotificationLog record with type='in_app_banner', status='unacknowledged', returns notification ID
- **Files**: `services/InAppNotificationService.js`
- **User Story**: P2 User Story 4, Scenario 2

#### Subtask 2.2: Implement Notification Fetch Endpoint

- **Description**: Create API endpoint to fetch unacknowledged in-app notifications for current user
- **Acceptance**: Endpoint GET /api/notifications/unread returns array of notifications, filters by user's company_id, includes notification type, message, created_at, notification_id
- **Files**: `controllers/NotificationController.js`
- **User Story**: P2 User Story 4, Scenario 2

#### Subtask 2.3: Implement Notification Acknowledgment Endpoint

- **Description**: Create API endpoint to mark notification as acknowledged/dismissed
- **Acceptance**: Endpoint POST /api/notifications/:id/acknowledge updates acknowledged_at timestamp, returns success, notification won't appear in future /unread calls
- **Files**: `controllers/NotificationController.js`

#### Subtask 2.4: Create WebSocket/Polling for Real-Time Notifications (Optional Enhancement)

- **Description**: Implement real-time notification push via WebSocket or server-sent events
- **Acceptance**: When new notification created, push to connected client immediately, fallback to polling every 60s if WebSocket unavailable
- **Files**: `services/WebSocketService.js`, `sockets/notificationSocket.js`
- **Note**: Optional enhancement, can start with polling-based refresh

---

### Main Task 3: Scheduled Notification Jobs

**Description**: Implement background jobs to check thresholds and trigger notifications.

**Requirement Reference**: FR-011, FR-012

**User Story Reference**: P2 User Story 4 - Receive Interview Quota Alerts

#### Subtask 3.1: Create Quota Threshold Checker Job

- **Description**: Build scheduled job to check all companies' quota usage and trigger 80%/100% alerts
- **Acceptance**: Job runs every 15 minutes, queries all active subscriptions, calculates usage percentage, triggers notifications if threshold crossed AND not already notified this subscription year, logs job execution
- **Files**: `jobs/QuotaThresholdChecker.js`, `services/NotificationService.js`
- **Requirement**: FR-011, FR-012
- **User Story**: P2 User Story 4, Scenarios 1, 3, 4

#### Subtask 3.2: Create Balance Threshold Checker Job

- **Description**: Build scheduled job to check all companies' account balance and trigger low balance alerts (< RM50)
- **Acceptance**: Job runs every 15 minutes, queries all companies, checks available_balance < 50, triggers notification if not already notified since last balance increase above threshold, logs job execution
- **Files**: `jobs/BalanceThresholdChecker.js`, `services/NotificationService.js`
- **User Story**: P2 User Story 4, Scenarios 6, 7, 8

#### Subtask 3.3: Create Notification Flag Reset Job

- **Description**: Build scheduled job to reset notification flags on subscription anniversary dates
- **Acceptance**: Job runs daily at midnight company timezone, finds companies with subscription_anniversary_date = today, deletes old subscription_year notification records OR updates flag to allow retriggering
- **Files**: `jobs/NotificationFlagReset.js`
- **User Story**: P2 User Story 4, Scenario 5

#### Subtask 3.4: Implement Job Scheduler Configuration

- **Description**: Configure job scheduler (node-cron/Bull/Agenda) to run notification jobs at specified intervals
- **Acceptance**: Jobs registered with scheduler, run at correct intervals (15min for checkers, daily for reset), logs visible in monitoring dashboard, failures retry with exponential backoff
- **Files**: `config/scheduler.js`, `jobs/index.js`

---

## Testing

### Main Task 1: Unit Tests for Core Services

**Description**: Write comprehensive unit tests for subscription, quota, billing, and notification services.

#### Subtask 1.1: Test Subscription Service Methods

- **Description**: Write unit tests for subscription management methods (status retrieval, quota calculations, availability checks)
- **Test Cases**:
  - Test `getSubscriptionStatus()` returns correct data
  - Test `calculateRemainingQuota()` with various usage scenarios
  - Test `checkQuotaAvailability()` with exhausted/available quota
  - Test `determineQuotaSource()` prioritizes plan quota over individual
- **Coverage Target**: > 90% line coverage
- **Files**: `tests/unit/services/SubscriptionService.test.js`

#### Subtask 1.2: Test Quota Reservation & Settlement Logic

- **Description**: Write unit tests for quota reservation, settlement, and release methods
- **Test Cases**:
  - Test `reserveInterviewQuota()` with sufficient quota
  - Test reservation blocks when quota exhausted
  - Test reservation blocks when balance insufficient
  - Test `settleInterviewCompletion()` consumes correct quota source
  - Test settlement deducts correct cost from balance
  - Test `releaseReservation()` increments available quota
  - Test mixed quota consumption (plan first, then individual)
- **Coverage Target**: > 95% (critical path)
- **Files**: `tests/unit/services/QuotaReservationService.test.js`

#### Subtask 1.3: Test Billing Service Methods

- **Description**: Write unit tests for billing transaction creation and invoice generation
- **Test Cases**:
  - Test `createTransaction()` with all transaction types
  - Test invoice number generation uniqueness
  - Test `getBillingHistory()` with filters (date range, type)
  - Test `recordInterviewBilling()` creates correct records
- **Coverage Target**: > 90%
- **Files**: `tests/unit/services/BillingService.test.js`

#### Subtask 1.4: Test Notification Service Logic

- **Description**: Write unit tests for notification triggers and duplicate prevention
- **Test Cases**:
  - Test `checkAndTriggerAlerts()` sends at 80% threshold
  - Test `checkAndTriggerAlerts()` sends at 100% threshold
  - Test duplicate prevention (no second 80% notification)
  - Test notification flag reset on anniversary
  - Test low balance alert trigger
  - Test notification retrigger after balance increase/decrease cycle
- **Coverage Target**: > 90%
- **Files**: `tests/unit/services/NotificationService.test.js`

#### Subtask 1.5: Test Prorated Charge Calculator

- **Description**: Write unit tests for prorated charge/refund calculations
- **Test Cases**:
  - Test upgrade prorated charge (Gold → Dolphin with 200 days remaining)
  - Test downgrade prorated refund (Whale → Dolphin with 100 days remaining)
  - Test edge case: 1 day remaining
  - Test edge case: 364 days remaining
  - Test precision (round to 2 decimal places)
- **Coverage Target**: 100% (pure utility function)
- **Files**: `tests/unit/utils/prorationCalculator.test.js`

---

### Main Task 2: Integration Tests for API Endpoints

**Description**: Write integration tests for all API endpoints with database mocking.

#### Subtask 2.1: Test Subscription Management Endpoints

- **Description**: Write integration tests for subscription API endpoints
- **Test Cases**:
  - GET /api/subscriptions/status returns correct data
  - GET /api/subscriptions/plans returns 3 plans
  - POST /api/subscriptions/upgrade succeeds with valid plan
  - POST /api/subscriptions/upgrade fails if insufficient balance
  - POST /api/subscriptions/downgrade succeeds and adjusts quota
  - POST /api/subscriptions/cancel calculates refund correctly
- **Coverage Target**: > 85%
- **Files**: `tests/integration/api/subscription.test.js`

#### Subtask 2.2: Test Quota Management Endpoints

- **Description**: Write integration tests for quota reservation and validation endpoints
- **Test Cases**:
  - POST /api/quota/reserve succeeds with available quota
  - POST /api/quota/reserve fails with exhausted quota
  - POST /api/quota/reserve fails with insufficient balance
  - POST /api/quota/settle consumes quota and deducts balance
  - POST /api/quota/release increments available quota
  - GET /api/quota/validate-invitation returns correct eligibility
- **Coverage Target**: > 85%
- **Files**: `tests/integration/api/quota.test.js`

#### Subtask 2.3: Test Purchase & Top-Up Endpoints

- **Description**: Write integration tests for individual purchase and balance top-up endpoints
- **Test Cases**:
  - POST /api/purchases/create generates reference code
  - POST /api/purchases/:id/approve credits interviews and deducts balance
  - POST /api/purchases/create fails if balance insufficient
  - POST /api/balance/topup generates reference code
  - POST /api/admin/balance/:id/approve credits balance
- **Coverage Target**: > 85%
- **Files**: `tests/integration/api/purchase.test.js`, `tests/integration/api/balance.test.js`

#### Subtask 2.4: Test Billing History Endpoints

- **Description**: Write integration tests for billing history and invoice endpoints
- **Test Cases**:
  - GET /api/billing/history returns paginated results
  - GET /api/billing/history filters by date range correctly
  - GET /api/billing/history filters by transaction type
  - GET /api/billing/invoices/:id returns invoice details
  - GET /api/billing/invoices/:id/download generates PDF
  - GET /api/billing/invoices/:id returns 403 for different company
- **Coverage Target**: > 85%
- **Files**: `tests/integration/api/billing.test.js`

#### Subtask 2.5: Test Usage Analytics Endpoints

- **Description**: Write integration tests for usage analytics data endpoints
- **Test Cases**:
  - GET /api/usage/timeseries returns daily data points
  - GET /api/usage/timeseries returns monthly aggregates
  - GET /api/usage/timeseries filters by date range
  - GET /api/usage/breakdown calculates percentages correctly
  - GET /api/usage/estimate-depletion uses rolling average
- **Coverage Target**: > 85%
- **Files**: `tests/integration/api/usage.test.js`

---

### Main Task 3: End-to-End Tests for User Flows

**Description**: Write E2E tests using Playwright/Cypress for critical user journeys.

#### Subtask 3.1: Test Subscription Dashboard View Flow

- **Description**: E2E test for viewing subscription status and usage (User Story 1)
- **Test Steps**:
  1. Login as Company Admin
  2. Navigate to subscription dashboard
  3. Verify subscription tier displayed
  4. Verify quota counters (used, remaining, reserved, available)
  5. Verify account balance displayed
  6. Verify warning banners appear at correct thresholds
- **Acceptance**: Test passes on all major browsers, completes in < 30s
- **Files**: `tests/e2e/subscription-dashboard.spec.js`

#### Subtask 3.2: Test Usage Analytics Interaction Flow

- **Description**: E2E test for viewing and filtering usage charts (User Story 2)
- **Test Steps**:
  1. Login as Company Admin
  2. Navigate to usage analytics
  3. Verify line graph renders with data
  4. Toggle daily/monthly view
  5. Apply date range filter
  6. Verify pie chart renders with correct percentages
- **Acceptance**: Charts render without errors, filters apply correctly
- **Files**: `tests/e2e/usage-analytics.spec.js`

#### Subtask 3.3: Test Billing History & Invoice Download Flow

- **Description**: E2E test for viewing billing history and downloading invoices (User Story 3)
- **Test Steps**:
  1. Login as Company Admin
  2. Navigate to billing history
  3. Verify table displays transactions
  4. Apply date range filter
  5. Apply transaction type filter
  6. Click download PDF button
  7. Verify PDF downloaded successfully
- **Acceptance**: Table filters work, PDF downloads without errors
- **Files**: `tests/e2e/billing-history.spec.js`

#### Subtask 3.4: Test Individual Interview Purchase Flow

- **Description**: E2E test for purchasing individual interviews (User Story 5)
- **Test Steps**:
  1. Login as Company Admin with exhausted quota
  2. Attempt to send invitation (blocked)
  3. Click [Purchase Interviews]
  4. Select quantity (10)
  5. Verify total cost calculation
  6. Submit purchase request
  7. Verify payment instructions displayed
  8. (Mock) Admin approves purchase
  9. Verify quota credited, balance deducted
  10. Verify confirmation email sent (check logs)
- **Acceptance**: Full purchase flow completes successfully, quota updated
- **Files**: `tests/e2e/individual-purchase.spec.js`

#### Subtask 3.5: Test Quota Exhaustion & Upgrade Flow

- **Description**: E2E test for quota exhaustion blocking and subscription upgrade
- **Test Steps**:
  1. Login as Company Admin with 100% quota used
  2. Attempt to send invitation
  3. Verify blocked with options modal
  4. Click [Upgrade Plan]
  5. Select Dolphin tier
  6. Verify prorated charge displayed
  7. Confirm upgrade
  8. Verify quota immediately increased
  9. Verify able to send invitation now
- **Acceptance**: Upgrade flow works end-to-end, quota increases immediately
- **Files**: `tests/e2e/quota-exhaustion-upgrade.spec.js`

---

### Main Task 4: Performance & Load Testing

**Description**: Conduct performance testing for high-traffic scenarios (quota checks, analytics queries).

#### Subtask 4.1: Load Test Quota Reservation Endpoint

- **Description**: Simulate 1000 concurrent quota reservation requests to measure throughput
- **Test Setup**: Use Apache JMeter or k6, target: 1000 req/min
- **Acceptance**: Average response time < 1s, 99th percentile < 3s, 0% error rate
- **Files**: `tests/performance/quota-reservation.k6.js`

#### Subtask 4.2: Load Test Usage Analytics Queries

- **Description**: Test analytics endpoint performance with 3 years of data
- **Test Setup**: Pre-populate database with 10,000 interview completion records, query with various date ranges
- **Acceptance**: Query returns in < 2s for any 3-year range, index usage confirmed in EXPLAIN plan
- **Files**: `tests/performance/usage-analytics.k6.js`

#### Subtask 4.3: Load Test Notification Threshold Checker Job

- **Description**: Test notification job performance with 10,000 companies
- **Test Setup**: Seed database with 10,000 company subscriptions at various usage levels, run job
- **Acceptance**: Job completes in < 5 minutes, sends correct notifications only (no duplicates)
- **Files**: `tests/performance/notification-job.test.js`

---

## DevOps & Infrastructure

### Main Task 1: Environment Configuration

**Description**: Set up environment variables and configuration management for subscription feature.

#### Subtask 1.1: Define Environment Variables

- **Description**: Document and configure all required environment variables for subscription feature
- **Variables**:
  - `EMAIL_PROVIDER_API_KEY` - SendGrid/AWS SES API key
  - `EMAIL_FROM_ADDRESS` - Sender email address
  - `BANK_ACCOUNT_NUMBER` - Bank account for transfers
  - `BANK_ACCOUNT_NAME` - Account holder name
  - `BANK_NAME` - Bank name for payment instructions
  - `PDF_STORAGE_BUCKET` - Cloud storage bucket for PDFs
  - `INDIVIDUAL_INTERVIEW_PRICE` - Price per interview (RM15)
  - `LOW_BALANCE_THRESHOLD` - Threshold for balance alerts (RM50)
- **Acceptance**: Variables documented in `.env.example`, loaded in application config
- **Files**: `.env.example`, `config/subscription.js`

#### Subtask 1.2: Set Up Cloud Storage for Invoices & Receipts

- **Description**: Configure cloud storage (AWS S3/Azure Blob) for storing PDF invoices and payment receipts
- **Acceptance**: Storage bucket created, access credentials configured, file upload/download tested, public URL generation works
- **Files**: `config/storage.js`, infrastructure scripts

---

### Main Task 2: Background Job Scheduling

**Description**: Set up and configure job scheduler for notification jobs and quota cleanup.

#### Subtask 2.1: Install & Configure Job Scheduler

- **Description**: Install job scheduler library (Bull/Agenda/node-cron) and configure Redis (if using Bull)
- **Acceptance**: Scheduler runs jobs at configured intervals, job logs visible in console, failures trigger alerts
- **Files**: `config/scheduler.js`, `package.json`

#### Subtask 2.2: Register Scheduled Jobs

- **Description**: Register all notification and cleanup jobs with scheduler
- **Jobs**:
  - QuotaThresholdChecker - Every 15 minutes
  - BalanceThresholdChecker - Every 15 minutes
  - ExpiredReservationCleanup - Every hour
  - NotificationFlagReset - Daily at midnight (company timezone)
- **Acceptance**: All jobs registered, run at correct intervals, can be monitored via admin dashboard
- **Files**: `jobs/index.js`

---

### Main Task 3: Monitoring & Alerting

**Description**: Set up monitoring for subscription metrics and alerting for critical failures.

#### Subtask 3.1: Define Subscription Metrics

- **Description**: Define and instrument key metrics for monitoring subscription health
- **Metrics**:
  - Total active subscriptions by tier
  - Average quota usage percentage
  - Total individual interviews purchased (daily)
  - Low balance companies count
  - Quota exhausted companies count
  - Notification delivery success rate
  - Invoice generation success rate
- **Acceptance**: Metrics exported to monitoring system (Prometheus/Datadog), dashboards created
- **Files**: `services/MetricsService.js`

#### Subtask 3.2: Set Up Critical Alerts

- **Description**: Configure alerts for critical subscription failures
- **Alerts**:
  - Notification job fails 3 times consecutively
  - Invoice PDF generation fails
  - Quota reservation fails (availability issues)
  - Email delivery failure rate > 10%
- **Acceptance**: Alerts fire when thresholds breached, notifications sent to on-call team via Slack/PagerDuty

---

## Task Summary

- **Total Main Tasks**: 24 (API tasks integrated into Backend)
- **Total Subtasks**: 141+
- **Priority Distribution**:
  - P1 (MVP): 18 main tasks (Frontend dashboard, Backend quota/billing with API endpoints, Database schema)
  - P2 (Enhancement): 4 main tasks (Notification system, Usage analytics advanced features)
  - P3 (Nice-to-have): 2 main tasks (WebSocket notifications, Performance optimization)

**Cross-Referencing Notes**:

- All Frontend tasks reference Backend APIs they depend on (Uses field)
- All Backend service methods show which endpoints/Frontend call them (Called By field)
- All Backend API endpoints integrated into Backend section (not separate)
- All Database tables show which Backend services use them (Used By field)
- All implementation tasks link to test tasks that validate them (Tested By field)

---

## Dependencies & Sequencing

### Phase 1 (Must complete first - Weeks 1-2):

- Database schema design and migrations (all tables)
- Seed subscription plans
- Core backend services (SubscriptionService, QuotaReservationService)
- Basic API endpoints (subscription status, quota validation)

**Blockers**: All subsequent work depends on database schema

### Phase 2 (Can parallelize - Weeks 3-4):

**Team A - Frontend**:

- Subscription dashboard UI (Task: Frontend Main Task 1, 2)
- Usage analytics charts (Task: Frontend Main Task 2)
- Billing history table (Task: Frontend Main Task 3)

**Team B - Backend**:

- Quota reservation/settlement logic (Task: Backend Main Task 1)
- Individual purchase processing (Task: Backend Main Task 2)
- Billing transaction service (Task: Backend Main Task 4)

**Team C - API**:

- All API endpoints (Tasks: API Main Tasks 1-6)

**Dependencies**: Frontend depends on API endpoints being available

### Phase 3 (Integration - Week 5):

- Individual interview purchase flow (end-to-end integration)
- Subscription upgrade/downgrade flows
- Invoice PDF generation
- Bank transfer payment processing
- Testing (unit, integration, E2E)

**Dependencies**: All Phase 2 work must be complete

### Phase 4 (Enhancements - Week 6):

- Notification system (email + in-app)
- Background jobs (quota alerts, expiry cleanup)
- Account balance top-up feature
- Performance optimization
- Monitoring & alerting

**Dependencies**: Core functionality from Phase 3

---

## Notes

- **Hybrid Quota + Balance Model**: This feature uses a hybrid approach where subscriptions provide annual interview quotas AND companies maintain account balances for pay-as-you-go costs. Quota is reserved on invitation send, actual cost settled on completion.

- **Bank Transfer Payment Processing**: Initial implementation uses manual bank transfer with back-office approval workflow. Credit card integration (Stripe/PayPal) planned for future phase.

- **Anniversary-Based Billing**: All subscription billing operates on anniversary date (subscription start date) rather than calendar months. Quota resets occur on anniversary date each year.

- **Notification Delivery**: All billing notifications sent ONLY to users with admin flag. Regular users do not receive billing emails.

- **Prorated Calculations**: All subscription changes (upgrades, downgrades, cancellations) use prorated calculations based on (annual_price / 365) × days_remaining formula.

- **Quota Consumption Priority**: System ALWAYS consumes plan quota first, then individual purchased interviews only after plan quota is exhausted.

- **Reservation Expiry**: Invitations that are not completed within 3 days have their reserved quota automatically released back to the available pool (hourly cleanup job).

- **Balance Insufficiency Blocking**: Even if company has available quota, system blocks invitation send if account balance is insufficient to cover estimated interview cost.

- **Testing Priority**: Focus E2E testing on critical paths (quota exhaustion flow, purchase flow, upgrade flow) as these directly impact revenue and user experience.
