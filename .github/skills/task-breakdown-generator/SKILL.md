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
- **Backend** - Business logic, data processing, domain models, API endpoints (REST/GraphQL), request/response handling
- **Database** - Schema design, migrations, indexes
- **Authentication/Authorization** - User management, permissions, RBAC
- **Integration** - Third-party services, external APIs
- **DevOps/Infrastructure** - Deployment, monitoring, CI/CD
- **Testing** - Unit tests, integration tests, E2E tests
- **Documentation** - API docs, user guides, deployment guides

**Custom Categories**: Add domain-specific areas as needed (e.g., "AI Model Integration", "Report Generation", "Audit Logging")

**Note**: API endpoints are organized under Backend as they represent the backend's interface layer.

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

### 3.1. Add Cross-Area Linkages

**Purpose**: Make the documentation easier to navigate by explicitly showing dependencies and relationships between areas.

**Linkage Types**:

1. **Frontend → Backend**: Reference which Backend API endpoints/methods the Frontend depends on
   - Example: "Depends on Backend Task 2.1 (GET /api/candidates endpoint)"
   - Example: "Calls CandidateService.createCandidate() method from Backend Task 2.2"

2. **Backend → Database**: Reference which database tables/migrations must exist
   - Example: "Requires Database Task 1.1 (candidates table) to be completed"

3. **Testing → Frontend/Backend**: Reference which tasks the test is validating
   - Example: "Tests Frontend Task 1.1 (Candidate List View) and Backend Task 2.1 (GET /api/candidates)"

4. **Backend API → Backend Service**: Reference which service layer methods the API calls
   - Example: "API endpoint delegates to CandidateService.updateStatus() from Backend Task 2.3"

**How to Document Linkages**:

- Add a **"Depends On"** or **"Uses"** field in subtask descriptions
- Add a **"Tested By"** field referencing which test tasks validate this functionality
- Add a **"Called By"** field in Backend tasks showing which Frontend/API tasks consume them
- Use task numbers for easy cross-referencing (e.g., "Frontend 1.2", "Backend 2.3", "Testing 1.1")

### 4. Write Task Descriptions

For each task and subtask, include:

- **What**: Clear description of the work
- **Why**: Business value or technical necessity (reference spec user story/requirement)
- **Scope**: Key components, files, or areas affected
- **Depends On**: Other tasks that must complete first (cross-area references with task numbers)
- **Uses**: Which Backend APIs/methods this task calls (for Frontend tasks)
- **Called By**: Which Frontend/API tasks consume this (for Backend tasks)
- **Tested By**: Which test tasks validate this functionality
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
- **Uses**: Backend Task [X.Y] - [API endpoint or method name]
- **Tested By**: Testing Task [Z.W]
- **Acceptance**: [Success criteria]
- **Files**: [Key files to create/modify]

#### Subtask 1.2: [Another specific element]

- **Description**: [Detailed description]
- **Uses**: Backend Task [X.Y] - [API endpoint or method name]
- **Tested By**: Testing Task [Z.W]
- **Acceptance**: [Success criteria]
- **Files**: [Key files]

### Main Task 2: [Another Component]

...

---

## Backend

### Main Task 1: [Service/Module Name]

**Description**: [What this accomplishes]

**Requirement Reference**: [Functional requirement from spec]

#### Subtask 1.1: [API Endpoint - HTTP Method /api/path]

- **Description**: [Endpoint purpose and functionality]
- **Request**: [Request body/params schema]
- **Response**: [Response schema]
- **Service Method**: Calls [ServiceName.methodName()] from Backend Task [X.Y]
- **Called By**: Frontend Task [A.B]
- **Tested By**: Testing Task [Z.W]
- **Acceptance**: [API success criteria]

#### Subtask 1.2: [Service Layer - Method Name]

- **Description**: [Business logic details]
- **Depends On**: Database Task [X.Y] - [Table name]
- **Called By**: Backend Task [A.B] - [API endpoint], Frontend (via API)
- **Tested By**: Testing Task [Z.W]
- **Acceptance**: [Criteria]

##### Sub-Subtask 1.1.1: [Granular implementation detail]

- **Description**: [Specific work item]
- **Why**: [Why this is needed]
- **Acceptance**: [Success criteria for this specific item]

##### Sub-Subtask 1.1.2: [Another granularJobs detail]

- **Description**: [Details]
- **Acceptance**: [Criteria]

...

---

## Database

### Main Task 1: Schema Design

