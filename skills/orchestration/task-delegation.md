---
name: task-delegation
description: |
  Delegate tasks to the appropriate specialist agent based on task requirements.
  Used by the orchestrator to route work to the right team member.
---

# Task Delegation Skill

## Purpose
Analyze tasks and delegate them to the most appropriate specialist agent.

## Delegation Matrix

| Task Type | Primary Agent | Backup Agent |
|-----------|---------------|--------------|
| Requirements gathering | product-manager | - |
| PRD creation | product-manager | - |
| Architecture design | architect | - |
| Tech stack selection | architect | research-explorer |
| Task breakdown | project-manager | - |
| UI components | frontend-engineer | fullstack-engineer |
| Styling/CSS | frontend-engineer | - |
| API endpoints | backend-engineer | fullstack-engineer |
| Authentication | backend-engineer | security-specialist |
| Database schema | database-specialist | - |
| Migrations | database-specialist | - |
| Frontend-backend integration | fullstack-engineer | - |
| Unit tests | qa-engineer | - |
| Integration tests | qa-engineer | - |
| Code review | code-reviewer | - |
| Security audit | security-specialist | - |
| Docker/containers | devops-engineer | - |
| CI/CD pipeline | devops-engineer | - |
| Documentation | documentation-writer | - |
| Library research | research-explorer | - |

## Delegation Format

When delegating a task, use the Task tool with this structure:

```
Agent: [agent-name]
Task: [specific task description]
Context: [relevant background information]
Deliverable: [expected output]
Priority: [high/medium/low]
Dependencies: [tasks that must complete first]
```

## Delegation Rules

1. **Single Responsibility**: Each task should map to one primary agent
2. **Clear Scope**: Define exactly what the agent should deliver
3. **Context Sharing**: Provide all relevant context from previous work
4. **Dependency Awareness**: Note which tasks must complete first
5. **Quality Gates**: Define acceptance criteria when appropriate

## Parallel vs Sequential

**Parallel Tasks** (can run simultaneously):
- Frontend components + Backend APIs (with agreed interface)
- Unit tests for different modules
- Documentation + Code review

**Sequential Tasks** (must run in order):
- Database schema → API endpoints → Frontend integration
- Architecture → Implementation → Testing
- Code → Review → Fixes → Re-review

## Handling Blockers

If an agent reports a blocker:
1. Assess if another agent can help
2. Check if the task scope needs adjustment
3. Escalate to user if external input needed
4. Update the project plan accordingly
