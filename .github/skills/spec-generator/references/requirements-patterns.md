# Functional Requirements Patterns

This reference provides templates for writing clear, testable functional requirements.

## Table of Contents

1. [CRUD Operations](#crud-operations)
2. [Data Persistence](#data-persistence)
3. [Search & Filtering](#search--filtering)
4. [Permissions & Authorization](#permissions--authorization)
5. [Validation & Error Handling](#validation--error-handling)
6. [User Interface](#user-interface)
7. [Notifications & Alerts](#notifications--alerts)
8. [Integration & External Services](#integration--external-services)
9. [Data Management](#data-management)

---

## CRUD Operations

```
- **FR-001**: System MUST allow users to create new [resource] by filling required fields and clicking save
- **FR-002**: System MUST validate all required fields before allowing save
- **FR-003**: System MUST assign a unique identifier to each newly created [resource]
- **FR-004**: System MUST allow users to view [resource] details including all attributes
- **FR-005**: System MUST allow users to modify existing [resource] attributes
- **FR-006**: System MUST persist all changes to [resource] in permanent storage
- **FR-007**: System MUST allow users to delete [resource] with confirmation dialog
- **FR-008**: System MUST prevent deletion of [resource] if it has dependent records
```

---

## Data Persistence

```
- **FR-009**: System MUST store all user-created records indefinitely unless manually deleted
- **FR-010**: System MUST maintain complete history of all status changes with timestamp and actor
- **FR-011**: System MUST capture user who performed each action for audit purposes
- **FR-012**: System MUST enforce data consistency across all instances
- **FR-013**: System MUST provide data export in CSV format
```

---

## Search & Filtering

```
- **FR-014**: System MUST support searching by [specific fields] using text matching
- **FR-015**: System MUST support filtering by [specific attributes] with multiple simultaneous filters
- **FR-016**: System MUST combine multiple filters using AND logic
- **FR-017**: System MUST persist filter selection when user navigates and returns
- **FR-018**: System MUST support sorting results by relevant fields (ascending/descending)
- **FR-019**: System MUST display result count for current search/filter combination
```

---

## Permissions & Authorization

```
- **FR-020**: System MUST enforce role-based access control on all features
- **FR-021**: System MUST prevent users from accessing resources outside their organization
- **FR-022**: System MUST hide features from users lacking required permissions
- **FR-023**: System MUST log all permission denials for audit purposes
- **FR-024**: System MUST support multiple roles assigned to single user (additive permissions)
- **FR-025**: System MUST support custom permission groups at company level
- **FR-026**: System MUST allow company admin to assign users to permission groups
```

---

## Validation & Error Handling

```
- **FR-027**: System MUST validate [field] format before accepting input
- **FR-028**: System MUST prevent duplicate entries for [unique field]
- **FR-029**: System MUST provide specific error message for each validation failure
- **FR-030**: System MUST highlight invalid fields in the form for user correction
- **FR-031**: System MUST preserve form data after validation error
- **FR-032**: System MUST gracefully handle missing or corrupted data
- **FR-033**: System MUST prevent null/empty values in required fields
- **FR-034**: System MUST validate cross-field relationships (e.g., start_date < end_date)
```

---

## User Interface

```
- **FR-035**: System MUST display [resource] list with pagination (default 25 items per page)
- **FR-036**: System MUST provide breadcrumb navigation showing current location
- **FR-037**: System MUST indicate loading state during long operations
- **FR-038**: System MUST show success confirmation message after user action
- **FR-039**: System MUST display error messages in consistent location and format
- **FR-040**: System MUST provide "back" button to previous view with state preserved
- **FR-041**: System MUST display form validation errors before allowing submission
```

---

## Notifications & Alerts

```
- **FR-042**: System MUST notify [affected users] when [event] occurs
- **FR-043**: System MUST allow users to control notification preferences
- **FR-044**: System MUST send notifications within [X minutes] of event occurrence
- **FR-045**: System MUST include relevant context in notification message
- **FR-046**: System MUST provide notification history accessible to users
- **FR-047**: System MUST support in-app notifications and email notifications
```

---

## Integration & External Services

```
- **FR-048**: System MUST integrate with [external service] for [specific capability]
- **FR-049**: System MUST retry failed external calls up to [N] times with exponential backoff
- **FR-050**: System MUST gracefully degrade if external service is unavailable
- **FR-051**: System MUST log all external service calls for debugging
- **FR-052**: System MUST validate responses from external services before processing
- **FR-053**: System MUST handle timeout errors from external services within [X seconds]
```

---

## Data Management

```
- **FR-054**: System MUST support bulk import of [resource] via CSV file
- **FR-055**: System MUST validate all records in bulk import before processing any
- **FR-056**: System MUST provide detailed report of bulk import results (successes and failures)
- **FR-057**: System MUST support bulk delete with confirmation
- **FR-058**: System MUST support bulk status updates
- **FR-059**: System MUST maintain audit trail of all bulk operations
- **FR-060**: System MUST prevent duplicate records during import
```

---

## Common Requirement Categories

When defining requirements, ensure coverage of:

| Category           | Example Requirements                             |
| ------------------ | ------------------------------------------------ |
| **Functional**     | Users can create, read, update, delete resources |
| **Data**           | Data is stored, retrieved, validated, persisted  |
| **Access Control** | Users can only access authorized resources       |
| **Notifications**  | Users are informed of relevant events            |
| **Error Handling** | System handles failures gracefully               |
| **Performance**    | System responds within acceptable timeframes     |
| **Audit**          | System logs all significant actions              |
| **Integration**    | System works with external services              |

---

## Writing Better Requirements

### Use Specific Language

❌ "System should handle errors"
✅ "System MUST retry failed requests up to 3 times with exponential backoff"

❌ "Users can search for jobs"
✅ "System MUST support searching jobs by title, company, and location with partial text matching"

### Be Testable

❌ "System MUST be user-friendly"
✅ "System MUST allow users to complete account creation in under 2 minutes"

❌ "Performance should be good"
✅ "System MUST return search results within 1 second for queries on up to 10,000 records"

### Cover Edge Cases

Always consider:

- What if input is empty?
- What if user lacks permissions?
- What if external service fails?
- What if two users perform same action simultaneously?
- What if record doesn't exist?

### Distinguish MUST vs SHOULD

- **MUST** = Critical for feature to work (implement for MVP)
- **SHOULD** = Important but can be deferred (implement after MVP)
- **MAY** = Nice-to-have (future enhancement)