#### Subtask 1.1: [Entity name] Table

- **Description**: Design and create migration for [entity] table
- **Columns**: [Key columns and relationships]
- **Indexes**: [Performance indexes]
- **Used By**: Backend Task [X.Y] - [Service/method name]
- **Acceptance**: Migration runs successfully in dev/staging

...

---

## Testing

### Main Task 1: [Test Suite Name]

**Description**: [What this test suite covers]

#### Subtask 1.1: [Specific test case or test file]

- **Description**: [What is being tested]
- **Tests**: Frontend Task [X.Y], Backend Task [A.B]
- **Test Type**: [Unit/Integration/E2E]
- **Scenarios**: [Key test scenarios from spec]
- **Acceptance**: All tests pass, coverage > [X]%

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
- **Who** consumes/depends on this (cross-references)

For sub-subtasks, add:

- **Specific Implementation Detail**: What exactly is this item doing
- **Expected Behavior**: How it should work or integrate

### Cross-Referencing

- **Use task numbers consistently**: Frontend 1.2, Backend 2.3, Database 1.1, Testing 3.1
- **Be explicit about dependencies**: Don't say "depends on API", say "depends on Backend Task 2.1 (GET /api/candidates endpoint)"
- **Show bidirectional links**:
  - Frontend tasks list which Backend tasks they use
  - Backend tasks list which Frontend tasks call them
  - Testing tasks list which Frontend/Backend tasks they validate
- **Update links when tasks change**: If you renumber tasks, update all cross-references

### Traceability

- Reference spec sections explicitly (e.g., "User Story 1.2", "Requirement FR-003")
- Link acceptance criteria back to spec Given-When-Then scenarios
- Preserve spec terminology (entity names, field names, status values)
- Cross-reference related tasks using task numbers (e.g., "Depends On: Backend 2.1", "Called By: Frontend 1.3")
- Include "Tested By" references in implementation tasks to link to test coverage

## Example Invocations

**User**: "Generate task breakdown for Feature 001 AI Interview System"

**Process**:

1. Read `001-ai-hr-interview-system/spec.md`
2. Extract user stories (P1/P2/P3), requirements, entities
3. Categorize into Frontend, Backend (including APIs), Database, AI Integration, Testing
4. Create main tasks (Interview Module, Resume Upload, AI Scoring Engine, etc.)
5. Break each into subtasks with descriptions and cross-references
6. Add "Uses", "Called By", "Tested By" fields to show relationships
7. Generate `001-ai-hr-interview-system/TASK_BREAKDOWN.md`

**User**: "Break down the SaaS Admin Portal spec into implementation tasks"

**Process**:

1. Read `002-saas-admin-portal-company-management/spec.md`
2. Identify areas: Frontend (admin UI), Backend (company management, API endpoints), Database, Auth (God Mode), Testing
3. Create tasks organized by area with cross-references
4. Add linkages: Frontend UI → Backend APIs → Database tables → Testing
5. Reference spec requirements and user stories in descriptions
6. Generate `002-saas-admin-portal-company-management/TASK_BREAKDOWN.md`

## Tips

- **Read the spec thoroughly** - Don't skip entities, requirements, or success metrics sections
- **Preserve spec language** - Use exact field names, entity names, status values from the spec
- **Include edge cases** - If spec mentions validation rules or error handling, create subtasks for them
- **Think deployment** - Include tasks for migrations, config, environment variables
- **Consider testing** - Add testing subtasks for critical paths (unit, integration, E2E)
- **Add cross-references early** - As you create Backend APIs, note which Frontend tasks will call them
- **Number tasks sequentially** - Use consistent numbering (1.1, 1.2, 2.1) for easy cross-referencing
- **Link bidirectionally** - If Frontend 1.2 uses Backend 2.3, note this in both tasks
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
✅ **Clear dependencies**: "Depends On: Database Task 1.1 (candidates table migration)"

❌ **No cross-references**: Frontend task doesn't mention which Backend API it calls  
✅ **Clear linkage**: "Uses: Backend Task 2.1 (GET /api/candidates endpoint)"

❌ **One-way references**: Backend task doesn't show who uses it  
✅ **Bidirectional links**: "Called By: Frontend Task 1.2 (Candidate List View)"

❌ **Too granular**: "Write line 42 of controller.py"  
✅ **Right size**: "Implement candidate CRUD controller with validation"

❌ **Missing acceptance**: No way to know when task is done  
✅ **Clear acceptance**: "API returns 200 with candidate list, supports pagination (limit/offset)"

