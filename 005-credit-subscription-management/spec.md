# Feature Specification: Credit & Subscription Management

**Feature Branch**: `005-credit-subscription-management`  
**Created**: December 13, 2025  
**Status**: Draft  
**Input**: User description: "Client-Side Credit Logic - subscription plan viewing, token usage tracking/visualization, billing history, payment methods, usage notifications, self-service subscription management"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - View Subscription Status & Account Balance (Priority: P1)

Company Admins need to monitor their subscription status and prepaid account balance to ensure uninterrupted service and manage costs.

**Why this priority**: Core visibility into subscription status and account balance is fundamental for any pay-per-use SaaS billing system. Without this, users cannot manage their spending or prevent service interruptions.

**Independent Test**: Can be fully tested by logging in as Company Admin, navigating to subscription dashboard, and verifying that subscription status, account balance (monetary), token usage rate, and balance depletion rate are displayed accurately. Delivers immediate value by providing transparency into account state.

**Acceptance Scenarios**:

1. **Given** I am logged in as a Company Admin with an active subscription (subscribed on Dec 1, 2025) and $500 prepaid balance, **When** I navigate to the subscription dashboard, **Then** I see subscription status "Active", subscription start date "Dec 1, 2025", current balance "$500.00", token pricing rate (e.g., "$25 per 1M tokens"), and total tokens consumed to date

2. **Given** I am logged in as a Company Admin and my company has $500 balance with $300 already spent on tokens in December 2025, **When** I view the subscription dashboard on December 13, 2025, **Then** I see "Current Balance: $200.00" and "Total Usage This Month: $300.00 (12M tokens)"

3. **Given** I am logged in as a Company Admin with $50 remaining balance and daily usage averaging $20/day, **When** I view the subscription dashboard, **Then** I see a warning banner "⚠️ Low Balance: $50 remaining. At your current usage rate, balance will be depleted in approximately 2.5 days. [Top Up Now]"

4. **Given** my account balance has reached $0, **When** I navigate to the subscription dashboard, **Then** I see status banner "❌ Service Suspended - Zero Balance", balance shown as "$0.00", and a prominent call-to-action "Add Funds to Resume Service" with [Top Up] button

---

### User Story 2 - View Token Usage History & Analytics (Priority: P1)

Company Admins need to analyze token consumption patterns over time to optimize usage and forecast future needs.

**Why this priority**: Usage analytics are essential for cost management and planning. Without historical data visualization, admins cannot identify trends, anomalies, or optimize their token consumption.

**Independent Test**: Can be fully tested by creating sample token usage records across different dates and purposes, then verifying that the dashboard displays accurate line graphs (daily/monthly views), date range filtering works, and pie charts show correct usage breakdowns by purpose. Delivers standalone value by enabling data-driven usage decisions.

**Acceptance Scenarios**:

1. **Given** my company has token usage data for the past 3 months, **When** I view the token usage chart in "Daily" mode with date range "Dec 1-13, 2025", **Then** I see a line graph with 13 data points showing daily token consumption, with dates on X-axis and token count on Y-axis

2. **Given** my company has token usage data, **When** I switch to "Monthly" view with date range "Oct 2025 - Dec 2025", **Then** I see a line graph with 3 data points showing aggregate monthly consumption

3. **Given** my company used tokens for: 10M for "Interview - Question Generation", 5M for "Interview - Response Scoring", 3M for "Job Description Analysis", **When** I view the usage breakdown pie chart for December 2025, **Then** I see 3 segments: "Interview - Question Generation" (55.6%), "Interview - Response Scoring" (27.8%), "Job Description Analysis" (16.7%)

4. **Given** I have token usage data, **When** I view the combined dual-axis chart, **Then** I see a line graph for usage over time overlaid with a bar chart showing usage by purpose for the selected period

5. **Given** I select an invalid date range (end date before start date), **When** I apply the filter, **Then** I see an error message "Invalid date range: End date must be after start date" and the chart retains the previous valid filter

---

### User Story 3 - View Billing History & Invoices (Priority: P1)

Company Admins need to access past invoices, receipts, and payment history for accounting, auditing, and expense tracking purposes.

**Why this priority**: Billing transparency and invoice access are mandatory for any B2B SaaS product. Finance departments require downloadable invoices for bookkeeping and tax compliance.

**Independent Test**: Can be fully tested by creating sample billing records with various statuses and dates, then verifying that the billing history table displays correctly, filters work (date range, payment status), and PDF downloads generate proper invoices. Delivers immediate value for financial record-keeping.

