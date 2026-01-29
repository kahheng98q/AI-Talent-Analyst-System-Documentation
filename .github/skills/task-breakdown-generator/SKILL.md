---
name: task-breakdown-generator
description: Generate structured task breakdown markdown files from specification documents, organizing main tasks and subtasks by implementation area (Frontend, Backend, API, etc.) with descriptions. Use when users ask to create task lists, development breakdowns, work items, or implementation plans from feature specs, requirements documents, or technical specifications.
license: Complete terms in LICENSE.txt
---

# Task Breakdown Generator

Automatically generates structured markdown files that break down feature specifications into actionable main tasks and subtasks, organized by implementation area (Frontend, Backend, API, etc.).

## When to Use This Skill

- User provides a specification document and wants a task breakdown
- User asks to "create task list", "generate work items", "break down into tasks"
- User needs implementation planning from requirements/specs
- User wants to organize work by team/area (frontend, backend, etc.)

## Task Breakdown Process

### 1. Analyze Source Document

Read the provided specification document (spec.md or similar) to extract:

- **User Stories**: Priority-tagged scenarios (P1, P2, P3)
- **Requirements**: Functional and non-functional needs
- **Entities & Data Models**: Database schemas, API contracts
- **Features**: Core capabilities described in the spec
- **Acceptance Criteria**: Given-When-Then scenarios

### 2. Categorize by Implementation Area

Organize tasks into logical categories based on the work:

**Standard Categories:**

- **Frontend** - UI components, pages, forms, user interactions
- **Backend** - Business logic, data processing, domain models
- **API** - REST/GraphQL endpoints, request/response handling
- **Database** - Schema design, migrations, indexes
- **Authentication/Authorization** - User management, permissions, RBAC
- **Integration** - Third-party services, external APIs
- **DevOps/Infrastructure** - Deployment, monitoring, CI/CD
- **Testing** - Unit tests, integration tests, E2E tests
- **Documentation** - API docs, user guides, deployment guides

**Custom Categories**: Add domain-specific areas as needed (e.g., "AI Model Integration", "Report Generation", "Audit Logging")

### 3. Create Task Hierarchy

**Main Tasks** = High-level feature areas or modules (1-5 days of work)

- Examples: "Candidate Module", "Job Description Module", "Authentication System"
- Each main task represents a cohesive unit of work

**Subtasks** = Specific implementation items under each main task (0.5-2 days of work)

- Examples: "Create candidate list view", "Implement resume upload API", "Design user table schema"
- Each subtask should be independently testable and deployable where possible
- Use when: A main task has 3-5 distinct deliverables

**Sub-Subtasks** = Granular implementation details within complex subtasks (2-8 hours of work)

- Examples: Under "Register Scheduled Jobs", list: "Register QuotaThresholdChecker", "Register BalanceThresholdChecker", etc.
- Use when: A subtask is complex enough to warrant further breakdown (> 1 day of work)
- Improves clarity by explaining what specific work items are included in abstract subtasks

### 4. Write Task Descriptions

For each task and subtask, include:

- **What**: Clear description of the work
- **Why**: Business value or technical necessity (reference spec user story/requirement)
- **Scope**: Key components, files, or areas affected
- **Dependencies**: Other tasks that must complete first (if any)
- **Acceptance**: Brief success criteria (derived from spec Given-When-Then scenarios)

## Output Format

Generate a markdown file with this structure:

```markdown
# Task Breakdown: [Feature Name]

**Source**: [Path to spec file]  
**Generated**: [Date]  
**Status**: Planning

---

## Overview

[Brief summary of the feature and scope of work]

---

## Frontend

### Main Task 1: [Component/Module Name]

**Description**: [What this main task accomplishes]

**User Story Reference**: [P1/P2/P3 User Story from spec]

#### Subtask 1.1: [Specific UI element or page]

- **Description**: [Detailed description of work]
- **Acceptance**: [Success criteria]
- **Files**: [Key files to create/modify]

#### Subtask 1.2: [Another specific element]

- **Description**: [Detailed description]
- **Acceptance**: [Success criteria]
- **Files**: [Key files]

### Main Task 2: [Another Component]

...

---

## Backend

### Main Task 1: [Service/Module Name]

**Description**: [What this accomplishes]

**Requirement Reference**: [Functional requirement from spec]

#### Subtask 1.1: [Specific logic or feature]

- **Description**: [Details]
- **Acceptance**: [Criteria]
- **Dependencies**: [If any]

##### Sub-Subtask 1.1.1: [Granular implementation detail]

- **Description**: [Specific work item]
- **Why**: [Why this is needed]
- **Acceptance**: [Success criteria for this specific item]

##### Sub-Subtask 1.1.2: [Another granularJobs detail]

- **Description**: [Details]
- **Acceptance**: [Criteria]

...

---

## API

### Main Task 1: [Endpoint Group]

**Description**: [Purpose of these endpoints]

#### Subtask 1.1: [Specific endpoint]

- **Description**: POST /api/candidates - Create new candidate
- **Request Body**: [Schema reference]
- **Response**: [Schema reference]
- **Acceptance**: Returns 201 with candidate ID on success

...

---

## Database

### Main Task 1: Schema Design

#### Subtask 1.1: [Entity name] Table

- **Description**: Design and create migration for [entity] table
- **Columns**: [Key columns and relationships]
- **Indexes**: [Performance indexes]
- **Acceptance**: Migration runs successfully in dev/staging

...

---

## [Other Categories as Needed]

---

## Task Summary

- **Total Main Tasks**: [Count]
- **Total Subtasks**: [Count]
- **Priority Distribution**:
  - P1 (MVP): [Count] main tasks
  - P2 (Enhancement): [Count] main tasks
  - P3 (Nice-to-have): [Count] main tasks

---

## Dependencies & Sequencing

1. **Phase 1** (Must complete first):
   - [Task dependencies that block other work]

2. **Phase 2** (Can parallelize):
   - [Independent tasks that can run concurrently]

3. **Phase 3** (Final integration):
   - [Integration and E2E work]

---

## Notes

- [Any clarifications, assumptions, or open questions]
```

