# Acceptance Scenario Patterns

This reference provides tested Given-When-Then patterns for common specification scenarios.

## Table of Contents

1. [CRUD Operations](#crud-operations)
2. [Search & Filtering](#search--filtering)
3. [Permission & Access Control](#permission--access-control)
4. [Data Validation](#data-validation)
5. [Error Handling](#error-handling)
6. [Status Transitions](#status-transitions)
7. [Multi-User Interactions](#multi-user-interactions)
8. [Bulk Operations](#bulk-operations)

---

## CRUD Operations

### Create (Happy Path)

```
Given user is on the create form
When user fills all required fields and clicks submit
Then system saves the record and displays success message with new record ID
```

### Create (Missing Required Field)

```
Given user is on the create form with empty required field
When user attempts to submit
Then system shows validation error for missing field and keeps form open
```

### Create (Duplicate Prevention)

```
Given a record with email "test@example.com" already exists
When user tries to create another record with same email
Then system rejects creation and shows "email already exists" error
```

### Read (View Details)

```
Given user is viewing a list of records
When user clicks on a specific record
Then system displays all record details on detail view
```

### Update (Successful)

```
Given user has open record with initial value "A"
When user changes value to "B" and clicks save
Then system persists change and shows confirmation message
```

### Update (Concurrent Edit Conflict)

```
Given user A and user B both have the same record open
When user A saves changes and user B tries to save conflicting changes
Then system alerts user B of conflict and prevents overwrite
```

### Delete (With Confirmation)

```
Given user is viewing a record
When user clicks delete and confirms in dialog
Then system removes record and returns to list view
```

### Delete (Prevent Orphan Records)

```
Given a company has 5 jobs associated with it
When user tries to delete the company
Then system shows warning that deletion will affect 5 jobs and requires confirmation
```

---

## Search & Filtering

### Simple Search

```
Given user is on list view with 100 items
When user enters "John" in search box and presses enter
Then system displays only 5 items matching "John"
```

### Multiple Filters (AND Logic)

```
Given user is viewing job list with 50 total jobs
When user filters for status="active" AND competency="Python"
Then system shows 8 jobs matching both criteria
```

### Filter Removal

```
Given user has 3 active filters applied showing 12 results
When user removes one filter
Then system updates results to show all matching items for remaining filters
```

### Empty Search Results

```
Given user searches for "impossible-value-xyz"
When system processes search
Then system displays empty state message "No results found"
```

### Filter Persistence

```
Given user applies filters and navigates to detail view
When user returns to list view
Then filters remain applied
```

---

## Permission & Access Control

### Permitted Action (Happy Path)

```
Given user has "manager" role
When user attempts to create a new job
Then system allows action and processes request
```

### Denied Action

```
Given user has "viewer" role
When user attempts to delete a record
Then system shows "Access Denied - insufficient permissions" error
```

### Role-Based Feature Visibility

```
Given user with "admin" role logs in
When user views the left sidebar
Then user sees "Settings" and "Reports" menu items
```

```
Given user with "candidate" role logs in
When user views the left sidebar
Then user only sees "My Interviews" and "Profile" menu items
```

### Permission Inheritance

```
Given user has "interviewer" role within "Company A" group
When user attempts to view candidates from "Company A"
Then system allows access
```

```
Given user has "interviewer" role within "Company A" group
When user attempts to view candidates from "Company B"
Then system denies access with "You don't have permission for this company" message
```

---

## Data Validation

### Email Format Validation

```
Given user is creating an account
When user enters invalid email "not-an-email"
Then system shows "Please enter a valid email address" error
```

### Required Field Validation

```
Given user is creating a job posting
When user leaves "job title" field empty and clicks save
Then system highlights field in red and shows "This field is required" message
```

### Length Validation

```
Given system allows passwords between 8-20 characters
When user enters password with 5 characters
Then system rejects with "Password must be at least 8 characters"
```

### Unique Value Validation

```
Given "john.doe@company.com" is already registered
When new user tries to register with same email
Then system shows "This email is already in use" error
```

### Cross-Field Validation

```
Given user enters start_date = Jan 15 and end_date = Jan 10
When user clicks save
Then system shows "End date must be after start date" error
```

---

## Error Handling

### Network Error Recovery

```
Given user is submitting a form when internet connection drops
When system attempts to save record
Then system shows "Connection lost - please try again" message and preserves form data
```

### Timeout Handling

```
Given user initiates a long operation (e.g., bulk import)
When operation takes more than 30 seconds
Then system shows progress indicator and allows user to continue waiting or cancel
```

### Invalid State Error

```
Given interview is in "completed" state
When user tries to restart interview
Then system shows "Cannot restart completed interview" error
```

### Cascading Failures

```
Given external AI service is unavailable
When user tries to conduct interview that needs AI scoring
Then system shows "Interview service temporarily unavailable - try again later" and suggests alternative
```

### Graceful Degradation

```
Given some features require AI service that is down
When user accesses system
Then system shows core features normally but disables AI-dependent features with explanation
```

---

## Status Transitions

### Valid Transition

```
Given job status is "draft"
When hiring manager clicks "publish"
Then system changes status to "active" and sends notifications to all recruiters
```

### Invalid Transition Prevention

```
Given job status is "completed"
When user tries to change status to "draft"
Then system prevents transition and shows "Cannot unpublish completed job"
```

### Status History Tracking

```
Given job has transitioned: draft → active → on-hold
When user views job history
Then system shows all 3 status changes with timestamps and who made each change
```

### Conditional Transition

```
Given interview status is "in-progress"
When all responses are submitted
Then system automatically transitions status to "completed"
```

---

## Multi-User Interactions

### Real-Time Update Notification

```
Given user A is viewing a candidate profile
When user B updates the candidate's phone number
Then user A sees real-time notification of update
```

### Optimistic Locking

```
Given user A and B both open same interview
When user A submits responses first
Then user A's submission succeeds
```

```
Given user A and B both open same interview
When user A submits responses and user B tries to submit conflicting responses
Then user B's submission is rejected with notification that interview was modified
```

### Collaborative Workflow

```
Given user A (recruiter) and user B (manager) are reviewing candidate
When user A adds comment "Strong Python skills"
Then user B sees comment and can reply to create discussion thread
```

---

## Bulk Operations

### Bulk Create Success

```
Given user has CSV file with 100 valid job records
When user uploads file via import interface
Then system creates all 100 jobs and shows summary "100 jobs imported successfully"
```

### Bulk Create With Partial Failure

```
Given user has CSV file with 100 records where 5 have invalid data
When user uploads file
Then system imports 95 records and shows summary "95 successful, 5 failed" with details on failures
```

### Bulk Update Status

```
Given user has selected 10 jobs with status "draft"
When user clicks bulk action "Publish All"
Then system changes all 10 to "active" and shows confirmation
```

### Bulk Delete Confirmation

```
Given user has selected 15 interviews to delete
When user clicks delete
Then system shows confirmation "Are you sure you want to delete 15 interviews?" with cancel/confirm options
```

---

## Anti-Patterns to Avoid

❌ "Given system is working normally" (vague initial state)
❌ "When user clicks buttons" (vague action)
❌ "Then system responds" (vague outcome)

✅ "Given job list shows 50 active jobs" (specific state)
✅ "When user applies "Python" competency filter" (specific action)
✅ "Then system displays 12 jobs tagged with Python" (specific, verifiable outcome)
