---
name: user-story-writer
description: |
  Write well-structured user stories with acceptance criteria.
  Creates testable, implementable user stories following best practices.
---

# User Story Writer Skill

## Purpose
Create clear, actionable user stories that developers can implement and testers can verify.

## User Story Format

### Standard Format
```markdown
## US-[ID]: [Short Title]

**As a** [type of user]
**I want** [goal/desire]
**So that** [benefit/value]

### Description
[Additional context and details about the story]

### Acceptance Criteria
```gherkin
Given [context/precondition]
When [action/trigger]
Then [expected outcome]
```

### Technical Notes
[Implementation hints, API contracts, or technical considerations]

### UI/UX Notes
[Design references, interaction details, or mockup links]

### Dependencies
- Depends on: [US-XX, US-YY]
- Blocks: [US-ZZ]

### Estimation
- Story Points: [1, 2, 3, 5, 8, 13]
- T-Shirt Size: [S, M, L, XL]
```

## Examples

### Example 1: Authentication
```markdown
## US-001: User Login

**As a** registered user
**I want** to log in with my email and password
**So that** I can access my personal dashboard

### Acceptance Criteria
```gherkin
Given I am on the login page
When I enter valid email and password
Then I am redirected to the dashboard
And I see a welcome message with my name

Given I am on the login page
When I enter invalid credentials
Then I see an error message "Invalid email or password"
And I remain on the login page

Given I am logged in
When I refresh the page
Then I remain logged in
```

### Technical Notes
- Use JWT for authentication
- Token expiry: 15 minutes
- Refresh token: 7 days
- Rate limit: 5 attempts per 15 minutes

### Dependencies
- Depends on: US-000 (User registration)
```

### Example 2: CRUD Operation
```markdown
## US-010: Create New Task

**As a** project member
**I want** to create a new task
**So that** I can track work that needs to be done

### Acceptance Criteria
```gherkin
Given I am on the project board
When I click "Add Task"
Then a modal opens with a task creation form

Given I have filled in required fields (title)
When I click "Create"
Then the task is created
And I see the task on the board
And I see a success notification

Given I have not filled in required fields
When I click "Create"
Then I see validation errors
And the task is not created

Given I am creating a task
When I click outside the modal or press Escape
Then the modal closes
And no task is created
```

### UI/UX Notes
- Modal should be centered
- Required fields: Title
- Optional fields: Description, Due date, Assignee, Priority
- Auto-focus on title field
```

## Writing Good User Stories

### INVEST Criteria
- **I**ndependent: Can be developed separately
- **N**egotiable: Details can be discussed
- **V**aluable: Delivers user value
- **E**stimable: Can estimate effort
- **S**mall: Fits in a sprint
- **T**estable: Clear pass/fail criteria

### Good vs Bad Stories

**Bad:**
- "User can use the system" (too vague)
- "Implement caching" (technical task, not user value)
- "Build entire checkout flow" (too large)

**Good:**
- "As a shopper, I want to add items to my cart so I can purchase multiple items at once"
- "As an admin, I want to export user data as CSV so I can analyze it in Excel"

## Acceptance Criteria Guidelines

### Given-When-Then Structure
- **Given**: Setup/preconditions
- **When**: Action performed
- **Then**: Expected outcome

### Include:
- Happy path (success case)
- Error cases
- Edge cases
- Validation rules
- State changes

### Avoid:
- Technical implementation details
- Vague outcomes ("works correctly")
- Missing error handling

## Story Breakdown

### Epic → Feature → Story

```
Epic: User Authentication
├── Feature: User Registration
│   ├── US-001: Register with email
│   ├── US-002: Email verification
│   └── US-003: Profile completion
├── Feature: User Login
│   ├── US-010: Login with credentials
│   ├── US-011: Password reset
│   └── US-012: Remember me
└── Feature: Session Management
    ├── US-020: Auto logout
    └── US-021: Multi-device sessions
```

## Quality Checklist

- [ ] Follows standard format
- [ ] Written from user perspective
- [ ] Provides clear value
- [ ] Has testable acceptance criteria
- [ ] Small enough for one sprint
- [ ] Dependencies identified
- [ ] Technical notes for complex items
