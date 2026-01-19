# WebApp Workshop Agents

A Claude Code plugin providing a multi-agent web development team with **14 specialized AI agents** and **35 core skills** for building any type of web application.

## Overview

This plugin implements an orchestrator pattern where Claude acts as a development team lead, delegating tasks to specialist agents. Each agent has domain expertise and access to relevant skills for their area.

## Installation

1. Clone or copy this plugin to your Claude Code plugins directory
2. The plugin will be automatically discovered via the `.claude-plugin/plugin.json` manifest

## Agents (14 Total)

### Orchestration Layer

| Agent | Model | Purpose |
|-------|-------|---------|
| **orchestrator** | opus | Lead development orchestrator - coordinates all agents, delegates tasks, synthesizes context |

### Planning & Design Layer

| Agent | Model | Purpose |
|-------|-------|---------|
| **product-manager** | sonnet | Product requirements, PRDs, user stories, acceptance criteria |
| **architect** | opus | Tech stack selection, system design, architecture decisions |
| **project-manager** | sonnet | Task decomposition, dependency mapping, sprint planning |

### Implementation Layer

| Agent | Model | Purpose |
|-------|-------|---------|
| **frontend-engineer** | sonnet | React components, styling, responsive design, state management |
| **backend-engineer** | sonnet | REST/GraphQL APIs, authentication, middleware, error handling |
| **database-specialist** | sonnet | Schema design, migrations, query optimization, seeding |
| **fullstack-engineer** | sonnet | Feature integration, API consumption, real-time features |

### Quality Layer

| Agent | Model | Purpose |
|-------|-------|---------|
| **qa-engineer** | sonnet | Unit tests, integration tests, test strategies |
| **code-reviewer** | sonnet | Code quality analysis, best practices, refactoring suggestions |
| **security-specialist** | sonnet | Vulnerability scanning, auth auditing, security hardening |

### Operations Layer

| Agent | Model | Purpose |
|-------|-------|---------|
| **devops-engineer** | sonnet | Docker, CI/CD pipelines, environment configuration |
| **documentation-writer** | haiku | API documentation, README generation, technical writing |
| **research-explorer** | sonnet | Library research, technology evaluation, documentation lookup |

## Skills (35 Total)

### Orchestration Skills (3)
- `task-delegation` - Delegate tasks to appropriate specialist agents
- `context-synthesis` - Synthesize information from multiple agents
- `progress-tracking` - Track project progress and dependencies

### Planning Skills (6)
- `prd-generator` - Generate Product Requirements Documents
- `user-story-writer` - Write user stories with acceptance criteria
- `tech-stack-selector` - Evaluate and select technology stacks
- `system-diagrammer` - Create architecture and system diagrams
- `task-decomposer` - Break down features into actionable tasks
- `dependency-mapper` - Map task dependencies and critical paths

### Frontend Skills (4)
- `component-builder` - Build React/Next.js components
- `styling-expert` - Implement Tailwind CSS styling
- `responsive-design` - Create responsive layouts
- `state-manager` - Implement state management (Zustand, Context, etc.)

### Backend Skills (4)
- `api-builder` - Build REST and GraphQL APIs
- `auth-implementer` - Implement JWT/session authentication
- `middleware-creator` - Create Express/Next.js middleware
- `error-handler` - Implement error handling patterns

### Database Skills (4)
- `schema-designer` - Design database schemas with Prisma
- `migration-creator` - Create database migrations
- `query-optimizer` - Optimize database queries
- `seeder-creator` - Create database seed files

### Fullstack Skills (3)
- `feature-integrator` - Integrate frontend and backend features
- `api-consumer` - Connect frontend to APIs with React Query
- `realtime-implementer` - Implement WebSocket/real-time features

### Quality Skills (5)
- `unit-test-writer` - Write unit tests with Vitest/Jest
- `integration-test-writer` - Write integration and E2E tests
- `code-quality-checker` - Analyze code quality and suggest improvements
- `vulnerability-scanner` - Scan for security vulnerabilities
- `auth-auditor` - Audit authentication implementations

### DevOps Skills (3)
- `dockerfile-creator` - Create optimized Dockerfiles
- `ci-cd-builder` - Build GitHub Actions CI/CD pipelines
- `env-config-manager` - Manage environment configuration

### Documentation Skills (3)
- `api-documenter` - Create OpenAPI/Swagger documentation
- `readme-generator` - Generate comprehensive README files
- `library-researcher` - Research and evaluate libraries

## Usage Examples

### Build a Complete Feature
```
User: "Build me a user authentication system"

Claude (orchestrator) will:
1. Delegate to architect for auth strategy
2. Delegate to database-specialist for user schema
3. Delegate to backend-engineer for auth endpoints
4. Delegate to frontend-engineer for login/signup forms
5. Delegate to qa-engineer for auth tests
6. Delegate to security-specialist for security audit
```

### Start a New Project
```
User: "Help me plan a task management app"

Claude (orchestrator) will:
1. Delegate to product-manager for PRD
2. Delegate to architect for tech stack
3. Delegate to project-manager for task breakdown
4. Coordinate implementation with specialist agents
```

### Code Review
```
User: "Review my API implementation"

Claude will trigger code-reviewer agent to:
1. Analyze code quality
2. Check for best practices
3. Identify potential issues
4. Suggest improvements
```

## Directory Structure

