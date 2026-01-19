# Project Instructions

## Plugin Guidelines

1. **Never modify agent files** (`agents/*.md`) without confirming with the user first
2. **Never modify skill files** (`skills/**/*.md`) without confirming with the user first
3. **Always use the orchestrator agent first** for multi-step tasks - let it delegate to specialists

## Usage Pattern

When building web applications or features:
1. Start with the **orchestrator** agent for coordination
2. Let orchestrator delegate to specialist agents as needed
3. Follow the orchestrator's task breakdown

## Agent Hierarchy

- **Orchestrator** (opus) - Always start here for complex tasks
- **Planning agents** - Requirements, architecture, task breakdown
- **Implementation agents** - Frontend, backend, database, fullstack
- **Quality agents** - Testing, code review, security
- **Operations agents** - DevOps, documentation, research
