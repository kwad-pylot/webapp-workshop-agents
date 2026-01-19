# Project Instructions

## Plugin Guidelines

1. **Never modify agent files** (`agents/*.md`) without confirming with the user first
2. **Never modify skill files** (`skills/**/*.md`) without confirming with the user first
3. **Always use the orchestrator agent first** for multi-step tasks - let it delegate to specialists

## Agent Hierarchy

- **Orchestrator** (opus) - Always start here for complex tasks
- **Planning agents** - Requirements, architecture, task breakdown
- **Implementation agents** - Frontend, backend, database, fullstack
- **Quality agents** - Testing, code review, security
- **Operations agents** - DevOps, documentation, research

---

## Integration with GSD (Get Shit Done)

This plugin is designed to work alongside [GSD](https://github.com/glittercowboy/get-shit-done).

### Framework Roles

| Framework | Role | When to Use |
|-----------|------|-------------|
| **GSD** | Project Manager | Process, planning, tracking, context persistence |
| **WebApp Workshop** | Development Team | Implementation, code generation, domain expertise |

### How They Work Together

```
GSD = Manages the PROCESS (what to build, when, tracking)
WebApp Workshop = Does the WORK (actual code, implementation)
```

### Recommended Workflow

**For multi-phase projects:**
1. Use `/gsd:new-project` to define the project
2. Use `/gsd:create-roadmap` to plan phases
3. Use `/gsd:plan-phase` to create detailed plans
4. During `/gsd:execute-plan`, use WebApp Workshop agents for implementation:
   - Orchestrator coordinates the work
   - Specialist agents write the actual code
5. Use `/gsd:verify-work` for testing
6. Use `/gsd:progress` to track completion

**For quick tasks (no planning needed):**
- Use WebApp Workshop agents directly
- Orchestrator coordinates specialists as needed

### Decision Guide

| Situation | Action |
|-----------|--------|
| Starting a new project | `/gsd:new-project` first |
| Resuming after a break | `/gsd:resume-work` |
| Need to track multi-day work | Use GSD for persistence |
| Quick feature or fix | WebApp Workshop agents directly |
| Debugging an issue | `/gsd:debug` |
| Code review | WebApp Workshop `code-reviewer` |
| Writing implementation code | WebApp Workshop specialist agents |
| Planning phases/milestones | GSD commands |

### Key Commands Reference

**GSD (Process Management):**
- `/gsd:new-project` - Initialize project
- `/gsd:create-roadmap` - Plan phases
- `/gsd:plan-phase N` - Create detailed plan
- `/gsd:execute-plan <path>` - Execute a plan
- `/gsd:progress` - Check status
- `/gsd:resume-work` - Resume after break
- `/gsd:debug` - Systematic debugging

**WebApp Workshop (Implementation):**
- `orchestrator` - Coordinate complex builds
- `architect` - Tech stack decisions
- `frontend-engineer` - UI/React components
- `backend-engineer` - APIs/auth
- `database-specialist` - Schema/queries
- `qa-engineer` - Tests
- `code-reviewer` - Code quality
- `security-specialist` - Security audit
