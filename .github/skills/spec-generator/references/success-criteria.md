# Success Criteria Patterns

This reference provides templates for defining measurable, technology-agnostic success criteria.

## Table of Contents

1. [Performance Metrics](#performance-metrics)
2. [User Experience Metrics](#user-experience-metrics)
3. [Quality & Accuracy Metrics](#quality--accuracy-metrics)
4. [Business Metrics](#business-metrics)
5. [Adoption & Engagement](#adoption--engagement)
6. [Reliability & Availability](#reliability--availability)

---

## Performance Metrics

### Response Time

```
- **SC-001**: Users can complete [action] within [X] minutes
- **SC-002**: System returns search results within [X] seconds for up to [Y] records
- **SC-003**: Page loads within [X] seconds on standard internet connection
- **SC-004**: File upload completes within [X] minutes for [Y] MB file
- **SC-005**: Bulk import processes [X] records within [Y] minutes
```

**Examples:**

- "Users can complete account creation in under 2 minutes"
- "System returns search results within 1 second for queries on up to 10,000 records"
- "Interview report generates within 30 seconds"

### Throughput

```
- **SC-006**: System handles [X] concurrent users without degradation
- **SC-007**: System processes [X] transactions per second
- **SC-008**: System can import [X] records per minute
- **SC-009**: System supports [X] simultaneous interviews
```

**Examples:**

- "System handles 1,000 concurrent users without degradation"
- "System processes 100 interview submissions per minute"
- "System can import 10,000 records per minute"

---

## User Experience Metrics

### Task Completion

```
- **SC-010**: [X]% of users complete [task] successfully on first attempt
- **SC-011**: Users require fewer than [X] steps to complete [task]
- **SC-012**: Users require fewer than [X] clicks to reach [feature]
- **SC-013**: [X]% of users find [feature] without assistance
```

**Examples:**

- "85% of users complete interview successfully on first attempt"
- "Users complete job posting in fewer than 5 steps"
- "90% of users find filter functionality within 2 clicks"

### Satisfaction & Feedback

```
- **SC-014**: Average user satisfaction score above [X]/5 for [feature]
- **SC-015**: [X]% of users rate [feature] as "useful" or higher
- **SC-016**: Support tickets related to [issue] reduced by [X]%
- **SC-017**: Net Promoter Score (NPS) for feature above [X]
```

**Examples:**

- "Average user satisfaction score above 4/5 for interview experience"
- "75% of users rate interview interface as 'intuitive' or higher"
- "Support tickets related to permission errors reduced by 50%"

### Error Rate

```
- **SC-018**: Failed submission rate below [X]%
- **SC-019**: Form abandonment rate below [X]%
- **SC-020**: [X]% of uploads complete without errors
- **SC-021**: Data validation error rate below [X]%
```

**Examples:**

- "Job posting failure rate below 1%"
- "Form abandonment rate below 5%"
- "90% of bulk imports complete without errors"

---

## Quality & Accuracy Metrics

### Data Quality

```
- **SC-022**: [X]% of imported records are valid and processable
- **SC-023**: Duplicate detection catches [X]% of attempted duplicates
- **SC-024**: All required fields populated for [X]% of records
- **SC-025**: Data accuracy validates against external source at [X]% match
```

**Examples:**

- "95% of imported job records are valid and processable"
- "Duplicate detection catches 99% of attempted duplicate job submissions"
- "100% of required fields populated for completed interviews"

### Correctness

```
- **SC-026**: Competency scoring accuracy within [X]% of human assessment
- **SC-027**: Search results relevance score above [X]% for top results
- **SC-028**: Recommendation accuracy above [X]%
- **SC-029**: Classification accuracy above [X]% for AI-generated categories
```

**Examples:**

- "Competency scoring accuracy within 5% of human assessment"
- "Search results relevance score above 80% for top 3 results"
- "Resume parsing accuracy above 95%"

---

## Business Metrics

### Adoption

```
- **SC-030**: Feature used by [X]% of eligible users within [Y] weeks
- **SC-031**: [X]% of new users complete onboarding successfully
- **SC-032**: Daily active user rate above [X]% of total registered
- **SC-033**: Feature adoption increases by [X]% month-over-month
```

**Examples:**

- "Feature used by 60% of eligible users within 2 weeks"
- "80% of new users complete interview onboarding successfully"
- "Daily active user rate above 40% of total registered users"

### Conversion & Workflow Completion

```
- **SC-034**: Interview completion rate above [X]%
- **SC-035**: Job posting-to-hire conversion rate above [X]%
- **SC-036**: User signup-to-first-interview rate above [X]%
- **SC-037**: Pipeline advancement rate above [X]% per stage
```

**Examples:**

- "Interview completion rate above 80%"
- "Job posting to first interview rate above 60%"
- "Candidate pipeline advancement rate above 75% per stage"

### Efficiency Gains

```
- **SC-038**: Time-to-hire reduced by [X]%
- **SC-039**: Recruiter time savings of [X] hours per hire
- **SC-040**: Processing time reduced by [X]% compared to manual workflow
- **SC-041**: Cost per hire reduced by [X]%
```

**Examples:**

- "Time-to-hire reduced by 30% compared to previous process"
- "Recruiter saves 2 hours per hire with new workflow"
- "Bulk import processing time reduced by 70%"

---

## Adoption & Engagement

### Usage Frequency

```
- **SC-042**: Average user session length above [X] minutes
- **SC-043**: Users return to platform [X] times per week
- **SC-044**: Active feature usage above [X]% for new users
- **SC-045**: Feature usage growth of [X]% month-over-month
```

**Examples:**

- "Average user session length above 10 minutes"
- "Power users return to platform 3+ times per week"
- "80% of new users use key features within first week"

### Retention

```
- **SC-046**: [X]% of users remain active after 30 days
- **SC-047**: Churn rate below [X]% per month
- **SC-048**: [X]% of companies maintain subscription after trial period
```

**Examples:**

- "80% of users remain active after 30 days"
- "Monthly churn rate below 5%"
- "70% of companies convert from trial to paid subscription"

---

## Reliability & Availability

### System Uptime

```
- **SC-049**: System availability at [X]% uptime (e.g., 99.9%)
- **SC-050**: Feature availability at [X]% uptime during business hours
- **SC-051**: Scheduled maintenance window limited to [X] minutes per month
```

**Examples:**

- "System maintains 99.9% uptime"
- "Interview service availability 99.95% during business hours"
- "Scheduled maintenance limited to 60 minutes per month"

### Error Recovery

```
- **SC-052**: Failed operations retry automatically up to [X] times
- **SC-053**: System recovers from errors within [X] minutes
- **SC-054**: Data loss occurs in less than 1 in [X] failure scenarios
- **SC-055**: User session recovery after network interruption succeeds [X]% of time
```

**Examples:**

- "System recovers from failures within 5 minutes"
- "Session recovery succeeds 95% of the time after network interruption"
- "Failed interviews can be resumed without data loss"

---

## How to Structure Success Criteria

Every specification should include 3-5 success criteria balanced across:

| Metric Type         | Weight      | Example                       |
| ------------------- | ----------- | ----------------------------- |
| **Performance**     | Must-have   | Response time, throughput     |
| **Quality**         | Must-have   | Accuracy, error rate          |
| **User Experience** | Should-have | Task completion, satisfaction |
| **Business**        | Should-have | Adoption, conversion          |
| **Reliability**     | Should-have | Uptime, availability          |

### Anti-Patterns to Avoid

❌ "System performs well" (vague, unmeasurable)
✅ "System returns results within 1 second"

❌ "Users are satisfied" (subjective)
✅ "80% of users rate feature as 'useful' or higher"

❌ "API response time is under 200ms" (implementation detail)
✅ "Users see results instantly (within 1 second)"

❌ "Database handles 1000 TPS" (implementation focused)
✅ "System processes 100 interview submissions per minute"

### Technology-Agnostic Framing

| ❌ Technical                            | ✅ User-Focused                                    |
| --------------------------------------- | -------------------------------------------------- |
| "Redis cache hit rate 80%"              | "Search results display in under 1 second"         |
| "GraphQL query resolves in 150ms"       | "User sees details instantly"                      |
| "Database handles 10k conn/s"           | "System handles 5,000 concurrent users"            |
| "React components render efficiently"   | "Page loads in under 2 seconds"                    |
| "Elasticsearch returns top-10 in 100ms" | "Users see first search result within 0.5 seconds" |
