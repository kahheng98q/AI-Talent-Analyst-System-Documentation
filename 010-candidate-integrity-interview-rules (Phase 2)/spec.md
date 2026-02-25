# Feature Specification: Candidate Integrity & Interview Rules

**Feature Branch**: `010-candidate-integrity-interview-rules`  
**Created**: January 28, 2026  
**Status**: Draft  
**Input**: User description: "Anti-cheating system with tab-switching detection, copy/paste prevention, question time limits, and session integrity auditing to ensure fair and authentic candidate evaluation"

> **Admin Portal Status**: Not Implemented in `ai-talent-analyst-system-admin-portal`. These rules apply to the candidate-facing chatbot app (`atas-chatbot`) and have not been built yet. HR-facing integrity reporting (US4) is also not built.

## User Scenarios & Testing _(mandatory)_

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE.
-->

### User Story 1 - Tab Switch Detection & Prevention (Priority: P1) `❌ Not Implemented`

As the Interview System, I must detect and log when candidates navigate away from the interview tab during active questions, so that HR managers can assess interview authenticity and the system can enforce integrity rules.

**Why this priority**: Critical for preventing candidates from searching for answers during the interview. Without this, candidates can freely access external resources, making scores unreliable and unfair to honest candidates.

**Independent Test**: Can be fully tested by triggering `visibilitychange` events (simulating tab switches) during a live interview session and verifying that: (1) violations are logged in the database, (2) warnings appear on candidate's screen upon return, (3) interview is blocked after threshold exceeded, and (4) HR sees integrity report in results. Delivers immediate value by enabling detection of cheating behavior.

**Acceptance Scenarios**:

1. **Given** a candidate is actively answering Question 3 of their interview, **When** they switch to another browser tab (e.g., to Google search), **Then** the system detects the `visibilitychange` event, logs a `CandidateIntegrityEvent` record with type "TAB_SWITCH_OUT", timestamps the event, and increments the violation counter for this session.

2. **Given** a candidate has switched away from the interview tab, **When** they return to the interview tab within 30 seconds, **Then** the system displays a warning banner: "⚠️ Tab switching detected. Repeated violations may result in interview termination." and logs a "TAB_SWITCH_RETURN" event.

3. **Given** a candidate has already triggered 2 tab-switch violations during their interview, **When** they switch tabs a 3rd time, **Then** the system immediately terminates the interview session, displays message "Interview Terminated - Integrity Violation Detected. Your HR manager has been notified.", sets interview status to "TERMINATED_INTEGRITY", and prevents further answer submission.

4. **Given** a candidate switches tabs multiple times within a 10-second window (e.g., accidental clicks), **When** the system evaluates violations, **Then** it treats rapid consecutive switches (< 10 seconds apart) as a single violation to avoid false positives from accidental navigation.

5. **Given** an interview session has tab-switch violations, **When** the HR manager views the candidate's results page, **Then** they see an "Integrity Report" section displaying: total violations count, timestamps of each violation, duration away from tab, and an overall integrity score (e.g., "High Risk" if > 2 violations).

---

### User Story 2 - Disable Copy/Paste Operations (Priority: P1) `❌ Not Implemented`

As the Interview System, I must prevent candidates from copying interview questions or pasting pre-written answers, so that responses represent authentic, real-time thinking rather than prepared content or AI-generated text.

**Why this priority**: Prevents candidates from copying questions to external AI tools (ChatGPT, Claude) or pasting pre-written responses. Essential for maintaining evaluation authenticity and preventing widespread cheating via LLM assistance.

**Independent Test**: Can be fully tested by attempting to copy question text or paste content into answer fields during an active interview and verifying that: (1) operations are blocked at the browser level, (2) violation events are logged, (3) user sees feedback message, and (4) HR can review copy/paste attempt history. Delivers standalone value by preventing a major cheating vector.

**Acceptance Scenarios**:

1. **Given** a candidate is viewing an interview question, **When** they attempt to select and copy the question text using keyboard shortcut (Ctrl+C / Cmd+C) or right-click menu, **Then** the system prevents the copy operation, displays a toast notification "Copy disabled during interview for integrity purposes", and logs a `CandidateIntegrityEvent` with type "COPY_ATTEMPT".

2. **Given** a candidate has copied text from an external source, **When** they attempt to paste it into the answer textarea using keyboard shortcut (Ctrl+V / Cmd+V) or right-click menu, **Then** the system blocks the paste operation, displays message "Paste disabled - answers must be typed manually", and logs a "PASTE_ATTEMPT" event.

