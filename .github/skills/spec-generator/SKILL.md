---
name: spec-generator
description: Generate structured feature specifications following the spec-template.md format with prioritized user stories (P1/P2/P3), acceptance criteria (Given-When-Then), functional requirements, and success metrics. Use when creating feature specifications for the AI-Talent-Analyst platform or similar projects using the speckit.specify workflow.
---

# Spec Generator

Transform feature descriptions into comprehensive, stakeholder-ready specifications that guide implementation and testing.

## When to Use This Skill

- Creating a new feature specification from natural language requirements
- Converting feature ideas into structured user stories with priorities
- Generating acceptance criteria in Given-When-Then format
- Defining measurable success criteria for features
- Building specifications that can be independently tested and implemented

## What This Skill Does

1. **Parses feature description** - Extracts key concepts, actors, actions, data, and constraints
2. **Generates prioritized user stories** - Creates P1 (MVP-critical), P2 (enhancement), P3 (nice-to-have) stories
3. **Defines acceptance scenarios** - Uses Given-When-Then format for testable criteria
4. **Identifies functional requirements** - Lists system capabilities and behaviors
5. **Creates success criteria** - Defines measurable, technology-agnostic outcomes
6. **Maps key entities** - Documents data models and relationships
7. **Flags clarifications** - Marks areas needing stakeholder input (max 3)

## Core Specification Rules

### User Story Structure

Each user story MUST be:

- **Independently testable** - Can be developed/tested in isolation and still deliver value
- **Prioritized** - Assigned P1 (critical MVP), P2 (enhancement), or P3 (nice-to-have)
- **Motivation-aware** - Explains _why_ this priority (business value/impact)
- **Scenario-based** - Includes 2+ Given-When-Then acceptance scenarios

### Acceptance Scenario Format (Given-When-Then)

Each scenario uses this structure:

```
Given [initial state]
When [user action]
Then [expected outcome]
```

**Rules:**

- All three parts (Given, When, Then) are mandatory
- Must be testable without knowing implementation details
- Focus on user-observable behavior, not internal logic

### Functional Requirements Format

Requirements use this format:

```
- **FR-### [Code]**: System MUST [specific capability]
```

**Rules:**

- All requirements must be testable
- No implementation details (frameworks, APIs, code patterns)
- Include edge cases and error scenarios
- Use "MUST" for mandatory, "SHOULD" for recommended

### Success Criteria Format

Success criteria must be:

- **Measurable** - Include specific metrics (time, percentage, count)
- **Technology-agnostic** - No frameworks, databases, or tools
- **User-focused** - Describe outcomes from user/business perspective
- **Verifiable** - Can be tested without implementation knowledge

**Examples:**

- ✅ "Users can complete checkout in under 3 minutes"
- ✅ "System handles 10,000 concurrent users without degradation"
- ❌ "API response time is under 200ms" (too technical)
- ❌ "Database can handle 1000 TPS" (implementation detail)

### Key Entities

When data is involved, document:

