---
name: task-breakdown-generator
description: Generate structured task breakdown markdown files from specification documents, organizing main tasks and subtasks by implementation area (Frontend, Backend, Database, Testing) with descriptions and clear dependency linkages. Use when users ask to create task lists, development breakdowns, work items, or implementation plans from feature specs, requirements documents, or technical specifications.
license: Complete terms in LICENSE.txt
---

# Task Breakdown Generator

Automatically generates structured markdown files that break down feature specifications into actionable main tasks and subtasks, organized by implementation area for developer clarity.

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

Organize tasks into **4-5 core categories** to avoid duplication:

**Core Categories (Always Use):**

- **Database** - Schema design, migrations, indexes, relationships
- **Backend** - Business logic, services, **AND API endpoints** (combined to avoid duplication)
- **Frontend** - UI components, pages, forms, user interactions
- **Testing** - Unit tests, integration tests, E2E tests

**Optional Categories (Add When Needed):**

- **Integration** - Third-party services, external APIs (e.g., AI models, payment gateways)
- **DevOps/Infrastructure** - Deployment, monitoring, CI/CD (if significant work)
- **Documentation** - API docs, user guides (if explicitly required)

**⚠️ IMPORTANT**: API endpoints belong in the **Backend** section alongside service layer logic. Do NOT create a separate "API" category—this causes duplication since APIs and services are tightly coupled.

### 3. Create Task Hierarchy

**Main Tasks** = High-level feature modules (1-5 days of work)

- Examples: "Candidate Management Module", "JD Versioning Service", "Version History UI"
- Each main task represents a cohesive unit of work

**Subtasks** = Specific implementation items (0.5-2 days of work)

- Examples: "Implement version creation logic", "Create version history API endpoint", "Build diff viewer component"
- Each subtask should be independently testable

**⚠️ AVOID Sub-Subtasks Unless Absolutely Necessary**

- Default to **2 levels only** (Main Task → Subtask)
- Only use sub-subtasks when a subtask contains **3+ distinct configuration items** that need listing
- If a subtask seems too large, promote it to a Main Task instead

### 3.1. Simplified Cross-References

Use **3 essential linkage types** to show relationships without excessive overhead:

| Field             | Purpose                         | Example                         |
| ----------------- | ------------------------------- | ------------------------------- |
| **Depends On**    | What must exist first           | `Database 1.1 (versions table)` |
| **Related**       | Connected tasks (bidirectional) | `Frontend 1.2, Backend 2.3`     |
| **Test Coverage** | Which test validates this       | `Testing 3.1`                   |

**Linkage Rules:**

1. **Frontend → Backend**: List which Backend subtasks the Frontend calls
2. **Backend → Database**: List which Database subtasks must exist
3. **Testing → Implementation**: List which tasks the test covers

**Format**: Use short references like `Backend 2.1` or `Database 1.3` (no need to repeat "Task")

### 4. Write Task Descriptions

For each subtask, include these **essential fields**:

- **Description**: What is being built (clear and specific)
- **Depends On**: Other tasks that must complete first
- **Related**: Connected tasks in other areas (bidirectional)
- **Acceptance**: How to know when it's done

**Optional fields** (use when helpful):

- **Why**: Business justification (for non-obvious tasks)
- **Files**: Key files to create/modify
- **API Details**: Request/response schema (for API endpoints)

## Output Format

Generate a markdown file with this streamlined structure:

