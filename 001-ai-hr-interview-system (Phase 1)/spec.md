# Feature Specification: AI-Powered HR Interview System

**Feature Branch**: `001-ai-hr-interview-system`
**Created**: 2025-11-05
**Status**: In Progress
**Input**: User description: "Generate a detailed specification for an AI-powered HR Interview System that allows HR teams to create job descriptions, schedule interviews, invite candidates, conduct AI-led interviews, and review analytical results. Include features such as question generation, scoring by competency, adaptive questioning, candidate ranking, and PDF reporting."

> **Admin Portal**: `ai-talent-analyst-system-admin-portal` (port 8032) — Company HR admin portal. This is the primary UI for HR users to manage job positions, interview sessions, and candidates.

## Clarifications

### Session 2025-11-07

- Q: How will the system integrate with the core AI model for features like question generation and scoring? → A: Integrate via a third-party API (e.g., OpenAI, Google Gemini).
- Q: What is the data retention policy for candidate data (resumes, interview responses, scores)? → A: See FR-017 (indefinite retention; manual deletion only by HR).
- Q: What is the expected behavior if a candidate loses internet connection during an AI-led interview? → A: The interview session is terminated, and the candidate must restart from the beginning.
- Q: How are the unique interview links secured to prevent unauthorized access or use? → A: Links require candidate authentication (e.g., email verification, password) before accessing the interview.
- Q: How are the "predefined competencies" managed and defined within the system? → A: Competencies are fixed and hardcoded within the system, applied universally to all jobs.

### Implementation Behavior Updates (Verified 2026-02-25)

- **Job Description Form**: HR enters structured fields — Job Title, Min/Max Salary, Role Overview Description, Core Responsibilities (each with Management Level L1–L5, Description, and Result Intended), Qualifications, and Required Skills. There is no file upload for JD. AI generates interview questions from this structured content.
- **Interview Invitation Flow (Changed)**: HR does NOT bulk-invite candidates by email. The actual flow is: (1) Create a Candidate record (name, phone, email, DOB, gender, nationality, country), (2) Create an Interview Session linking that Candidate + Job Position + Scheduled Date/Time, (3) Send the interview link via the "Send Interview Link" button on the session details page. The link is a session-specific token URL.
- **Interview Session Statuses**: `Upcoming` (ID: 4), `In Progress` (ID: 10), `Completed` (ID: 6), `Cancelled` (ID: 8), `Expired` (ID: 11).
- **Scoring System (Changed from generic competencies)**: Total Score = weighted sum: Interview Score 50% + Resume Score 30% + Salary Match Score 20%. Aspect Scores (Knowledge, Experience, Potential, Culture) are each scored /5 and displayed separately.
- **Candidate Ranking**: Completed sessions are listed on the Job Description details page with scores for HR comparison. There is no separate "Rankings" page.
- **Chatbot**: The candidate-facing interview experience (resume upload, AI-led interview Q&A) is handled by a separate application (`atas-chatbot`), not the admin portal.

## User Scenarios & Testing _(mandatory)_

### User Story 1 - HR Creates and Manages a Job Position (Priority: P1) `✅ Implemented`

As an HR manager, I want to create a new job position in the system, including a detailed job description, so that I can start the recruitment process.

**Why this priority**: This is the foundational step for the entire interview process.

**Independent Test**: An HR manager can create a job position, and it appears in the list of active positions.

**Acceptance Scenarios**:

1.  **Given** I am logged in as an HR manager, **When** I navigate to "Job Position" > "Create", **Then** I am presented with a structured form (not a file upload) with fields: Job Title, Min/Max Salary, Role Overview Description, Core Responsibilities (management level L1–L5, description, result intended), Qualifications, and Required Skills.
2.  **Given** I have filled out the job creation form, **When** I click "Confirm", **Then** the new job position is created and I am redirected to the job detail page. The job appears in the Job Position list.

> **Behavior Change**: Original spec described uploading a JD file. Actual implementation uses a structured form. AI generates interview questions from the structured fields.