## Best Practices

### Task Sizing

- **Main tasks** = 1-5 days of work for a developer
- **Subtasks** = 0.5-2 days of work (story-point size)
- **Sub-Subtasks** = 2-8 hours of work (implementation details)
- If a subtask is larger, break it down into sub-subtasks

### When to Use Sub-Subtasks

Use sub-subtasks (nested level 3) when:

1. **Complex Subtask**: A subtask is too large (> 1 day) and contains multiple distinct items to explain
2. **Abstract/Vague Naming**: The subtask name is abstract and needs concrete examples of what's included
   - ❌ Bad: "Register Scheduled Jobs" (too vague, what jobs?)
   - ✅ Good: Break into sub-subtasks: "Register QuotaThresholdChecker", "Register BalanceThresholdChecker", etc.
3. **Multiple Deliverables**: The subtask involves 3+ discrete items that should be tracked separately
4. **Clarity for Team**: The team won't understand what's included without further breakdown
5. **Configuration/Setup Tasks**: Tasks that involve setting up multiple related systems

Do **NOT** use sub-subtasks when:

- Subtask is already clear and concise (< 1 day of work)
- Breaking it down further adds no value
- The subtask is already granular (e.g., "Create login form")

### Priority Mapping

- Map tasks to spec user story priorities:
  - **P1 tasks** = Must-have for MVP (from P1 user stories)
  - **P2 tasks** = Important enhancements (from P2 user stories)
  - **P3 tasks** = Nice-to-have (from P3 user stories or technical debt)

### Naming Conventions

- Use action verbs: "Create", "Implement", "Design", "Build", "Configure", "Register", "Set up"
- Be specific: "Create candidate list view with filters" vs "Make candidate page"
- Reference spec entities: Use exact naming from spec's data models
- For sub-subtasks: Use specific names of items being registered/configured

### Description Quality

Each description should answer:

- **What** is being built
- **Why** it's needed (link to spec requirement)
- **How** it fits into the larger system
- **When** it should be done (dependency order)

For sub-subtasks, add:

- **Specific Implementation Detail**: What exactly is this item doing
- **Expected Behavior**: How it should work or integrate

### Traceability

- Reference spec sections explicitly (e.g., "User Story 1.2", "Requirement FR-003")
- Link acceptance criteria back to spec Given-When-Then scenarios
- Preserve spec terminology (entity names, field names, status values)

## Example Invocations

**User**: "Generate task breakdown for Feature 001 AI Interview System"

**Process**:

1. Read `001-ai-hr-interview-system/spec.md`
2. Extract user stories (P1/P2/P3), requirements, entities
3. Categorize into Frontend, Backend, API, Database, AI Integration, Testing
4. Create main tasks (Interview Module, Resume Upload, AI Scoring Engine, etc.)
5. Break each into subtasks with descriptions
6. Generate `001-ai-hr-interview-system/TASK_BREAKDOWN.md`

**User**: "Break down the SaaS Admin Portal spec into implementation tasks"

**Process**:

1. Read `002-saas-admin-portal-company-management/spec.md`
2. Identify areas: Frontend (admin UI), Backend (company management), API (CRUD endpoints), Auth (God Mode)
3. Create tasks organized by area
4. Reference spec requirements and user stories in descriptions
5. Generate `002-saas-admin-portal-company-management/TASK_BREAKDOWN.md`

## Tips

- **Read the spec thoroughly** - Don't skip entities, requirements, or success metrics sections
- **Preserve spec language** - Use exact field names, entity names, status values from the spec
- **Include edge cases** - If spec mentions validation rules or error handling, create subtasks for them
- **Think deployment** - Include tasks for migrations, config, environment variables
- **Consider testing** - Add testing subtasks for critical paths (unit, integration, E2E)
- **Update as needed** - Task breakdowns are living documents; update when specs change

## Anti-Patterns to Avoid

❌ **Vague descriptions**: "Do the frontend stuff"  
✅ **Specific descriptions**: "Create candidate registration form with email validation and resume upload"

❌ **Vague subtask with no breakdown**: "Register Scheduled Jobs" (what jobs? which ones?)  
✅ **Clear sub-subtasks**:

- Register QuotaThresholdChecker job (every 15 min)
- Register BalanceThresholdChecker job (every 15 min)
- Register ExpiredReservationCleanup job (every hour)
- Register NotificationFlagReset job (daily at midnight)

❌ **Missing dependencies**: Tasks that can't be done without others being complete first  
✅ **Clear dependencies**: "Depends on: Database schema migration for candidates table"

❌ **Too granular**: "Write line 42 of controller.py"  
✅ **Right size**: "Implement candidate CRUD controller with validation"

❌ **Missing acceptance**: No way to know when task is done  
✅ **Clear acceptance**: "API returns 200 with candidate list, supports pagination (limit/offset)"

❌ **Ignoring priorities**: All tasks marked equally important  
✅ **Priority-aware**: P1 MVP tasks separated from P2 enhancements

❌ **Abstract task names that need explanation**: "Configure Job Scheduler"  
✅ **Breakdown with concrete details**: Create sub-subtasks explaining which jobs to register and their schedules
