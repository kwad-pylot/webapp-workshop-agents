---
name: task-decomposer
description: |
  Break down features and requirements into implementable tasks.
  Creates structured work breakdown for development.
---

# Task Decomposer Skill

## Purpose
Transform high-level features into specific, actionable development tasks.

## Decomposition Process

### 1. Understand the Feature
- What is the user trying to accomplish?
- What are the inputs and outputs?
- What systems are involved?

### 2. Identify Layers
- Database changes needed
- Backend logic required
- Frontend components needed
- Integration points

### 3. Break Down by Layer

```markdown
## Feature: [Feature Name]

### Database Tasks
- [ ] DB-001: Create/modify schema
- [ ] DB-002: Write migration
- [ ] DB-003: Create seed data

### Backend Tasks
- [ ] BE-001: Create/modify models
- [ ] BE-002: Implement service logic
- [ ] BE-003: Create API endpoints
- [ ] BE-004: Add validation
- [ ] BE-005: Write tests

### Frontend Tasks
- [ ] FE-001: Create components
- [ ] FE-002: Implement state management
- [ ] FE-003: Connect to API
- [ ] FE-004: Add styling
- [ ] FE-005: Write tests

### Integration Tasks
- [ ] INT-001: End-to-end testing
- [ ] INT-002: Documentation
```

## Task Template

```markdown
## Task: [ID] - [Title]

**Feature:** [Parent feature]
**Layer:** Database | Backend | Frontend | Integration
**Estimated Effort:** [1-8 hours or S/M/L]

### Description
[What needs to be done]

### Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

### Technical Details
- Files to create/modify: [list]
- Dependencies: [list]
- API contracts: [if applicable]

### Dependencies
- Blocked by: [Task IDs]
- Blocks: [Task IDs]
```

## Example Decomposition

### Feature: User Authentication

```markdown
## Feature: User Authentication

### Overview
Users can register, login, logout, and reset password.

### Database Tasks

#### DB-001: Create users table
- **Effort:** S (1-2 hours)
- **Description:** Create users table with email, password_hash, created_at
- **Acceptance:**
  - [ ] Users table exists with correct columns
  - [ ] Email has unique constraint
  - [ ] Indexes on email column

#### DB-002: Create sessions table (if needed)
- **Effort:** S
- **Description:** Table for refresh tokens
- **Depends on:** DB-001

### Backend Tasks

#### BE-001: User model and service
- **Effort:** M (2-4 hours)
- **Description:** Create User model and UserService with CRUD operations
- **Depends on:** DB-001
- **Acceptance:**
  - [ ] User model with TypeScript types
  - [ ] UserService with create, findByEmail, findById
  - [ ] Password hashing on create

#### BE-002: Auth service
- **Effort:** M
- **Description:** Implement authentication logic
- **Depends on:** BE-001
- **Acceptance:**
  - [ ] Login method (verify password, generate tokens)
  - [ ] Register method (validate, create user)
  - [ ] Password reset flow

#### BE-003: Auth API endpoints
- **Effort:** M
- **Description:** REST endpoints for auth
- **Depends on:** BE-002
- **Acceptance:**
  - [ ] POST /api/auth/register
  - [ ] POST /api/auth/login
  - [ ] POST /api/auth/logout
  - [ ] POST /api/auth/forgot-password
  - [ ] POST /api/auth/reset-password

#### BE-004: Auth middleware
- **Effort:** S
- **Description:** JWT verification middleware
- **Depends on:** BE-002
- **Acceptance:**
  - [ ] Middleware extracts and verifies JWT
  - [ ] Attaches user to request
  - [ ] Returns 401 for invalid tokens

#### BE-005: Auth tests
- **Effort:** M
- **Description:** Unit and integration tests
- **Depends on:** BE-003, BE-004
- **Acceptance:**
  - [ ] Unit tests for AuthService
  - [ ] Integration tests for endpoints
  - [ ] Test invalid inputs and edge cases

### Frontend Tasks

#### FE-001: Auth components
- **Effort:** M
- **Description:** Login, Register, ForgotPassword forms
- **Acceptance:**
  - [ ] LoginForm component with validation
  - [ ] RegisterForm component with validation
  - [ ] ForgotPasswordForm component

#### FE-002: Auth state management
- **Effort:** S
- **Description:** Global auth state (user, token)
- **Acceptance:**
  - [ ] Auth context/store
  - [ ] Persist token in localStorage
  - [ ] Auto-restore session on load

#### FE-003: Protected routes
- **Effort:** S
- **Description:** Route protection for authenticated pages
- **Depends on:** FE-002
- **Acceptance:**
  - [ ] Redirect to login if not authenticated
  - [ ] Redirect to dashboard after login

#### FE-004: Auth pages
- **Effort:** M
- **Description:** Login, Register, ForgotPassword pages
- **Depends on:** FE-001, FE-002
- **Acceptance:**
  - [ ] /login page with form
  - [ ] /register page with form
  - [ ] /forgot-password page
  - [ ] Proper routing between pages

### Summary
| Layer | Tasks | Total Effort |
|-------|-------|--------------|
| Database | 2 | S+S = ~3h |
| Backend | 5 | S+M+M+S+M = ~12h |
| Frontend | 4 | M+S+S+M = ~8h |
| **Total** | **11** | **~23h** |
```

## Sizing Guidelines

| Size | Hours | Description |
|------|-------|-------------|
| XS | <1 | Trivial change, config update |
| S | 1-2 | Single file, straightforward |
| M | 2-4 | Multiple files, some complexity |
| L | 4-8 | Feature slice, several components |
| XL | 8+ | Should be broken down further |

## Quality Checklist

- [ ] Tasks are small enough (< 8 hours)
- [ ] Dependencies are explicit
- [ ] Acceptance criteria are testable
- [ ] No tasks are vague
- [ ] All layers covered
- [ ] Tests included in breakdown