```
webapp-workshop-agents/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── agents/
│   ├── orchestrator.md      # Lead coordinator
│   ├── product-manager.md   # Requirements
│   ├── architect.md         # System design
│   ├── project-manager.md   # Task management
│   ├── frontend-engineer.md # UI development
│   ├── backend-engineer.md  # API development
│   ├── database-specialist.md # Data layer
│   ├── fullstack-engineer.md  # Integration
│   ├── qa-engineer.md       # Testing
│   ├── code-reviewer.md     # Code quality
│   ├── security-specialist.md # Security
│   ├── devops-engineer.md   # Operations
│   ├── documentation-writer.md # Docs
│   └── research-explorer.md # Research
├── skills/
│   ├── orchestration/       # 3 skills
│   ├── planning/            # 6 skills
│   ├── frontend/            # 4 skills
│   ├── backend/             # 4 skills
│   ├── database/            # 4 skills
│   ├── fullstack/           # 3 skills
│   ├── quality/             # 5 skills
│   ├── devops/              # 3 skills
│   └── documentation/       # 3 skills
└── README.md
```

## Technology Stack Coverage

This plugin provides expertise in:

- **Frontend**: React, Next.js, TypeScript, Tailwind CSS, shadcn/ui
- **Backend**: Node.js, Express, Next.js API Routes, tRPC
- **Database**: PostgreSQL, Prisma ORM, Supabase
- **Auth**: JWT, NextAuth.js, session-based auth
- **Testing**: Vitest, Jest, Playwright, Testing Library
- **DevOps**: Docker, GitHub Actions, Vercel, Railway
- **Documentation**: OpenAPI/Swagger, Markdown, JSDoc

## MCP Server Integration

Some agents leverage MCP servers for enhanced capabilities:

- **database-specialist**: Uses Supabase MCP for direct database operations
- **research-explorer**: Uses Context7 MCP for library documentation lookup

## Agent Trigger Phrases

Each agent responds to specific trigger phrases:

| Agent | Trigger Phrases |
|-------|-----------------|
| orchestrator | "build me", "create a", "help me build", "develop" |
| product-manager | "requirements", "PRD", "user stories", "features" |
| architect | "architecture", "tech stack", "system design" |
| project-manager | "break down", "tasks", "sprint", "plan" |
| frontend-engineer | "component", "UI", "styling", "frontend" |
| backend-engineer | "API", "endpoint", "authentication", "backend" |
| database-specialist | "schema", "database", "migration", "query" |
| fullstack-engineer | "integrate", "connect", "full feature" |
| qa-engineer | "test", "testing", "coverage" |
| code-reviewer | "review", "code quality", "refactor" |
| security-specialist | "security", "vulnerability", "audit" |
| devops-engineer | "deploy", "Docker", "CI/CD", "pipeline" |
| documentation-writer | "document", "README", "API docs" |
| research-explorer | "research", "compare libraries", "find package" |

## Integration with GSD (Get Shit Done)

This plugin works best when combined with [GSD](https://github.com/glittercowboy/get-shit-done) - a context engineering and project management framework for Claude Code.

### Why Use Both?

| Framework | Role | Strength |
|-----------|------|----------|
| **GSD** | Project Manager | Process, tracking, context engineering, persistence |
| **WebApp Workshop** | Development Team | Domain expertise, code generation, implementation |

**GSD** ensures consistency and prevents context rot. **WebApp Workshop** provides specialized expertise for web development execution.

### Combined Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: GSD Handles the Process                           │
├─────────────────────────────────────────────────────────────┤
│  /gsd:new-project         → Define vision & requirements    │
│  /gsd:create-roadmap      → Plan phases & milestones        │
│  /gsd:discuss-phase 1     → Capture preferences             │
│  /gsd:plan-phase 1        → Create detailed PLAN.md         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: WebApp Workshop Does the Work                     │
├─────────────────────────────────────────────────────────────┤
│  /gsd:execute-plan        → Triggers execution              │
│    └── orchestrator       → Coordinates specialists         │
│        ├── architect      → Tech decisions                  │
│        ├── backend-eng    → API code                        │
│        ├── frontend-eng   → UI components                   │
│        ├── database-spec  → Schema & migrations             │
│        └── qa-engineer    → Tests                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: GSD Tracks Completion                             │
├─────────────────────────────────────────────────────────────┤
│  /gsd:verify-work         → UAT testing                     │
│  /gsd:progress            → Track completion                │
│  /gsd:complete-milestone  → Archive & tag release           │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Which

| Scenario | Use |
|----------|-----|
| Starting any project | GSD `/gsd:new-project` |
| Tracking multi-day work | GSD (STATE.md persistence) |
| Writing web app code | WebApp Workshop agents |
| Debugging issues | GSD `/gsd:debug` |
| Code review | WebApp Workshop `code-reviewer` |
| Resuming after break | GSD `/gsd:resume-work` |
| Quick feature (no planning) | WebApp Workshop agents directly |
| Complex multi-phase project | GSD + WebApp Workshop together |

### Installing GSD

```bash
npx get-shit-done-cc --global
```

Then verify with `/gsd:help` in Claude Code.

## Best Practices

1. **Start with Planning**: Use product-manager and architect before implementation
2. **Delegate Appropriately**: Let the orchestrator coordinate complex tasks
3. **Test Early**: Engage qa-engineer throughout development
4. **Security First**: Include security-specialist for auth features
5. **Document As You Go**: Use documentation-writer for API docs
6. **Use GSD for Projects**: For multi-phase work, use GSD for process management

## Contributing

To extend this plugin:

1. Add new agents to the `agents/` directory
2. Add new skills to the appropriate `skills/` subdirectory
3. Update the agent's skill references if needed
4. Test the new agent/skill triggers

## License

MIT License - Feel free to use and modify for your projects.