3. **Given** a candidate is typing their answer, **When** they attempt to use browser autocomplete or text prediction features (where available), **Then** the system allows native autocomplete (for words, not full sentences) but disables third-party browser extensions that inject content.

4. **Given** a candidate has triggered 5+ copy or paste attempts during their interview, **When** the interview completes, **Then** the results page shows an "⚠️ High Copy/Paste Activity" warning badge visible to the HR manager with count and timestamps of attempts.

5. **Given** an HR manager is reviewing a candidate's interview results, **When** they expand the Integrity Report, **Then** they see a detailed log of copy/paste attempts including: timestamp, question number during which attempt occurred, and attempt type (copy vs. paste).

---

### User Story 3 - Question Time Limits & Auto-Submission (Priority: P1) `❌ Not Implemented`

As the Interview System, I must enforce countdown timers for each interview question and automatically submit answers when time expires, so that all candidates are evaluated under identical time constraints and cannot spend unlimited time crafting perfect answers.

**Why this priority**: Core fairness mechanism. Without time limits, candidates can spend hours researching answers, making scores incomparable across candidates. Time pressure reveals true competency and reduces opportunity for external assistance.

**Independent Test**: Can be fully tested by starting an interview with configured time limits (e.g., 3 minutes per question), waiting for countdown to reach zero, and verifying that: (1) answer is auto-submitted, (2) candidate cannot edit after expiry, (3) time-exceeded status is logged, (4) next question loads automatically, and (5) timer display updates correctly. Delivers standalone value by standardizing evaluation conditions.

**Acceptance Scenarios**:

1. **Given** an HR manager has configured Question 2 with a 3-minute time limit during job description setup, **When** a candidate starts answering Question 2, **Then** the system starts a backend countdown timer at 180 seconds and displays a frontend timer showing "03:00" that counts down in real-time.

2. **Given** a candidate is typing their answer with 30 seconds remaining on the countdown, **When** the timer reaches 00:30, **Then** the timer display changes color to amber/yellow as a visual warning, and at 00:10, it changes to red to indicate imminent expiry.

3. **Given** a candidate is answering a timed question, **When** the countdown reaches 00:00, **Then** the system immediately auto-submits the current answer content (even if incomplete), disables the textarea, displays message "Time's up! Your answer has been submitted.", logs a `QuestionResponse` record with `time_exceeded=True`, and automatically navigates to the next question after 2 seconds.

4. **Given** a candidate has completed and submitted their answer before the time limit expires, **When** they click "Next Question", **Then** the system stops the countdown timer for that question, logs the remaining time (e.g., "Completed with 45 seconds remaining"), and proceeds to the next question without penalty.

5. **Given** a candidate's browser loses internet connectivity during a timed question, **When** connection is restored, **Then** the system recalculates the elapsed time based on server-side timer (not client-side), adjusts the displayed countdown to reflect true remaining time, and continues normally if time remains, or auto-submits if time expired during disconnection.

6. **Given** an HR manager is reviewing candidate results, **When** they view responses to timed questions, **Then** each response displays a time indicator: "✓ Completed in 2:15 (45s remaining)" for early submissions or "⚠️ Time Expired - Auto-submitted" for exceeded time limits.

7. **Given** a candidate closes the interview tab or navigates away during a timed question, **When** they return to the interview, **Then** the countdown timer has continued running server-side, and if time remains, they can continue answering; if time expired while away, the question is marked as "Not Answered - Time Expired" and they proceed to the next question.

---

### User Story 4 - Session Integrity Audit & Reporting (Priority: P1) `❌ Not Implemented`

As an HR Manager, I want to view a comprehensive integrity report for each candidate's interview session, so that I can assess the reliability of their scores and make informed hiring decisions when integrity violations are detected.

**Why this priority**: Without visibility into integrity violations, HR cannot distinguish between authentic high-performers and sophisticated cheaters. This report is the "output" of the previous 3 detection mechanisms and directly impacts hiring quality.

**Independent Test**: Can be fully tested by completing an interview session with various integrity violations (tab switches, copy attempts, time exceedances), then verifying that: (1) all events are aggregated into a unified report, (2) report is accessible from candidate results page, (3) integrity score/risk level is calculated correctly, (4) HR can filter/sort candidates by integrity, and (5) violations are preserved in audit trail. Delivers standalone value by enabling trust assessment of evaluation results.