---

### User Story 2 - HR Updates Job Description Status (Priority: P2) `❌ Not Implemented`

As an HR manager, I want to update the status of a job description (e.g., "Open", "Filled", "On Hold"), so that I can manage the recruitment pipeline effectively.

**Why this priority**: This provides better visibility and control over the recruitment process.

**Independent Test**: An HR manager can change the status of a job, and the new status is reflected in the job list.

**Acceptance Scenarios**:

1.  **Given** I am viewing the list of job positions, **When** I select a job and choose to update its status, **Then** I am presented with a list of possible statuses.
2.  **Given** I have selected a new status, **When** I save the change, **Then** the job's status is updated in the system.

---

### User Story 3 - HR Invites Candidates to an AI-Led Interview (Priority: P1) `🔶 Behavior Changed`

As an HR manager, I want to invite candidates to take an AI-led interview for a specific job position, so that I can efficiently screen a large number of applicants.

**Why this priority**: This is a core feature of the AI-powered system.

**Independent Test**: An HR manager can schedule an interview session for a candidate and send them the unique interview link.

**Acceptance Scenarios**:

1.  **Given** I have a created job position and a candidate record, **When** I navigate to "Sessions" > "Create" and select the Candidate, Job Position, and Scheduled Date/Time, **Then** an interview session is created with status "Upcoming".
2.  **Given** an interview session exists, **When** I open the session details and click "Send Interview Link", **Then** the candidate receives an email with a unique session token URL to access their AI-led interview.

> **Behavior Change**: Original spec described bulk email invite by entering email addresses. Actual implementation requires: (1) pre-existing Candidate record, (2) creating an Interview Session, (3) manually sending the link via "Send Interview Link" button. There is no bulk invite screen.

---

### User Story 4 - Candidate Uploads Resume for AI Assessment (Priority: P1) `✅ Implemented (Chatbot)`

As a candidate, I want to upload my resume after receiving the interview link, so that the system can pre-assess my qualifications against the job description.

> **Note**: This user story is implemented in the `atas-chatbot` application (candidate-facing), not the admin portal.

**Why this priority**: This enhances the screening process by providing an initial data point before the interview.

**Independent Test**: A candidate can upload a resume, and a score appears in the session table.

**Acceptance Scenarios**:

1.  **Given** I have received an interview invitation link, **When** I open the link, **Then** I am prompted to upload my resume.
2.  **Given** I have uploaded my resume, **When** I proceed, **Then** the system analyzes my resume and stores an assessment score.
3.  **Given** my resume has been successfully uploaded, **When** the system is ready, **Then** I am presented with a button to "Start AI Interview".

---

### User Story 5 - Candidate Completes an AI-Led Interview (Priority: P1) `✅ Implemented (Chatbot)`

As a candidate, I want to complete an AI-led interview, so that I can be considered for the job position.

> **Note**: This user story is implemented in the `atas-chatbot` application (candidate-facing), not the admin portal.

**Why this priority**: This is the primary interaction for the candidate.

**Independent Test**: A candidate can open the interview link, answer the questions, and submit the interview.

**Acceptance Scenarios**:

1.  **Given** I have successfully uploaded my resume, **When** I click the "Start AI Interview" button, **Then** I am taken to a page with instructions for the AI-led interview.
2.  **Given** I have started the interview, **When** I answer the questions presented by the AI, **Then** my responses are recorded.
3.  **Given** I have answered all the questions, **When** the interview concludes, **Then** the system will show a prompt message to let me know the interview is completed.

---

### User Story 6 - HR Reviews Interview Results and Rankings (Priority: P2) `✅ Implemented`

As an HR manager, I want to review the results of the AI-led interviews, including competency scores and candidate rankings, so that I can identify the most promising candidates.

**Why this priority**: This feature provides the primary value to the HR team.

**Independent Test**: An HR manager can view completed sessions for a job position and click into a session to see full scoring details.

**Acceptance Scenarios**:

1.  **Given** that candidates have completed their interviews, **When** I navigate to the Job Position details page, **Then** I see a list of completed interview sessions for that job position with scores.
2.  **Given** I am viewing a completed session, **When** I click into the session details, **Then** I can see: Overall Score (0–100), Interview Score (50% weight), Resume Score (30% weight), Salary Match Score (20% weight), Aspect Scores (Knowledge, Experience, Potential, Culture each rated /5), Interview Analysis (Strengths, Development Gaps, Personal Development Plan), and full Chat History of Q&A per aspect.

> **Behavior Change**: There is no dedicated "Rankings" page. Completed sessions are listed on the Job Description details page (Completed Interviews section). The Session List page (`/dashboard/interview-session/list`) also shows all sessions with tabs for Upcoming / In Progress / Completed / Expired / Cancelled. Scoring system: Total Score = Interview Score (50%) + Resume Score (30%) + Salary Match Score (20%). Aspect Scores: Knowledge, Experience, Potential, Culture.

---

### User Story 7 - HR Generates a PDF Report of Interview Results (Priority: P2) `❌ Not Implemented`

As an HR manager, I want to generate a PDF report of the interview results for a candidate, so that I can share it with the hiring team.

**Why this priority**: This allows for easy sharing and offline review of candidate performance.

**Independent Test**: An HR manager can download a PDF report for a specific candidate.

**Acceptance Scenarios**:

1.  **Given** I am viewing a candidate's detailed results, **When** I click the "Download PDF" button, **Then** a PDF file is generated and downloaded to my computer.
2.  **Given** I open the downloaded PDF, **When** I view the content, **Then** it contains the candidate's name, the job position, their overall score, and a breakdown of their competency scores.

---

### User Story 8 - HR Reviews AI Token Usage (Priority: P3) `❌ Not Implemented`

As an HR manager, I want to see how many AI tokens are being consumed by the various AI-powered features, so that I can monitor costs and usage.

**Why this priority**: This is important for cost management and understanding the ROI of the AI features.

**Independent Test**: An HR manager can navigate to a usage dashboard and see a breakdown of token consumption by feature (e.g., interviews, scoring).

**Acceptance Scenarios**:

1.  **Given** I am logged in as an HR manager, **When** I navigate to the "Usage" or "Billing" section, **Then** I can see a summary of AI token usage for the current billing period.
2.  **Given** I am viewing the usage summary, **When** I select a specific feature like "Interviews", **Then** I can see a more detailed breakdown of token consumption for that feature.

### Edge Cases

- If a candidate loses internet connection during the interview, the session is terminated, and the candidate must restart from the beginning.
- How does the system handle candidates who try to take the interview multiple times with the same link?
- What happens if the AI has difficulty understanding a candidate's accent or speech impediment?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: The system MUST allow HR teams to create, edit, and archive job descriptions.
- **FR-002**: The system MUST allow HR teams to invite candidates to interviews via email.
- **FR-003**: The system MUST provide a unique interview link to each candidate, requiring candidate authentication (e.g., email verification, password) before accessing the interview.
- **FR-004**: The system MUST conduct AI-led interviews with candidates, asking a series of questions.
- **FR-005**: The system MUST automatically generate relevant interview questions based on the job description.
- **FR-006**: The system MUST use a moderately complex machine learning model for adaptive questioning, where the difficulty or topic of the next question is based on the candidate's previous answers.
- **FR-007**: The system MUST score candidates based on a fixed set of hardcoded competencies.
- **FR-008**: The system MUST rank candidates based on their overall scores.
- **FR-009**: The system MUST allow HR teams to view detailed interview results, including scores and candidate answers.
- **FR-010**: The system MUST generate PDF reports of interview results.
- **FR-011**: The system MUST ensure the privacy and security of candidate data, complying with both GDPR and CCPA regulations.
- **FR-012**: The system MUST allow candidates to upload their resume in PDF or DOCX format.
- **FR-013**: The system MUST use an AI model to analyze the uploaded resume against the job description and generate a qualification score.
- **FR-014**: The system MUST store the resume assessment score in the interview session table.
- **FR-015**: The system MUST allow HR managers to update the status of a job description (e.g., "Open", "Interviewing", "Filled", "On Hold").
- **FR-016**: The system MUST save the token usage for each AI-powered action (e.g., interview, scoring) and display the aggregated consumption to HR personnel.
- **FR-017**: The system MUST retain candidate data (resumes, interview responses, scores) indefinitely until manually deleted by an HR manager.