❌ **Ignoring priorities**: All tasks marked equally important  
✅ **Priority-aware**: P1 MVP tasks separated from P2 enhancements

❌ **Abstract task names that need explanation**: "Configure Job Scheduler"  
✅ **Breakdown with concrete details**: Create sub-subtasks explaining which jobs to register and their schedules

---

## Verification & Validation Checklist (MANDATORY)

⚠️ **CRITICAL**: After generating ANY TASK_BREAKDOWN.md file, you MUST perform a **comprehensive cross-reference validation** before presenting it to the user. This step is NON-OPTIONAL to prevent broken references and incomplete dependency tracking.

### Validation Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK BREAKDOWN GENERATION                        │
├─────────────────────────────────────────────────────────────────────┤
│ Step 1: Generate Initial Breakdown                                  │
│         ↓                                                           │
│ Step 2: Build Cross-Reference Maps (Phase 1)                        │
│         ↓                                                           │
│ Step 3: Validate Bidirectional Links (Phase 2)                      │
│         ↓                                                           │
│ Step 4: Detect Error Patterns (Phase 3)                             │
│         ↓                                                           │
│ Step 5: Fix ALL Errors Found                                        │
│         ↓                                                           │
│ Step 6: Re-validate Until 0 Errors                                  │
│         ↓                                                           │
│ Step 7: Generate Validation Summary (Phase 4)                       │
│         ↓                                                           │
│ Step 8: Present Final Validated TASK_BREAKDOWN.md                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Build Cross-Reference Maps

**BEFORE presenting the file, extract ALL cross-references into structured maps:**

**1. Parse ALL linkage fields systematically:**

| Linkage Type     | Direction                     | Expected Reverse Linkage             |
| ---------------- | ----------------------------- | ------------------------------------ |
| `Uses`           | Frontend → Backend            | `Called By` on Backend               |
| `Called By`      | Backend → Frontend/Backend    | `Uses` or `Service Method` on caller |
| `Depends On`     | Any → Database/Other          | `Used By` on dependency              |
| `Used By`        | Database → Backend            | `Depends On` on user                 |
| `Tested By`      | Implementation → Testing      | `Tests` on Testing                   |
| `Tests`          | Testing → Implementation      | `Tested By` on Implementation        |
| `Service Method` | Backend API → Backend Service | `Called By` on Service               |

**2. Build reference inventory table:**

```
| Task ID       | References (Outgoing)                    | Referenced By (Incoming Expected) |
|---------------|------------------------------------------|-----------------------------------|
| Frontend 1.1  | Uses: Backend 2.1, Backend 2.3           | Should appear in Backend 2.1, 2.3 |
| Backend 2.1   | Depends On: Database 1.1                 | Should appear in Database 1.1     |
|               | Called By: Frontend 1.1                  | Should match Frontend 1.1 Uses    |
| Database 1.1  | Used By: Backend 2.1, Backend 2.2        | Should match Backend Depends On   |
| Testing 1.1   | Tests: Frontend 1.1, Backend 2.1         | Should appear as Tested By        |
```

**3. Create validation checklist for each task:**

For EVERY task in the breakdown, verify:

- [ ] All outgoing references point to existing tasks
- [ ] All outgoing references have corresponding incoming references on target
- [ ] Task numbers are consistent (no typos like "Backend 2.1" vs "Backend Task 2.1")
- [ ] Service method names match actual implementations

### Phase 2: Validate Bidirectional References (MANDATORY CHECKS)

**Rule 1: Uses ↔ Called By (Frontend ↔ Backend)**

```
✓ CORRECT:
  Frontend 1.2: "Uses: Backend Task 2.1 (GET /api/candidates)"
  Backend 2.1:  "Called By: Frontend Task 1.2 (Candidate List View)"

✗ INCORRECT:
  Frontend 1.2: "Uses: Backend Task 2.1"
  Backend 2.1:  [No "Called By" field or doesn't mention Frontend 1.2]
```

**Validation Action**: For EVERY `Uses` field, find the target task and verify it has a `Called By` that includes the source task.

**Rule 2: Depends On ↔ Used By (Implementation ↔ Database/Dependencies)**

```
✓ CORRECT:
  Backend 2.1:  "Depends On: Database Task 1.1 (candidates table)"
  Database 1.1: "Used By: Backend Task 2.1 (CandidateService)"

✗ INCORRECT:
  Backend 2.1:  "Depends On: Database Task 1.1"
  Database 1.1: [No "Used By" field or doesn't mention Backend 2.1]
```