**Acceptance Scenarios**:

1. **Given** a candidate has completed their interview with 2 tab-switch violations, 1 copy attempt, and 1 time-exceeded question, **When** the HR manager views the candidate's results page, **Then** they see an "Integrity Report" section at the top displaying: overall integrity risk level "⚠️ Medium Risk" (color-coded amber), total violation count "4 violations detected", and an expandable details panel.

2. **Given** an HR manager is viewing a candidate's Integrity Report, **When** they expand the details panel, **Then** they see a chronological table of all integrity events with columns: Timestamp, Question Number, Event Type (Tab Switch / Copy Attempt / Paste Attempt / Time Exceeded), Duration (for tab switches), and Severity (Low/Medium/High).

3. **Given** multiple candidates have completed interviews for the same job, **When** the HR manager views the candidate ranking dashboard, **Then** each candidate row displays an integrity badge: "✓ Clean" (green, 0 violations), "⚠️ Minor Issues" (yellow, 1-2 violations), or "⚠️ High Risk" (red, 3+ violations), and they can sort/filter candidates by integrity score.

4. **Given** a candidate's interview was terminated due to excessive tab-switching (3+ violations), **When** the HR manager views their results, **Then** the Integrity Report displays a prominent "❌ Interview Terminated - Integrity Violation" banner, shows the termination timestamp, lists all violations leading to termination, and marks all scores as "Invalid - Not Evaluated".

5. **Given** an HR manager suspects a candidate cheated based on integrity violations, **When** they click "Export Integrity Report" on the candidate's results page, **Then** the system generates a PDF document containing: candidate name, interview date, job title, full integrity event log with timestamps, screenshots of violation warnings shown to candidate (if available), and overall assessment, suitable for compliance/audit purposes.

6. **Given** a candidate completed the interview with zero integrity violations, **When** the HR manager views their results, **Then** the Integrity Report shows "✓ Clean Session - No Violations Detected", displays a green checkmark badge, and includes metadata: total session duration, average time per question, and confirmation that all integrity checks passed.

7. **Given** an HR manager wants to understand integrity trends across all candidates, **When** they navigate to the Job Analytics page, **Then** they see aggregate integrity statistics: percentage of candidates with clean sessions, most common violation types, and average violations per candidate, helping identify if job settings (e.g., time limits too strict) need adjustment.

---

### Edge Cases

- **Accidental Tab Switches**: Candidate accidentally clicks outside browser briefly (< 5 seconds); system treats consecutive switches within 10 seconds as single violation to avoid false positives.
- **Browser Developer Tools**: Opening browser DevTools (F12) triggers `visibilitychange` on some browsers; system whitelists DevTools-related visibility changes to avoid penalizing candidates troubleshooting technical issues.
- **Mobile Device Interruptions**: Incoming phone call or notification on mobile device triggers visibility change; system provides "Resume Interview" button without penalty if interruption is < 30 seconds.
- **Network Disconnection During Countdown**: Server-side timer continues running during network outage; on reconnection, system reconciles elapsed time and resumes or auto-submits based on actual time remaining.
- **Copy/Paste for Accessibility**: Candidates using screen readers or assistive technologies may require paste for accessibility; system provides HR-configurable exemption flag for ADA compliance (requires pre-approval before interview).
- **Concurrent Violations**: Candidate switches tabs AND attempts copy while timer expires simultaneously; all events logged separately with same timestamp; severity escalates based on total violation count.
- **Interview Restart After Termination**: Candidate terminated for integrity violations cannot restart the same interview session; HR must manually approve re-invitation if justified (new `InterviewSession` created).
- **Zero-Time Limit Questions**: If HR does not configure time limit for a question (unlimited time), no countdown timer displays and no time-exceeded violations can occur for that question.
- **Browser Refresh During Timed Question**: Candidate refreshes browser; session resumes at same question but countdown timer reflects server-side elapsed time; if time expired during refresh, question auto-submits.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST detect `visibilitychange` events on the interview page and log timestamp, event type (out/return), and question context when visibility is lost or regained.

- **FR-002**: System MUST increment a session-level violation counter when candidate switches tabs/windows away from the interview, treating rapid consecutive switches (< 10 seconds apart) as a single violation to avoid false positives.