```markdown
# Task Breakdown: [Feature Name]

**Source**: [Path to spec file]  
**Generated**: [Date]  
**Status**: Planning

---

## Overview

[Brief summary of the feature and scope of work]

---

## Database

### Main Task 1: [Schema/Module Name]

**Description**: [What this accomplishes]

#### Subtask 1.1: [Table name] Table

- **Description**: Create migration for [entity] table
- **Columns**: [Key columns and types]
- **Indexes**: [Performance indexes]
- **Related**: Backend 2.1, Backend 2.3
- **Acceptance**: Migration runs successfully

---

## Backend

### Main Task 2: [Service/Module Name]

**Description**: [What this accomplishes - includes both service logic AND API endpoints]

**Requirement Reference**: [FR-xxx from spec]

#### Subtask 2.1: [Feature] Service Logic

- **Description**: Implement core business logic for [feature]
- **Methods**: `ServiceName.methodOne()`, `ServiceName.methodTwo()`
- **Depends On**: Database 1.1
- **Related**: Frontend 1.1
- **Acceptance**: [Success criteria]

#### Subtask 2.2: API - GET /api/[resource]

- **Description**: List [resources] with pagination
- **Request**: Query params: `page`, `limit`, `filter`
- **Response**: `{ items: [...], total: number }`
- **Depends On**: Backend 2.1 (service logic)
- **Related**: Frontend 1.2
- **Test Coverage**: Testing 4.1
- **Acceptance**: Returns paginated results in <500ms

#### Subtask 2.3: API - POST /api/[resource]

- **Description**: Create new [resource]
- **Request**: `{ field1, field2, ... }`
- **Response**: `{ id, ...created resource }`
- **Depends On**: Backend 2.1
- **Related**: Frontend 1.3
- **Test Coverage**: Testing 4.2
- **Acceptance**: Creates resource, returns 201

---

## Frontend

### Main Task 3: [Component/Module Name]

**Description**: [What this accomplishes]

**User Story Reference**: [User Story X from spec]

#### Subtask 3.1: [Component/Page Name]

- **Description**: Build UI for [feature]
- **Components**: `ComponentA`, `ComponentB`
- **Depends On**: Backend 2.2 (API endpoint)
- **Test Coverage**: Testing 5.1
- **Acceptance**: [User-facing success criteria]
- **Files**: `components/X.tsx`, `pages/Y.tsx`

---

## Testing

### Main Task 4: [Test Suite Name]

**Description**: Test coverage for [feature area]

#### Subtask 4.1: [Test Name]

- **Description**: [What is being tested]
- **Covers**: Backend 2.1, Backend 2.2
- **Type**: Unit / Integration / E2E
- **Scenarios**: [Key test cases from spec GWT]
- **Acceptance**: All tests pass

---

## Task Summary

| Area      | Main Tasks | Subtasks |
| --------- | ---------- | -------- |
| Database  | X          | Y        |
| Backend   | X          | Y        |
| Frontend  | X          | Y        |
| Testing   | X          | Y        |
| **Total** | **X**      | **Y**    |

**Priority Distribution**:

- P1 (MVP): X tasks
- P2 (Enhancement): Y tasks

---

## Dependencies & Sequencing

### Phase 1: Foundation

- Database schema (blocks all backend work)
- Core service logic

### Phase 2: Core Features (can parallelize)

- Backend APIs
- Frontend components

### Phase 3: Integration & Testing

- E2E tests
- Feature integrations

---

## Notes

[Clarifications, assumptions, open questions]
```

## Best Practices

### Task Sizing

- **Main tasks** = 1-5 days of work for a developer
- **Subtasks** = 0.5-2 days of work (story-point size)
- If a subtask is larger than 2 days, split it into multiple subtasks or promote to Main Task

### Backend Task Organization

**Combine service logic and APIs within the same Main Task:**

```markdown
### Main Task 2: Candidate Management Service

#### Subtask 2.1: Core Service Logic

- CandidateService.create(), .update(), .delete(), .findAll()

#### Subtask 2.2: API - GET /api/candidates

- Calls CandidateService.findAll()

#### Subtask 2.3: API - POST /api/candidates

- Calls CandidateService.create()

#### Subtask 2.4: API - PUT /api/candidates/{id}

- Calls CandidateService.update()
```

**Benefits**:

- Developer sees full vertical slice (service + API) in one place
- No jumping between "Backend" and "API" sections
- Clear relationship between service methods and their API consumers

### Priority Mapping

- Map tasks to spec user story priorities:
  - **P1 tasks** = Must-have for MVP (from P1 user stories)
  - **P2 tasks** = Important enhancements (from P2 user stories)

### Naming Conventions

- Use action verbs: "Create", "Implement", "Build", "Configure"
- Be specific: "Build candidate list with pagination" vs "Make candidate page"
- For APIs: Use format "API - HTTP_METHOD /path" (e.g., "API - GET /api/candidates")
- Reference spec entities: Use exact naming from spec's data models

### Cross-Referencing Best Practices

- **Use short format**: `Backend 2.1` instead of `Backend Task 2.1`
- **Be specific**: Include what the reference is for: `Backend 2.3 (POST /api/jobs)`
- **Keep it simple**: Don't over-link; focus on direct dependencies
- **Bidirectional only when helpful**: Frontend lists Backend dependencies; Backend can note "Related: Frontend 1.2" but it's optional

