---
name: library-researcher
description: |
  Research and evaluate libraries and packages for project needs.
  Compares options and provides recommendations with documentation.
---

# Library Researcher Skill

## Purpose
Research, evaluate, and document library choices for informed decision-making.

## Research Process

### 1. Define Requirements
- What problem needs solving?
- What features are must-have?
- What constraints exist (size, dependencies, license)?
- What's the team's familiarity?

### 2. Find Candidates
Sources to search:
- npm/PyPI/etc. package registries
- GitHub trending/topics
- "Awesome" lists
- Blog posts and comparisons
- Context7 documentation

### 3. Evaluate Options

| Criterion | Weight | Questions |
|-----------|--------|-----------|
| Functionality | High | Does it solve the problem? |
| Maintenance | High | Active? Recent releases? |
| Documentation | High | Good docs and examples? |
| Community | Medium | Stars, downloads, issues? |
| Bundle Size | Medium | Impact on app size? |
| TypeScript | Medium | Good type definitions? |
| Dependencies | Low | How many dependencies? |
| License | Varies | Compatible with project? |

### 4. Document Findings

## Research Report Template

```markdown
# Library Research: [Category]

## Requirements
- Primary need: [description]
- Must-have features: [list]
- Constraints: [size, license, etc.]

## Candidates Evaluated

### Summary Table
| Library | Stars | Weekly Downloads | Size | TypeScript | Last Updated |
|---------|-------|------------------|------|------------|--------------|
| lib-a | 25k | 1M | 5kb | ✅ | 2024-01 |
| lib-b | 18k | 500k | 12kb | ✅ | 2024-01 |
| lib-c | 10k | 200k | 3kb | ❌ | 2023-10 |

### Feature Comparison
| Feature | lib-a | lib-b | lib-c |
|---------|-------|-------|-------|
| Feature 1 | ✅ | ✅ | ❌ |
| Feature 2 | ✅ | ❌ | ✅ |
| Feature 3 | ✅ | ✅ | ✅ |
| TypeScript | ✅ Native | ✅ Native | ❌ @types |
| Tree-shaking | ✅ | ✅ | ❌ |

### Detailed Analysis

#### lib-a
**Description:** Brief description of what it does.

**Pros:**
- Pro 1
- Pro 2
- Pro 3

**Cons:**
- Con 1
- Con 2

**Installation:**
```bash
npm install lib-a
```

**Basic Usage:**
```typescript
import { something } from 'lib-a';
// Example code
```

**Documentation:** [Link](https://lib-a.dev)

---

#### lib-b
[Similar structure]

---

#### lib-c
[Similar structure]

## Recommendation

**Recommended: lib-a**

**Reasoning:**
1. Best feature coverage for our needs
2. Excellent TypeScript support
3. Active maintenance and community
4. Reasonable bundle size

**Trade-offs:**
- Slightly larger than lib-c
- Steeper learning curve than lib-b

**Alternative:** If bundle size is critical, consider lib-c but note the maintenance concerns.

## Implementation Notes

- Import specific functions for tree-shaking
- Configure with [specific settings]
- Watch for [known issues]

## References
- [Official docs](link)
- [Comparison article](link)
- [Migration guide](link)
```

## Common Categories

### Form Libraries
- react-hook-form
- Formik
- Final Form
- TanStack Form

### State Management
- Zustand
- Jotai
- Redux Toolkit
- Recoil

### Data Fetching
- TanStack Query (React Query)
- SWR
- RTK Query

### UI Components
- shadcn/ui
- Radix UI
- Headless UI
- Chakra UI
- MUI

### Animation
- Framer Motion
- React Spring
- Auto Animate

### Date/Time
- date-fns
- Day.js
- Luxon

### Validation
- Zod
- Yup
- Valibot

### HTTP Clients
- Native fetch
- Axios
- ky

### Testing
- Vitest
- Jest
- Playwright
- Cypress

### ORM/Database
- Prisma
- Drizzle
- Kysely
- TypeORM

## Using Context7

```typescript
// First resolve the library ID
const library = await resolveLibraryId({
  libraryName: "react-hook-form",
  query: "form validation with zod"
});

// Then query the documentation
const docs = await queryDocs({
  libraryId: library.id,
  query: "how to integrate with zod resolver"
});
```

## Research Checklist

- [ ] Requirements clearly defined
- [ ] Multiple options evaluated
- [ ] Quantitative data gathered (stars, downloads)
- [ ] Documentation quality assessed
- [ ] TypeScript support verified
- [ ] Bundle size checked
- [ ] License compatibility confirmed
- [ ] Example code tested
- [ ] Recommendation justified
- [ ] Trade-offs documented