### Key Entities _(include if feature involves data)_

- **JobDescription** (`atas.job_descriptions`): `id`, `job_title`, `exp_min_salary`, `exp_max_salary`, `role_overview` (text), `core_responsibilities` (JSONField — array of objects: management level L1–L5, description, result intended), `qualifications` (JSONField — array), `skills_required` (JSONField — array), `company` (FK), `current_version` (FK → `JobDescriptionVersion`), `version_count`, `created_by`, `created_at`, `updated_at`, `deleted_at` (soft-delete). No status field.
- **Candidate** (`atas.candidates`): `id`, `name`, `gender`, `dob`, `email`, `nationality`, `country`, `phone`, `company` (FK), `status_id`, `created_by`, `created_at`, `updated_at`, `deleted_at` (soft-delete).
- **Session** (`atas.sessions`): `id`, `candidate` (FK), `job_description` (FK), `jd_version` (FK — locked at interview start via `jd_locked_at`), `status` (FK → `Status` module=SESSION: Upcoming/InProgress/Completed/Expired/Cancelled), `scheduled_at`, `started_at`, `ended_at`, `interview_score` (float 0–100, weighted from aspect scores), `jd_resume_score` (float), `salary_score` (float), `total_score` (float — Interview 50% + Resume 30% + Salary 20%), `resume_score_analysis` (text), `interview_analysis` (JSONField — `strengths[]`, `development_gaps[]`, `personal_development_plan[]`), `chat_history` (JSONField — per-aspect Q&A with `aspect`, `score` fields), `resume` (FK), `token_hash`, `token_expires_at`, `company` (FK), `created_by`, `created_at`, `updated_at`, `deleted_at` (soft-delete).
- **Resume** (`atas.resumes`): `id`, `candidate` (FK), `job_title`, `skills`, `experience_working`, `experience_title`, `content` (parsed full text), `exp_salary`, `sourcefile_name`, `company` (FK), `status_id`, `created_by`, `created_at`, `updated_at`, `deleted_at` (soft-delete).
- **CompanyAspect** (`atas.company_aspects`): Per-company interview aspect configuration. `company` (FK), `aspect` (FK → `Aspect` — e.g. Knowledge/Experience/Potential/Culture), `aspect_order`, `max_questions`, `max_followups`, `is_active`. Drives which aspects are scored per interview.
- **Status** (`atas.statuses`): Lookup table. `id`, `name`, `code`, `module` (SESSION/JOB_DESCRIPTION/SUBSCRIPTION/TOPUP_REQUEST), `color_hex`, `order`.

> **Not Yet Implemented**: `Question`, `Result`, and `TokenUsage` entities described in the original spec do not have corresponding API models and have not been built.

## Assumptions

- **AI Model Integration**: The system will integrate with a third-party AI provider (e.g., OpenAI, Google Gemini) via API for all AI-powered features, including question generation, resume analysis, and interview scoring.
- **Adaptive Questioning**: The system will use a moderately complex machine learning model for adaptive questioning.
- **Data Privacy**: The system will comply with both GDPR and CCPA data privacy regulations.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Reduce the time-to-hire by 30% by automating the initial screening process.
- **SC-002**: Increase the number of candidates screened by 50% without increasing the HR team's workload.
- **SC-003**: 90% of HR managers report that the AI-powered interviews provide valuable insights into candidate qualifications.
- **SC-004**: The system must be able to handle 100 concurrent interviews without performance degradation.
- **SC-005**: Candidate satisfaction with the interview process, as measured by a post-interview survey, should be at least 4 out of 5.
