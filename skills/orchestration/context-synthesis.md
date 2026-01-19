---
name: context-synthesis
description: |
  Synthesize outputs from multiple agents into coherent context for the next step.
  Maintains project context across agent boundaries.
---

# Context Synthesis Skill

## Purpose
Combine outputs from multiple agents into unified, actionable context for continued work.

## Synthesis Process

### 1. Collect Agent Outputs
Gather deliverables from each agent:
- Documents produced
- Code written
- Decisions made
- Issues identified

### 2. Identify Connections
Map relationships between outputs:
- How does the PRD inform the architecture?
- How does the schema affect the API design?
- What frontend needs does the backend satisfy?

### 3. Resolve Conflicts
If agent outputs conflict:
- Identify the source of disagreement
- Determine which approach better serves the goal
- Document the decision and reasoning

### 4. Create Unified View

```markdown
## Project Context Summary

### Current State
[Brief description of where the project stands]

### Completed Work
| Agent | Deliverable | Status |
|-------|-------------|--------|
| product-manager | PRD v1.0 | ✅ Complete |
| architect | System design | ✅ Complete |
| database-specialist | Schema migration | ✅ Complete |

### Key Decisions
1. **Tech Stack**: Next.js + PostgreSQL + Prisma
   - Rationale: Team familiarity, rapid development
2. **Auth**: JWT with refresh tokens
   - Rationale: Stateless, mobile-friendly

### Active Work
- frontend-engineer: Building dashboard components
- backend-engineer: Implementing user API

### Blockers & Dependencies
- [ ] Waiting for design assets for dashboard
- [ ] API schema needed before frontend integration

### Next Steps
1. Complete user API endpoints
2. Build dashboard UI
3. Integration testing
```

## Context Handoff Template

When passing context to the next agent:

```markdown
## Context for [Agent Name]

### Your Task
[Clear description of what to do]

### Background
[Relevant context from previous work]

### Inputs Available
- [Document/artifact 1]
- [Document/artifact 2]

### Constraints
- [Technical constraint]
- [Timeline constraint]

### Dependencies
- Depends on: [previous work]
- Blocks: [future work]

### Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2
```

## Maintaining Continuity

### Session State
Track across the session:
- Project name and description
- Tech stack decisions
- File structure
- Key interfaces/contracts
- Outstanding questions

### Decision Log
Record important decisions:
```markdown
| Date | Decision | Rationale | Made By |
|------|----------|-----------|---------|
| ... | Use Prisma | Team familiarity | architect |
| ... | JWT auth | Mobile support needed | backend-engineer |
```

## Quality Checks

Before passing context:
- [ ] All relevant information included
- [ ] No contradictions between outputs
- [ ] Clear next steps defined
- [ ] Blockers identified and communicated