- **FR-003**: System MUST display a warning banner when candidate returns after tab switch, showing message "⚠️ Tab switching detected. Repeated violations may result in interview termination." that dismisses after 5 seconds.

- **FR-004**: System MUST terminate interview session and block further answer submission when violation threshold is exceeded (default: 3 tab-switch violations), setting interview status to "TERMINATED_INTEGRITY".

- **FR-005**: System MUST disable copy operations (`oncopy` event) and paste operations (`onpaste` event) via JavaScript event handlers on all interview question and answer elements during active interview sessions.

- **FR-006**: System MUST log `CandidateIntegrityEvent` records for all copy/paste attempts with timestamp, question number, event type, and candidate ID, even when operations are successfully blocked.

- **FR-007**: System MUST display toast notifications when copy/paste attempts are blocked: "Copy disabled during interview for integrity purposes" and "Paste disabled - answers must be typed manually".

- **FR-008**: System MUST enforce server-side countdown timers for each timed question, starting when question is displayed and ending when time limit expires or candidate submits answer early.

- **FR-009**: System MUST display real-time countdown timer on frontend (MM:SS format) that syncs with server-side timer and changes color at warning thresholds (amber at 30 seconds, red at 10 seconds remaining).

- **FR-010**: System MUST auto-submit current answer content when countdown reaches zero, disable answer editing, log `time_exceeded=True` on the response record, and automatically navigate to next question after 2-second delay.

- **FR-011**: System MUST handle network disconnections during timed questions by reconciling server-side elapsed time on reconnection and resuming countdown or auto-submitting based on actual remaining time.

- **FR-012**: System MUST aggregate all integrity events (tab switches, copy/paste attempts, time exceedances) into a unified Integrity Report accessible from candidate results page.

- **FR-013**: System MUST calculate overall integrity risk level ("Clean", "Low Risk", "Medium Risk", "High Risk") based on violation count and severity, with thresholds: 0 violations = Clean, 1-2 = Low, 3-5 = Medium, 6+ = High.

- **FR-014**: System MUST display integrity badges on candidate ranking dashboard ("✓ Clean", "⚠️ Minor Issues", "⚠️ High Risk") and allow HR to sort/filter candidates by integrity score.

- **FR-015**: System MUST generate exportable PDF integrity reports containing full event log, timestamps, violation summaries, and overall assessment for audit/compliance purposes.

- **FR-016**: System MUST prevent interview restart after integrity-based termination without HR manual approval; terminated interviews display "Invalid - Not Evaluated" status on all scores.

- **FR-017**: System MUST support HR-configurable time limits per question during job description setup (0 = unlimited, 1-10 minutes recommended) with backend validation to prevent unrealistic limits (< 30 seconds or > 30 minutes).

- **FR-018**: System MUST whitelist browser developer tools (F12) visibility changes to avoid false positives; system detects DevTools context and excludes from violation count.

- **FR-019**: System MUST provide accessibility exemption flag (HR pre-approval required) to allow paste operations for candidates using assistive technologies, logging exemption reason for audit trail.

- **FR-020**: System MUST persist all integrity events indefinitely (aligned with data retention policy) and include them in candidate status history audit logs.

### Key Entities

- **CandidateIntegrityEvent**: Logs individual integrity violations; attributes include `id`, `interview_session_id`, `candidate_id`, `event_type` (enum: TAB_SWITCH_OUT, TAB_SWITCH_RETURN, COPY_ATTEMPT, PASTE_ATTEMPT, TIME_EXCEEDED), `question_number`, `timestamp`, `duration_seconds` (for tab switches), `severity` (Low/Medium/High), `metadata` (JSON for additional context).

- **InterviewSession** (extension): Add fields `integrity_violation_count`, `integrity_risk_level` (enum: CLEAN, LOW, MEDIUM, HIGH), `terminated_for_integrity` (boolean), `integrity_exemption_approved` (boolean for accessibility cases).

- **QuestionResponse** (extension): Add fields `time_limit_seconds`, `time_used_seconds`, `time_exceeded` (boolean), `submitted_method` (enum: MANUAL, AUTO_TIMEOUT, AUTO_TERMINATED).

- **IntegrityReport**: Aggregated view entity (can be computed on-demand or cached); attributes include `interview_session_id`, `total_violations`, `violation_breakdown` (count by type), `risk_level`, `clean_session` (boolean), `termination_reason` (if applicable).