**Acceptance Scenarios**:

1. **Given** my company has 6 months of billing history, **When** I navigate to the billing history page, **Then** I see a table with columns: Invoice Number, Date, Description, Amount, Status (Paid/Unpaid), and Actions (View/Download PDF)

2. **Given** I have billing records from Jan 2025 to Dec 2025, **When** I filter by date range "Oct 2025 - Dec 2025" and status "Paid", **Then** I see only invoices from October, November, and December 2025 with status "Paid"

3. **Given** I select an invoice from the billing history table, **When** I click "Download PDF", **Then** a PDF invoice is generated and downloaded containing: company name, invoice number, date, itemized charges (subscription fee, extra usage charges if any), total amount, payment method, and payment status

4. **Given** I have an unpaid invoice from November 2025, **When** I view the billing history, **Then** the unpaid invoice row is highlighted with a badge "Unpaid" and shows a "Pay Now" action button

5. **Given** I have no billing history (new account), **When** I navigate to the billing history page, **Then** I see an empty state message "No billing history available" with information about when the first invoice will be generated

---

### User Story 4 - Receive Low Balance Notifications (Priority: P2)

Company Admins need automated alerts when token consumption reaches critical thresholds (80%, 100%) to take timely action.

**Why this priority**: Automated notifications prevent service disruptions and unexpected charges. While not MVP-critical (users can manually check dashboard), it significantly improves user experience and reduces support burden.

**Independent Test**: Can be fully tested by simulating token usage that crosses 80% and 100% thresholds, then verifying that email notifications are sent and in-app banners/modals appear with correct messaging. Delivers value by enabling proactive account management.

**Acceptance Scenarios**:

1. **Given** I am subscribed to a plan with 20M tokens/month and have used exactly 16M tokens (80%), **When** the system calculates remaining credits, **Then** I receive an email notification with subject "Token Usage Alert: 80% Limit Reached" and body detailing current usage, remaining credits, and link to upgrade plan

2. **Given** I am subscribed to a plan with 20M tokens/month and have used exactly 16M tokens (80%), **When** I next log in to the system, **Then** I see an in-app banner at the top of the dashboard: "⚠️ You have used 80% of your token credits (16M/20M). Remaining: 4M tokens."

3. **Given** my company has used exactly 20M tokens (100%) of the allocated 20M, **When** the system calculates remaining credits, **Then** I receive an email notification with subject "Token Usage Alert: 100% Limit Reached - Overage Charges Apply" and body explaining overage pricing and upgrade options

4. **Given** my company has used exactly 20M tokens (100%), **When** I log in to the system, **Then** I see a modal dialog (cannot be dismissed until acknowledged): "🚨 You have used 100% of your token credits. Further usage will incur extra charges at $30 per 1M tokens. [View Upgrade Options] [Acknowledge]"

5. **Given** I have already received a 80% notification, **When** my usage increases from 16M to 17M (still under 100%), **Then** I do NOT receive duplicate 80% notifications (notification sent once per threshold per billing cycle)

6. **Given** I have crossed 100% usage threshold, **When** a new billing cycle starts and my credits reset, **Then** the notification flags are reset and I can receive new threshold notifications for the new cycle

---

### User Story 5 - Temporary Payment Method (Manual Bank Transfer) (Priority: P2)

For the initial release, Company Admins can top up account balance via bank transfer by sending payment receipt, which Super Admin/Finance team manually updates in the back office.

**Why this priority**: This is a temporary workaround until full payment gateway integration is complete. It's P2 because basic top-up functionality is needed to allow companies to add funds, but it's not the ideal long-term solution.

**Independent Test**: Can be fully tested by initiating a top-up request, making a bank transfer, uploading a payment receipt via designated channel (email), and verifying that back-office admin can credit the account balance and the system reflects the updated balance. Delivers value by enabling companies to fund their accounts during MVP phase.

**Acceptance Scenarios**:

1. **Given** I need to add funds to my account, **When** I click "Top Up" on the dashboard, **Then** I see payment instructions: "Bank Transfer Details: Bank Name: [BANK], Account Number: [ACCOUNT], Reference: [COMPANY_ID]-TOPUP. After payment, email receipt to billing@company.com with amount and company name"

2. **Given** I have made a bank transfer for $500 and emailed the receipt to billing@company.com, **When** the finance team receives the receipt, **Then** they can access the back-office admin panel, locate my company account, upload the receipt, and credit $500 to my account balance