### Traceability

- Reference spec sections: "User Story 1.2", "Requirement FR-003"
- Link acceptance criteria to spec Given-When-Then scenarios
- Preserve spec terminology (entity names, field names)

## Example Invocations

**User**: "Generate task breakdown for Feature 001 AI Interview System"

**Process**:

1. Read `001-ai-hr-interview-system/spec.md`
2. Extract user stories (P1/P2), requirements, entities
3. Categorize into: Database, Backend (services + APIs), Frontend, Testing
4. Create main tasks with clear subtasks
5. Add simplified cross-references (Depends On, Related, Test Coverage)
6. Generate `001-ai-hr-interview-system/TASK_BREAKDOWN.md`

**User**: "Break down the SaaS Admin Portal spec into implementation tasks"

**Process**:

1. Read `002-saas-admin-portal-company-management/spec.md`
2. Identify areas: Database (company/user tables), Backend (CRUD services + APIs), Frontend (admin UI), Testing
3. Organize Backend to include both service logic and API endpoints together
4. Add cross-references between areas
5. Generate `002-saas-admin-portal-company-management/TASK_BREAKDOWN.md`

## Tips

- **Read the spec thoroughly** - Extract all entities, requirements, and success metrics
- **Preserve spec language** - Use exact field names, entity names from the spec
- **Include edge cases** - If spec mentions validation, create subtasks for it
- **Keep Backend unified** - Service logic and APIs in same section, same main task when related
- **Test coverage matters** - Every key subtask should reference which test covers it
- **Think in phases** - Group tasks by dependency order for realistic sequencing

## Anti-Patterns to Avoid

❌ **Separate API section from Backend**: Creates duplication  
✅ **Combined Backend**: Service logic + API endpoints together

❌ **Too many linkage fields**: Uses, Called By, Depends On, Used By, Tested By, Tests, Service Method...  
✅ **Three fields**: Depends On, Related, Test Coverage

❌ **Sub-sub-subtasks**: Three+ levels of nesting  
✅ **Two levels max**: Main Task → Subtask (promote if too complex)

❌ **Vague descriptions**: "Do the backend stuff"  
✅ **Specific descriptions**: "Implement candidate CRUD with pagination and filtering"

❌ **Missing acceptance**: No way to know when done  
✅ **Clear acceptance**: "API returns paginated list in <500ms; 401 for unauthorized"

❌ **Ignoring priorities**: All tasks marked equally  
✅ **Priority-aware**: P1 MVP tasks clearly separated from P2 enhancements

---

## Validation Checklist (Quick Check)

Before presenting the TASK_BREAKDOWN.md, run these **3 quick checks**:

### Check 1: Reference Existence

Every task reference (e.g., `Backend 2.1`, `Database 1.3`) must point to a task that exists.

```
✓ Frontend 1.1 says "Depends On: Backend 2.1" → Backend 2.1 exists
✗ Frontend 1.2 says "Depends On: Backend 2.5" → Backend 2.5 doesn't exist (ERROR)
```

### Check 2: Dependency Flow

Dependencies should flow in logical order:

```
Database → Backend → Frontend → Testing
```

- Backend depends on Database (not vice versa)
- Frontend depends on Backend APIs (not vice versa)
- Testing covers implementation tasks

### Check 3: Test Coverage

Every P1 Backend and Frontend subtask should have a corresponding test:

```
✓ Backend 2.1 → has "Test Coverage: Testing 4.1"
✗ Frontend 1.3 → no test coverage mentioned (WARNING)
```

### Validation Summary (Optional)

Add at end of TASK_BREAKDOWN.md if helpful:

```markdown
---

## Validation Summary

✅ All references point to existing tasks
✅ Dependency flow is correct (Database → Backend → Frontend)
✅ P1 tasks have test coverage
```

---

## Summary of Key Principles

1. **4 Core Sections**: Database, Backend (includes APIs), Frontend, Testing
2. **2 Levels of Nesting**: Main Task → Subtask (avoid sub-subtasks)
3. **3 Linkage Types**: Depends On, Related, Test Coverage
4. **Unified Backend**: Service logic and API endpoints in same section/main task
5. **Quick Validation**: 3 checks before presenting (existence, flow, coverage)