- **JobQuestion** (extension): Add field `time_limit_seconds` to job description question configuration; 0 = unlimited, null = use default (180 seconds), otherwise specific limit.

### Dependencies

- **Feature 001 (AI-HR Interview System)**: Requires integration with existing interview session management, question display logic, and answer submission workflow. Integrity events are logged during the interview execution flow.

- **Feature 003 (Interview Status Tracking)**: Integrity violations and terminations must be recorded in candidate status history audit trail; status transitions (e.g., IN_PROGRESS → TERMINATED_INTEGRITY) follow existing status tracking patterns.

- **Feature 002 (SaaS Admin Portal)**: Integrity Report UI must be accessible from candidate results page in admin portal; HR permission checks required to view detailed violation logs.

- **Database**: New `CandidateIntegrityEvent` table required; schema migrations for `InterviewSession`, `QuestionResponse`, `JobQuestion` extensions.

- **Frontend Framework**: JavaScript event listeners for `visibilitychange`, `oncopy`, `onpaste`; real-time countdown timer using WebSocket or polling for server-side sync.

- **Backend Timer Service**: Server-side countdown tracking using scheduled jobs or in-memory timer management; must handle concurrent sessions and persist elapsed time on interruptions.

### Assumptions

- **Browser Compatibility**: Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+) support `visibilitychange` API and JavaScript event prevention; older browsers may have degraded integrity enforcement.

- **Network Latency**: Timer synchronization accounts for up to 2 seconds of network latency; frontend displays optimistic countdown while backend enforces authoritative time limit.

- **Violation Thresholds**: Default thresholds (3 tab switches = termination, 5 copy/paste = warning) are configurable per company or job via Feature 002 settings; initial deployment uses hardcoded defaults.

- **Accessibility Compliance**: ADA exemptions for paste operations require HR pre-approval workflow (Feature 007); without approval, all candidates subject to identical integrity rules.

- **Mobile Support**: Tab-switching detection works on mobile browsers when switching apps; mobile-specific interruptions (calls, notifications) are handled gracefully with "Resume Interview" flow.

- **Server-Side Authority**: Backend countdown timer is source of truth; client-side timer is for UX only and cannot extend time by manipulation.

- **Audit Retention**: See FR-020 (indefinite retention; no automatic deletion or archival).

- **HR Review Required**: High-risk integrity reports flag candidates for HR review but do NOT automatically disqualify them; HR makes final hiring decision with integrity context.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Zero false positives on accidental browser navigation (< 1% of clean sessions flagged incorrectly) through 10-second consecutive-switch detection.

- **SC-002**: 100% of tab-switch events logged with accurate timestamps (within 1-second precision) and question context.

- **SC-003**: Copy/paste operations blocked in 100% of attempts across supported browsers; all attempts logged for HR visibility.

- **SC-004**: Countdown timers maintain ≤ 2-second drift between server-side authoritative time and frontend display across 5-minute interview sessions.

- **SC-005**: Auto-submission triggered within 1 second of server-side countdown reaching zero; no candidate can submit answers after time expiry.

- **SC-006**: Integrity Report visible on candidate results page within 500ms of page load, aggregating all violations from session.

- **SC-007**: HR can identify high-risk candidates (3+ violations) through visual badges on ranking dashboard; badges display in < 1 second per candidate.

- **SC-008**: Interview termination for excessive violations occurs immediately (< 2 seconds) after threshold exceeded; candidate cannot bypass termination through browser refresh or back button.

- **SC-009**: Integrity events consume < 5KB storage per interview session on average; no performance degradation on results page with 100+ integrity events.

- **SC-010**: Accessibility exemptions (paste enabled) apply correctly to pre-approved candidates without affecting other candidates' integrity enforcement.

- **SC-011**: Integrity Report PDF export generates in < 5 seconds for sessions with up to 50 violation events; PDF includes all required audit information.

- **SC-012**: Network disconnection during timed question reconciles correctly on reconnection; candidate cannot gain extra time through intentional disconnection.

- **SC-013**: System accurately distinguishes browser DevTools visibility changes from genuine tab switches; DevTools usage does NOT increment violation counter.

- **SC-014**: Candidate satisfaction: < 5% of candidates report false termination or unfair integrity enforcement in post-interview surveys (indicates accurate violation detection).

- **SC-015**: HR trust in scores: ≥ 90% of HR managers report increased confidence in evaluation authenticity after reviewing integrity reports (measured via quarterly survey).