3. **Given** my top-up was processed by the finance team, **When** I refresh my dashboard, **Then** I see my account balance updated to reflect the new funds (e.g., $50 → $550), and the top-up transaction appears in my billing history

4. **Given** the finance team credited my account, **When** I check my email, **Then** I receive a payment confirmation email with subject "Account Top-Up Confirmed: $500 Added" containing company name, amount, new balance, and receipt attachment

---

### Edge Cases

- What happens when balance reaches $0 mid-interview? (CLARIFIED: Block further token consumption immediately. Interview cannot proceed without available balance. User must top up to continue.)
- How does the system handle concurrent top-up transactions? (Queue transactions, process sequentially, prevent race conditions on balance updates)
- What happens if a bank transfer top-up fails or is reversed after being credited? (Back-office admin manually deducts the amount, system logs adjustment, notify company admin)
- How are notifications handled for companies with multiple admins? (CLARIFIED: Send to all users with admin flag, future enhancement: notification preferences)
- What happens when a company cancels their subscription? (Set status to "Cancelled", remaining balance handling per Clarification #14 TBD, disable future token consumption)
- How to handle date range filters that span multiple months for usage analytics? (Show aggregated data across all selected months with monthly breakdown markers in charts)
- What happens if a Company Admin tries to download a PDF receipt/invoice still being generated? (Disable download button until generation complete, show "Processing..." status)
- How are timezone differences handled for usage tracking and transaction timestamps? (Use company's configured timezone for all billing operations, display times in that timezone)
- What happens if token pricing rate changes during active subscription? (New rate applies to future usage only, historical usage retains original rate, notify admins of rate change)
- How is low-balance threshold calculated if daily usage varies significantly? (Use rolling 7-day average usage to estimate depletion date, update estimate daily)

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST display subscription status (active/suspended/cancelled), subscription start date, and token pricing rate (e.g., "$25 per 1M tokens") on the subscription dashboard
- **FR-002**: System MUST display current account balance (prepaid funds remaining in USD) and calculate in real-time by subtracting total usage charges from total deposits
- **FR-003**: System MUST provide token usage visualization via line graphs with toggleable daily/monthly views and date range filtering
- **FR-004**: System MUST provide token usage breakdown by purpose (e.g., interview question generation, response scoring, job description analysis) in pie chart format
- **FR-005**: System MUST display current account balance (prepaid funds remaining) and calculate deductions based on token usage at the defined rate per million tokens
- **FR-006**: System MUST maintain billing history with records for all invoices and payments, including prorated charges for subscription changes
- **FR-007**: System MUST generate downloadable PDF invoices containing: company name, invoice number, date, itemized charges, total amount, payment method, and payment status
- **FR-008**: System MUST support billing history filtering by date range and transaction type (top-up/usage charge)
- **FR-009**: System MUST send email notifications when account balance falls below the defined low-balance threshold amount (TBD), with notifications sent only once until balance is topped up above threshold
- **FR-010**: System MUST display in-app banner alerts when balance is below low-balance threshold and modal alerts when balance reaches zero
- **FR-011**: System MUST allow Company Admins to initiate account top-ups via designated payment method (bank transfer in Phase 1)
- **FR-012**: System MUST allow Company Admins (non-enterprise) to self-service downgrade plans, generating prorated invoices and warning about usage-over-limit scenarios
- **FR-013**: System MUST allow Company Admins (non-enterprise) to cancel subscriptions, with service continuing until the end of the current billing period
- **FR-014**: System MUST allow Company Admins (non-enterprise) to change billing cycle (monthly ↔ yearly), applying prorated charges and showing savings calculations
- **FR-015**: System MUST prevent self-service subscription modifications unless an active payment method is linked to the account
- **FR-016**: System MUST disable self-service subscription management for Enterprise plan customers, directing them to contact account managers
- **FR-017**: System MUST provide bank transfer payment instructions on unpaid invoices, including bank details and unique reference number
- **FR-018**: System MUST allow back-office admins to manually mark invoices as paid and upload payment receipts for bank transfer payments
- **FR-019**: System MUST send payment confirmation emails when invoices are marked as paid by back-office admins
- **FR-020**: System MUST prevent duplicate threshold notifications within the same billing cycle (track notification state per company per cycle)
- **FR-021**: System MUST reset notification flags at the start of each new billing cycle
- **FR-022**: System MUST lock subscription modifications during active processing and display "Change in progress" status
- **FR-023**: System MUST block all token consumption when account balance reaches zero or remaining tokens reach zero, preventing any service usage until funds/credits are replenished
- **FR-024**: System MUST display prominent warnings when remaining balance falls below the defined low-balance threshold (amount TBD)
- **FR-025**: System MUST allow Company Admins to initiate account top-ups (prepaid fund deposits) via the designated payment method
- **FR-026**: System MUST deduct from account balance in real-time as tokens are consumed, updating balance immediately after each token usage event
- **FR-027**: System MUST send billing notifications (invoices, payment confirmations, low-balance alerts) ONLY to users with admin flag, not all company users
- **FR-028**: System MUST calculate billing cycles on anniversary basis (subscription start date to same date next period), not calendar month alignment
- **FR-029**: System MUST use company's configured timezone for all billing operations, dashboard displays, and notification scheduling
- **FR-030**: System MUST support dual-axis combination charts showing both usage over time (line) and usage by purpose (bar/area) on the same visualization

### Key Entities

- **SubscriptionPlan**: Defines the single pay-per-use plan with attributes: name (e.g., "Pay-As-You-Go"), description, token_price_per_million (rate charged per 1M tokens), currency (USD), features (JSON/array of included features like API access, support level, etc.)

- **CompanySubscription**: Links companies to their subscription with attributes: company_id (FK), subscription_plan_id (FK), status (active/suspended/cancelled), subscription_start_date, subscription_anniversary_date (annual renewal date), service_end_date (nullable, for cancelled subscriptions), created_at, updated_at

- **TokenUsage**: Records individual token consumption events (already exists per user input) with expected attributes: id, company_id (FK), user_id (FK - who triggered the usage), purpose (enum: interview_question_generation, interview_response_scoring, job_description_analysis, resume_parsing, etc.), token_count, timestamp, metadata (JSON - interview_id, job_id, etc. for audit trail)

- **BillingTransaction**: Stores all financial transactions (top-ups and usage charges) with attributes: id, company_id (FK), transaction_type (top_up/usage_charge), amount (positive for top-ups, negative for usage), balance_before, balance_after, transaction_date, description (e.g., "Top-up via bank transfer", "Token usage: 1.5M tokens @ $25/1M"), payment_method_id (FK, nullable for usage charges), payment_receipt_url (for bank transfers), processed_by_user_id (FK, for back-office credits), created_at, updated_at

- **PaymentMethod**: Stores payment method references (NOT card details) with attributes: id, company_id (FK), type (credit_card/bank_transfer/other), is_active (boolean), payment_gateway_token (tokenized reference from Stripe/PayPal/etc.), card_last_four (for display only), card_expiry_date, billing_address, created_at, updated_at

- **AccountBalance**: Tracks prepaid account balance for post-pay model with attributes: id, company_id (FK), current_balance (monetary amount), currency (fixed: USD), last_top_up_date, last_top_up_amount, total_lifetime_deposits, total_lifetime_usage, created_at, updated_at

- **TopUpTransaction**: Records all account top-up (deposit) events with attributes: id, company_id (FK), amount, payment_method_id (FK), transaction_date, status (pending/completed/failed), payment_gateway_transaction_id, initiated_by_user_id (FK), created_at, updated_at

- **NotificationLog**: Tracks sent notifications to prevent duplicates with attributes: id, company_id (FK), notification_type (email/in_app_banner/in_app_modal), trigger_event (low_balance/zero_balance), sent_at, acknowledged_at (for modals), created_at

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Company Admins can view subscription status and account balance in under 2 clicks from the main dashboard, with page load time under 2 seconds
- **SC-002**: Account balance calculations are accurate to the cent when compared to sum of all top-up and usage transactions
- **SC-003**: Usage charge calculations are 100% accurate and match the formula: (tokens consumed / 1,000,000) × token_price_per_million
- **SC-004**: Email notifications for low-balance and zero-balance alerts are delivered within 5 minutes of threshold being crossed
- **SC-005**: In-app notifications appear immediately upon user login after low-balance/zero-balance threshold is crossed (no delay beyond page load time)
- **SC-006**: Duplicate notifications for the same threshold within the same billing cycle occur in 0% of cases
- **SC-007**: PDF receipts/transaction records are generated within 10 seconds of request, with 100% accuracy in data (no missing/incorrect amounts)
- **SC-008**: Billing history page loads and displays up to 12 months of transactions in under 3 seconds, with pagination for older records
- **SC-009**: Date range filtering on usage charts returns results in under 2 seconds for up to 1 year of data
- **SC-010**: Token consumption is blocked within 1 second when account balance reaches zero, with 100% enforcement (no unauthorized usage)
- **SC-011**: Manual top-up processing (temporary bank transfer workflow) is completed within 1 business day from receipt submission to account balance credit
- **SC-012**: Account balance updates reflect within 5 seconds of token usage events for real-time accuracy
- **SC-015**: Chart visualizations (line graphs, pie charts, dual-axis charts) render correctly across all major browsers (Chrome, Firefox, Safari, Edge) with no data display errors

---

## Clarifications & Open Questions

### Answered Clarifications

0. **Subscription Model**: ✅ **CLARIFIED** - System has only ONE subscription plan (pay-as-you-go). All tokens are charged at a fixed rate per million tokens. NO free token allocation is provided with the subscription. Companies must maintain prepaid balance to use the service.

1. **Payment Gateway Integration Timeline**: When will the permanent credit card payment gateway be integrated to replace the temporary bank transfer workflow? Which gateway (Stripe, PayPal, Square)? [NEEDS CLARIFICATION]

2. **PCI Compliance**: ✅ **CLARIFIED** - Credit card details will NOT be stored in our database. Will use tokenized references from payment gateway. This reduces PCI compliance requirements significantly.

3. **Enterprise Plan Handling**: What constitutes an "Enterprise plan"? Is this determined by a flag in subscription_plan table, or by contract value/custom terms? Who manages Enterprise subscription changes (dedicated account manager, sales team)? [NEEDS CLARIFICATION]

4. **Prorating Rules**: Confirm prorating calculation method - is it daily proration using (price / days_in_month × days_in_partial_period), or calendar-day-based (price / days_in_billing_cycle × days_used)? [NEEDS CLARIFICATION]

5. **Billing Notification Recipients**: ✅ **CLARIFIED** - Only users with admin flag (Company Admins) will receive billing notifications and invoices. Regular users will not receive billing emails.

6. **Billing Cycle Start Date**: ✅ **CLARIFIED** - Billing cycles are anniversary-based (subscription date to same date next month/year), NOT calendar month aligned. Example: Subscribe on Dec 13 → billing cycle is Dec 13 to Jan 13.

7. **Low Balance Alerts**: ✅ **CLARIFIED** - System will alert/send email when remaining account balance (monetary, not token count) falls below a specific threshold amount. [Threshold amount TBD - requires stakeholder input on notification trigger level, e.g., $50 remaining]

8. **Token Overuse Policy**: ✅ **CLARIFIED** - System will NOT allow token usage when account balance reaches zero. Token consumption is BLOCKED immediately at $0 balance until funds are added. This is a hard stop with no overage charges or credit extension.

9. **Billing Model - Prepaid/Top-Up**: ✅ **CLARIFIED** - System uses POST-PAY model with prepaid top-up functionality. Users deposit funds (e.g., $100 top-up), system deducts from account balance based on actual token usage. Balance decreases as tokens are consumed.

10. **Multi-Currency Support**: ✅ **CLARIFIED** - Multi-currency is NOT supported. All billing, invoices, and payments will be in a single currency (likely USD).

### Remaining Open Questions

11. **Low Balance Alert Threshold**: What is the specific remaining balance threshold that triggers low-balance alerts? (e.g., $50 remaining, or 10% of average monthly usage, or 5M tokens remaining?) [NEEDS CLARIFICATION]

12. **Top-Up Mechanism**: How do users initiate top-ups? Is it manual (user clicks "Add Funds" and makes payment), automatic (auto-recharge when balance hits threshold), or both options available? [NEEDS CLARIFICATION]

13. **Minimum Top-Up Amount**: Is there a minimum top-up amount (e.g., minimum $50 deposit)? Maximum top-up amount? [NEEDS CLARIFICATION]

14. **Unused Balance Handling**: If a user cancels their subscription, what happens to unused prepaid balance? (Refunded, rolled over to next billing cycle, forfeited?) [NEEDS CLARIFICATION]

15. **Token Pricing Model**: How is token pricing calculated for the single pay-as-you-go plan? Fixed flat rate per million tokens regardless of volume, or tiered volume-based pricing (e.g., first 10M at $25/M, next 10M at $20/M, etc.)? [NEEDS CLARIFICATION]

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