- Entity name and purpose
- Key attributes (without implementation)
- Relationships to other entities
- No schema details (that's for implementation)

## Specification Generation Process

### Step 1: Parse Feature Description

From the user input, extract:

- **Primary actors** - Who uses this feature?
- **Main actions** - What do they do?
- **Data involved** - What entities/information?
- **Constraints** - Any limits or special cases?

### Step 2: Generate User Stories

Create 3-5 user stories covering:

**P1 (MVP-Critical)**: Bare minimum for feature to deliver value

- Can be implemented first
- Standalone functional slice
- Demonstrates core value
- Example: "User Story 1 - Create Job (Priority: P1)"

**P2 (Enhancement)**: Valuable additions that extend core functionality

- Builds on P1
- Adds user convenience or power features
- Example: "User Story 2 - Filter Jobs by Competency (Priority: P2)"

**P3 (Nice-to-Have)**: Future improvements or edge cases

- Can be deferred
- Handles edge cases or optimizations
- Example: "User Story 3 - Bulk Import Jobs (Priority: P3)"

### Step 3: Write Acceptance Scenarios

For each user story, provide 2-3 Given-When-Then scenarios covering:

- **Happy path** - Normal, successful flow
- **Alternative path** - Different user choice or scenario
- **Error case** - Error condition and recovery

### Step 4: Define Requirements

List 4-7 functional requirements covering:

- Core capabilities (what system MUST do)
- Data handling (storage, validation, persistence)
- Error handling (what happens when things go wrong)
- User interactions (how users accomplish tasks)
- Integrations (if applicable)

### Step 5: Identify Entities

If the feature involves data:

- List each data entity
- Describe its purpose and attributes
- Show relationships to other entities
- Keep it technology-agnostic

### Step 6: Create Success Criteria

Define 3-5 measurable outcomes:

- Include time/speed metrics
- Include volume/scale metrics
- Include quality/accuracy metrics
- Include user satisfaction metrics

### Step 7: Handle Clarifications

If specification is ambiguous:

- Mark unclear areas with `[NEEDS CLARIFICATION: question]`
- Limit to maximum 3 clarification markers
- Prioritize by impact: scope > security > UX > technical
- Ask user for their choice (provide 2-3 options)
- Update spec with user's answer

## Output Format

The generated specification should follow [spec-template.md](../../Template/spec-template.md):

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "[FEATURE DESCRIPTION]"

## User Scenarios & Testing

### User Story 1 - [Title] (Priority: P1)

[Describe journey]
**Why this priority**: [Value and business case]
**Independent Test**: [How to test independently]
**Acceptance Scenarios**:

1. Given... When... Then...
2. Given... When... Then...

[Additional user stories...]

## Requirements

### Functional Requirements

- **FR-001**: System MUST...
- **FR-002**: System MUST...
  [...]

### Key Entities

- **[Entity]**: [Description]
- **[Entity]**: [Description]

## Success Criteria

### Measurable Outcomes

- **SC-001**: [Metric], e.g., "Users can complete task in under 2 minutes"
- **SC-002**: [Metric], e.g., "System handles 1000 concurrent users"
  [...]
```

## Quick Reference: Given-When-Then Examples

### Example 1: Create Resource

```
Given user is on the create page
When user fills form and clicks submit
Then system saves record and shows confirmation message
```

### Example 2: Search with Filters

```
Given user is viewing the list with 50 items
When user applies filter for "active" status
Then system shows only 12 active items
```

### Example 3: Permission Check

```
Given user has "viewer" role
When user tries to delete a record
Then system shows "access denied" message
```

### Example 4: Error Recovery

```
Given user submits form with missing required field
When user sees validation error
Then user can correct the field and resubmit
```

## Process Tips

- **Make informed guesses** on ambiguous items using industry standards and common patterns
- **Document assumptions** in the Assumptions section for clarity
- **Think like a tester** - Every requirement should be independently testable
- **Keep it concise** - Avoid implementation details that cloud the business intent
- **Validate completeness** - After drafting, verify no [NEEDS CLARIFICATION] markers remain (max 3)
- **Use the template** - Always structure output using spec-template.md format

## Common Pitfalls to Avoid

❌ Implementation details in requirements ("Use React for UI")
❌ Vague acceptance criteria ("System should work well")
❌ Technology-specific success metrics ("API response < 200ms")
❌ Non-independent user stories (can't test one without others)
❌ Missing Given-When-Then format in scenarios
❌ More than 3 clarification markers
❌ Success criteria without measurable values

## Bundled References

This skill includes tested pattern libraries to accelerate spec creation:

### 1. **[references/acceptance-scenarios.md](references/acceptance-scenarios.md)**

Reference tested Given-When-Then patterns for common scenarios:

- **When to use**: Writing acceptance scenarios for user stories
- **Contents**: 40+ patterns covering CRUD, search, permissions, validation, errors, status transitions, multi-user interactions, bulk operations
- **Example**: Looking for patterns on "how to write an acceptance scenario for permission denial"

### 2. **[references/requirements-patterns.md](references/requirements-patterns.md)**

Reference functional requirement templates and best practices:

- **When to use**: Defining functional requirements section
- **Contents**: FR templates for CRUD, data persistence, search, permissions, validation, UI, notifications, integrations, data management
- **Example**: Need template for "system MUST support searching by..."

### 3. **[references/success-criteria.md](references/success-criteria.md)**

Reference measurable success criteria patterns and anti-patterns:

- **When to use**: Creating success criteria section
- **Contents**: SC templates for performance, UX, quality, business metrics, adoption, reliability
- **Example**: Need metrics for "interview completion rate" or "system uptime"
