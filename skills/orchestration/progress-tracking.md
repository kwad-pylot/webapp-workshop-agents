---
name: progress-tracking
description: |
  Track project progress across all agents and phases.
  Maintain visibility into overall project status.
---

# Progress Tracking Skill

## Purpose
Monitor and report on project progress across all workstreams.

## Progress Tracking Structure

### Project Status Board

```markdown
## Project: [Name]

### Phase Progress
| Phase | Status | Progress | Blockers |
|-------|--------|----------|----------|
| 1. Planning | ✅ Complete | 100% | - |
| 2. Setup | ✅ Complete | 100% | - |
| 3. Database | 🔄 In Progress | 75% | - |
| 4. Backend | 🔄 In Progress | 50% | Waiting on schema |
| 5. Frontend | ⏳ Pending | 0% | - |
| 6. Testing | ⏳ Pending | 0% | - |
| 7. Deployment | ⏳ Pending | 0% | - |

### Active Tasks
| Task | Agent | Started | Status |
|------|-------|---------|--------|
| User API endpoints | backend-engineer | 10:30 | In Progress |
| Posts schema | database-specialist | 10:45 | In Progress |

### Completed Today
- ✅ Project setup (devops-engineer)
- ✅ PRD approved (product-manager)
- ✅ Architecture design (architect)
- ✅ Users table migration (database-specialist)

### Upcoming
1. [ ] Complete posts schema
2. [ ] Implement posts API
3. [ ] Build post list component
```

## TodoWrite Integration

Use TodoWrite to maintain real-time progress:

```typescript
// Update when starting a task
TodoWrite([
  { content: "Implement user API", status: "in_progress", activeForm: "Implementing user API" },
  { content: "Create posts schema", status: "pending", activeForm: "Creating posts schema" },
]);

// Update when completing
TodoWrite([
  { content: "Implement user API", status: "completed", activeForm: "Implementing user API" },
  { content: "Create posts schema", status: "in_progress", activeForm: "Creating posts schema" },
]);
```

## Progress Metrics

### Velocity Tracking
- Tasks completed per phase
- Average task completion time
- Blockers encountered

### Health Indicators

| Indicator | Good | Warning | Critical |
|-----------|------|---------|----------|
| Blockers | 0 | 1-2 | 3+ |
| Overdue tasks | 0 | 1 | 2+ |
| Phase progress | On track | 1 phase behind | 2+ behind |

## Status Update Format

### Brief Update (for user)
```
📊 Project Status: [Name]

✅ Completed: 5 tasks
🔄 In Progress: 2 tasks
⏳ Remaining: 8 tasks

Current: Building user API endpoints
Next up: Frontend dashboard

Any blockers? No
```

### Detailed Update (for handoff)
```markdown
## Status Report: [Timestamp]

### Summary
Project is 40% complete. On track for delivery.

### Phase Breakdown
[Detailed phase status]

### Risk Items
- [ ] External API integration untested
- [ ] Design assets delayed

### Decisions Needed
- [ ] Confirm deployment platform
- [ ] Review API schema before frontend work
```

## Progress Communication

### When to Update User
- Phase completion
- Blocker encountered
- Decision needed
- Significant milestone reached

### Update Cadence
- Major milestone: Immediate notification
- Phase completion: Summary report
- End of session: Full status report

## Recovery Tracking

When issues occur:
1. Log the issue
2. Identify impact on timeline
3. Propose recovery plan
4. Update estimates
5. Communicate to user
