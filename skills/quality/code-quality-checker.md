---
name: code-quality-checker
description: |
  Review code for quality issues, best practices, and maintainability.
  Identifies problems and suggests improvements.
---

# Code Quality Checker Skill

## Purpose
Analyze code for quality issues and provide actionable improvement suggestions.

## Quality Dimensions

| Dimension | What to Check |
|-----------|---------------|
| Correctness | Does it work as intended? |
| Readability | Is it easy to understand? |
| Maintainability | Is it easy to modify? |
| Performance | Is it efficient? |
| Security | Is it safe? |
| Testability | Is it easy to test? |

## Code Review Checklist

### General
- [ ] Code does what it's supposed to
- [ ] No obvious bugs or logic errors
- [ ] Error cases are handled
- [ ] No hardcoded values that should be configurable
- [ ] No commented-out code
- [ ] No TODO comments without tracking

### Naming
- [ ] Variable names are descriptive
- [ ] Function names describe what they do
- [ ] Consistent naming conventions
- [ ] No abbreviations that aren't clear

### Functions
- [ ] Functions do one thing
- [ ] Functions are reasonably short (<30 lines)
- [ ] Parameters are reasonable in number (<5)
- [ ] No side effects where not expected
- [ ] Return types are consistent

### TypeScript
- [ ] No `any` types (use `unknown` if needed)
- [ ] Types are specific, not overly broad
- [ ] Interfaces/types are well-defined
- [ ] Generics used appropriately
- [ ] Strict mode enabled

### React Components
- [ ] Components are focused (single responsibility)
- [ ] Props interface is well-defined
- [ ] Hooks follow rules of hooks
- [ ] No unnecessary re-renders
- [ ] Keys are stable and unique
- [ ] Effects have proper dependencies
- [ ] Cleanup in useEffect when needed

### Error Handling
- [ ] Errors are caught and handled
- [ ] Error messages are helpful
- [ ] Errors don't expose sensitive info
- [ ] Async errors are handled
- [ ] Validation errors are user-friendly

## Common Issues

### 1. Magic Numbers/Strings
```typescript
// Bad
if (status === 1) { ... }
setTimeout(fn, 86400000);

// Good
const STATUS_ACTIVE = 1;
const ONE_DAY_MS = 24 * 60 * 60 * 1000;

if (status === STATUS_ACTIVE) { ... }
setTimeout(fn, ONE_DAY_MS);
```

### 2. Deep Nesting
```typescript
// Bad
function process(data) {
  if (data) {
    if (data.user) {
      if (data.user.isActive) {
        if (data.user.hasPermission) {
          // Do something
        }
      }
    }
  }
}

// Good
function process(data) {
  if (!data?.user?.isActive || !data.user.hasPermission) {
    return;
  }
  // Do something
}
```

### 3. Long Functions
```typescript
// Bad: 100+ line function doing everything

// Good: Break into smaller functions
function processOrder(order: Order) {
  validateOrder(order);
  const total = calculateTotal(order);
  const payment = processPayment(order, total);
  sendConfirmation(order, payment);
  return payment;
}
```

### 4. Implicit Type Coercion
```typescript
// Bad
if (value) { ... }  // Empty string, 0 are falsy
if (array.length) { ... }

// Good
if (value !== null && value !== undefined) { ... }
if (array.length > 0) { ... }
```

### 5. Missing Cleanup
```typescript
// Bad
useEffect(() => {
  const interval = setInterval(fetchData, 5000);
  // Missing cleanup!
}, []);

// Good
useEffect(() => {
  const interval = setInterval(fetchData, 5000);
  return () => clearInterval(interval);
}, []);
```

### 6. Prop Drilling
```typescript
// Bad: Passing props through many levels
<App user={user}>
  <Layout user={user}>
    <Header user={user}>
      <UserMenu user={user} />

// Good: Use context
const UserContext = createContext<User | null>(null);

function App() {
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}
```

### 7. Unnecessary State
```typescript
// Bad: Derived state
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// When items change, must also update total

// Good: Compute derived values
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

### 8. Missing Error Boundaries
```tsx
// Bad: Error crashes entire app

// Good: Error boundaries
<ErrorBoundary fallback={<ErrorPage />}>
  <RiskyComponent />
</ErrorBoundary>
```

## Review Report Format

```markdown
# Code Review: [File/Component Name]

## Summary
Brief overview of the code and overall assessment.

## Critical Issues 🔴
Issues that must be fixed before merge.

### Issue 1: [Title]
**Location:** `file.ts:42`
**Problem:** Description of the issue.
**Suggestion:** How to fix it.
```typescript
// Before
...
// After
...
```

## Major Issues 🟠
Significant issues that should be addressed.

## Minor Issues 🟡
Code smells and improvements.

## Suggestions 🔵
Optional improvements and best practices.

## Positive Observations 💚
Good practices found in the code.

## Checklist
- [ ] Correctness verified
- [ ] No security issues
- [ ] Performance acceptable
- [ ] Tests adequate
- [ ] Documentation sufficient
```

## Linting Configuration

### ESLint
```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "next/core-web-vitals"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "react-hooks/exhaustive-deps": "warn",
    "no-console": "warn"
  }
}
```

### Prettier
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

## Quality Checklist

- [ ] No linting errors
- [ ] No TypeScript errors
- [ ] Code is formatted consistently
- [ ] Complex logic is commented
- [ ] Public APIs are documented
- [ ] Tests cover critical paths
- [ ] No security vulnerabilities
- [ ] Performance is acceptable