**Validation Action**: For EVERY `Depends On` field, find the target task and verify it has a `Used By` that includes the source task.

**Rule 3: Tested By ↔ Tests (Implementation ↔ Testing)**

```
✓ CORRECT:
  Backend 2.1: "Tested By: Testing Task 3.1"
  Testing 3.1: "Tests: Backend Task 2.1, Frontend Task 1.2"

✗ INCORRECT:
  Backend 2.1: "Tested By: Testing Task 3.1"
  Testing 3.1: "Tests: Backend Task 2.2"  ← Mismatch!
```

**Validation Action**: For EVERY `Tested By` field, find the target Testing task and verify its `Tests` field includes the source task.

**Rule 4: Service Method References ↔ Backend Service Tasks**

```
✓ CORRECT:
  Backend 2.1 (API): "Service Method: Calls CandidateService.create() from Backend Task 2.5"
  Backend 2.5:       "Called By: Backend Task 2.1 (POST /api/candidates API)"

✗ INCORRECT:
  Backend 2.1: "Calls CandidateService.create()"
  Backend 2.5: [Doesn't implement create() or no "Called By" reference]
```

**Validation Action**: For EVERY service method call, verify the service task implements that method and has a `Called By` reference.

**Rule 5: Task Number Format Consistency**

```
✓ CORRECT (pick ONE format and stick to it):
  "Uses: Backend Task 2.1"
  "Called By: Frontend Task 1.2"

✗ INCORRECT (mixing formats):
  "Uses: Backend 2.1"           ← Missing "Task"
  "Called By: Frontend Task 1.2" ← Has "Task"
```

**Validation Action**: Ensure ALL task references use the SAME format throughout the document.

### Phase 3: Detect and Fix Error Patterns

**Run these checks BEFORE finalizing the document:**

| Error Pattern              | Detection Method                                                    | Fix                                 |
| -------------------------- | ------------------------------------------------------------------- | ----------------------------------- |
| **Orphaned Reference**     | Task X mentions Task Y, but Task Y doesn't exist                    | Create Task Y or remove reference   |
| **One-Way Link**           | Task X references Task Y, but Y doesn't reference X                 | Add reverse reference to Task Y     |
| **Task Number Typo**       | "Backend 2.1" vs "Backend Task 2.1" vs "Backend 21"                 | Standardize all references          |
| **Missing Tested By**      | Implementation task has no Tested By field                          | Add Testing task reference          |
| **Stale Reference**        | Reference to task that was renamed/renumbered                       | Update reference to current task ID |
| **Duplicate Task Numbers** | Two tasks have same number (e.g., two "Backend 2.1")                | Renumber one task                   |
| **Cross-Section Mismatch** | Frontend 1.2 says "Uses Backend 2.1" but Backend section has no 2.1 | Create Backend 2.1 or fix reference |

**Error Detection Checklist (MUST complete for every generation):**

```
[ ] Count all Frontend tasks and verify each has:
    [ ] At least one "Uses" reference (if it calls backend)
    [ ] "Tested By" reference
[ ] Count all Backend tasks and verify each has:
    [ ] "Called By" reference (who calls this API/service?)
    [ ] "Depends On" reference (what database tables?)
    [ ] "Tested By" reference
[ ] Count all Database tasks and verify each has:
    [ ] "Used By" reference (which backend tasks use this table?)
[ ] Count all Testing tasks and verify each has:
    [ ] "Tests" reference listing what it tests
[ ] Cross-check: Every "Uses" has matching "Called By"
[ ] Cross-check: Every "Depends On" has matching "Used By"
[ ] Cross-check: Every "Tested By" has matching "Tests"
```

### Phase 4: Generate Validation Summary

**ALWAYS include this section at the end of TASK_BREAKDOWN.md:**

```markdown
---

## Cross-Reference Validation Summary

**Validation Date**: [Date]
**Validation Status**: ✅ PASSED / ⚠️ WARNINGS / ❌ FAILED

### Reference Counts

| Linkage Type | Total Count | Validated | Issues |
|--------------|-------------|-----------|--------|
| Uses → Called By | X | X | 0 |
| Depends On → Used By | X | X | 0 |
| Tested By → Tests | X | X | 0 |
| Service Method refs | X | X | 0 |

### Validation Checks Performed

- [x] All Frontend tasks have matching Backend "Called By" references
- [x] All Backend tasks have matching Database "Used By" references
- [x] All Implementation tasks have "Tested By" references
- [x] All Testing tasks have "Tests" references pointing to valid tasks
- [x] Task numbering is consistent (no duplicates, no gaps)
- [x] Task reference format is consistent throughout

### Issues Found and Fixed

| Issue | Location | Resolution |
|-------|----------|------------|
| (None if validation passed) | | |

---
```

