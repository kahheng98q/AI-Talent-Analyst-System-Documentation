---
name: skill-update-suggester
description: Analyzes existing skills and generates targeted suggestions for improvements, enhancements, and optimizations. Use when asking to "review skills for improvements", "suggest skill updates", "analyze skill effectiveness", "find gaps in skills", or "optimize existing skills".
license: Complete terms in LICENSE.txt
---

# Skill Update Suggester

Systematically analyzes existing skills to identify improvement opportunities, optimization possibilities, and missing capabilities that could enhance their effectiveness.

## When to Use This Skill

- User asks to review existing skills for improvements
- User wants to identify gaps or limitations in current skills
- User requests optimization suggestions for skill effectiveness
- User needs to prioritize skill enhancement work
- User wants to analyze skill coverage across their organization
- User asks "what skills need updating?" or "how can we improve our skills?"

## Skill Update Analysis Process

### 1. Assess Current Skill State

Evaluate each skill across these dimensions:

**Coverage & Scope**
- Does the skill address the full domain or are there gaps?
- Are edge cases and variations documented?
- Is the skill scope clearly defined in the description?
- Does the skill cover both common and advanced use cases?

**Audience & Clarity**
- Is the skill's trigger condition clear? (When should it be used?)
- Can the target audience easily understand when to invoke it?
- Are prerequisite skills or knowledge documented?
- Is the language concise and jargon-free?

**Documentation Quality**
- Is the SKILL.md well-organized and scannable?
- Are examples provided for key workflows?
- Are warnings or gotchas documented?
- Is there clear guidance on when NOT to use the skill?

**Usability & Workflow**
- Are multi-step processes clearly broken down?
- Are decision trees provided where applicable?
- Are templates or starting points available?
- Does the skill integrate with other skills?

**Technical Depth**
- Does the skill provide sufficient technical guidance?
- Are configuration options documented?
- Are troubleshooting sections included?
- Are performance or reliability considerations noted?

### 2. Categorize Suggestions

Organize improvement suggestions into:

**High Priority (Core Functionality)**
- Gaps that prevent the skill from working correctly
- Missing critical use cases
- Unclear trigger conditions
- Broken workflows or outdated information

**Medium Priority (Enhancement)**
- Additional use cases that would broaden skill value
- Improved clarity or organization
- Missing examples or templates
- Better integration with other skills

**Low Priority (Polish)**
- Minor wording improvements
- Additional nice-to-have features
- Enhanced formatting or presentation
- Extended examples or variations

### 3. Generate Actionable Suggestions

For each suggestion, provide:

- **What**: The specific improvement or addition needed
- **Why**: Business or technical justification
- **How**: Concrete steps to implement the suggestion
- **Impact**: Expected benefit to users or organization
- **Effort**: Estimated complexity (Quick/Medium/Complex)

### 4. Create Prioritized Update Plan

Organize suggestions into phases:

- **Phase 1**: High-priority updates that unlock other work
- **Phase 2**: Medium-priority enhancements that increase value
- **Phase 3**: Low-priority polish and optimization

## Suggestion Template Structure

### Skill-Level Suggestions

```
## [Skill Name]

### Current State
- Brief description of current skill
- Key strengths
- Known limitations

### Suggested Updates

#### 1. [Suggestion Title]
- **Priority**: High/Medium/Low
- **Category**: Coverage/Clarity/Documentation/Usability/Technical
- **Current Gap**: What's missing or unclear
- **Proposed Change**: What should be added/modified
- **Implementation Steps**: Concrete actions
- **Expected Impact**: Benefits to users
- **Estimated Effort**: Quick/Medium/Complex

#### 2. [Next Suggestion]
...
```

## Cross-Skill Analysis

When analyzing multiple skills, also identify:

- **Redundancy**: Skills that overlap and could be consolidated
- **Integration Opportunities**: Skills that could reference each other
- **Dependency Gaps**: Missing prerequisite skills
- **Coverage Analysis**: Domains well-covered vs. underserved
- **Maturity Levels**: Which skills are mature vs. need development

## Output Format

Generate a structured markdown file with:

1. **Executive Summary**: Overall skill portfolio health
2. **Skill-by-Skill Analysis**: Detailed suggestions for each skill
3. **Cross-Skill Recommendations**: Portfolio-level improvements
4. **Implementation Roadmap**: Phased approach to addressing suggestions
5. **Success Metrics**: How to measure improvement effectiveness

## Tips for Effective Analysis

- **Be specific**: Avoid vague suggestions; reference exact sections or workflows
- **Consider context**: Understand who uses the skill and their needs
- **Balance completeness**: Suggest useful additions without bloat
- **Prioritize ruthlessly**: Focus on high-impact improvements first
- **Link to requirements**: Connect suggestions to user needs and business goals
- **Enable action**: Make suggestions immediately implementable