### Phase 5: Iterative Validation Loop

**DO NOT present the TASK_BREAKDOWN.md until validation passes:**

```
WHILE validation_errors > 0:
    1. Identify all errors from Phase 3 checks
    2. Fix each error:
       - Add missing reverse references
       - Correct task number typos
       - Create missing tasks or remove orphaned references
    3. Re-run Phase 2 validation
    4. Update error count

IF validation_errors == 0:
    Add Validation Summary (Phase 4)
    Present final TASK_BREAKDOWN.md
```

### Quick Reference: Linkage Pairs

**Memorize these pairs - they ALWAYS go together:**

| If Task A has...                   | Then Target Task B MUST have...   |
| ---------------------------------- | --------------------------------- |
| `Uses: B`                          | `Called By: A`                    |
| `Called By: B`                     | `Uses: A` (or Service Method ref) |
| `Depends On: B`                    | `Used By: A`                      |
| `Used By: B`                       | `Depends On: A`                   |
| `Tested By: B`                     | `Tests: A`                        |
| `Tests: B`                         | `Tested By: A`                    |
| `Service Method: calls B.method()` | `Called By: A`                    |

### Example: Complete Validation Walkthrough

**Scenario**: Validating a Feature 009 TASK_BREAKDOWN.md

**Step 1: Extract References**

```
Frontend 1.1 → Uses: Backend 2.1, Backend 2.3
Frontend 1.2 → Uses: Backend 2.5
Backend 2.1  → Depends On: Database 1.1; Tested By: Testing 3.1
Backend 2.3  → Depends On: Database 1.1; Tested By: Testing 3.2
Backend 2.5  → Depends On: Database 1.2; Tested By: Testing 3.3
Database 1.1 → (check for Used By)
Database 1.2 → (check for Used By)
Testing 3.1  → Tests: (should include Backend 2.1)
Testing 3.2  → Tests: (should include Backend 2.3)
Testing 3.3  → Tests: (should include Backend 2.5)
```

**Step 2: Validate Bidirectional Links**

```
Check 1: Frontend 1.1 Uses Backend 2.1
  → Does Backend 2.1 have "Called By: Frontend 1.1"? YES ✓

Check 2: Frontend 1.1 Uses Backend 2.3
  → Does Backend 2.3 have "Called By: Frontend 1.1"? NO ✗
  → FIX: Add "Called By: Frontend Task 1.1" to Backend 2.3

Check 3: Backend 2.1 Depends On Database 1.1
  → Does Database 1.1 have "Used By: Backend 2.1"? YES ✓

Check 4: Backend 2.1 Tested By Testing 3.1
  → Does Testing 3.1 have "Tests: Backend 2.1"? YES ✓
```

**Step 3: Fix Errors**

```
ERROR: Backend 2.3 missing "Called By: Frontend Task 1.1"
ACTION: Add "Called By: Frontend Task 1.1 (Version History View)" to Backend 2.3
```

**Step 4: Re-validate**

```
All checks pass ✓
```

**Step 5: Add Validation Summary**

```markdown
## Cross-Reference Validation Summary

**Validation Date**: 2026-02-02
**Validation Status**: ✅ PASSED

### Issues Found and Fixed

| Issue                       | Location    | Resolution                           |
| --------------------------- | ----------- | ------------------------------------ |
| Missing Called By reference | Backend 2.3 | Added "Called By: Frontend Task 1.1" |
```

---

## Validation Anti-Patterns (NEVER DO THESE)

❌ **Skip validation because "it looks right"**
✅ **Always run the full validation checklist**

❌ **Fix errors without re-validating**
✅ **Always re-run validation after fixes**

❌ **Present TASK_BREAKDOWN.md with known errors**
✅ **Fix ALL errors before presenting to user**

❌ **Use inconsistent task reference formats**
✅ **Pick one format ("Backend Task 2.1") and use it everywhere**

❌ **Add references without checking target exists**
✅ **Verify every referenced task actually exists in the document**

❌ **Assume one-way references are OK**
✅ **Every reference MUST have a reverse reference (bidirectional)**
