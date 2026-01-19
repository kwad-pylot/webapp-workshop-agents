---
name: system-diagrammer
description: |
  Create system architecture diagrams using Mermaid syntax.
  Visualizes system components, data flow, and interactions.
---

# System Diagrammer Skill

## Purpose
Create clear, maintainable architecture diagrams that document system design.

## Diagram Types

### 1. High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        Web[Web App<br/>Next.js]
        Mobile[Mobile App<br/>React Native]
    end

    subgraph "API Layer"
        Gateway[API Gateway<br/>Next.js API Routes]
        Auth[Auth Service<br/>NextAuth]
    end

    subgraph "Business Layer"
        Users[User Service]
        Products[Product Service]
        Orders[Order Service]
    end

    subgraph "Data Layer"
        DB[(PostgreSQL)]
        Cache[(Redis)]
        Storage[S3 Storage]
    end

    Web --> Gateway
    Mobile --> Gateway
    Gateway --> Auth
    Gateway --> Users
    Gateway --> Products
    Gateway --> Orders
    Users --> DB
    Products --> DB
    Products --> Cache
    Orders --> DB
    Users --> Storage
```

### 2. Request Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant C as Client
    participant A as API
    participant D as Database

    U->>C: Click "Login"
    C->>A: POST /api/auth/login
    A->>D: Query user by email
    D-->>A: User record
    A->>A: Verify password
    A->>A: Generate JWT
    A-->>C: { token, user }
    C->>C: Store token
    C-->>U: Redirect to dashboard
```

### 3. Database Schema Diagram

```mermaid
erDiagram
    USER ||--o{ POST : writes
    USER ||--o{ COMMENT : writes
    POST ||--o{ COMMENT : has
    POST }o--o{ TAG : has

    USER {
        uuid id PK
        string email UK
        string name
        string password_hash
        timestamp created_at
    }

    POST {
        uuid id PK
        uuid author_id FK
        string title
        text content
        boolean published
        timestamp created_at
    }

    COMMENT {
        uuid id PK
        uuid post_id FK
        uuid author_id FK
        text content
        timestamp created_at
    }

    TAG {
        uuid id PK
        string name UK
    }
```

### 4. Component Diagram

```mermaid
graph LR
    subgraph "Pages"
        Home[Home Page]
        Dashboard[Dashboard]
        Profile[Profile]
    end

    subgraph "Features"
        Auth[Auth Module]
        Tasks[Tasks Module]
        Settings[Settings Module]
    end

    subgraph "Shared"
        UI[UI Components]
        Hooks[Custom Hooks]
        Utils[Utilities]
    end

    Home --> UI
    Dashboard --> Tasks
    Dashboard --> UI
    Dashboard --> Hooks
    Profile --> Settings
    Profile --> UI
    Auth --> Hooks
    Tasks --> Hooks
    Tasks --> Utils
```

### 5. Deployment Diagram

```mermaid
graph TB
    subgraph "Vercel"
        Frontend[Next.js App]
        API[API Routes]
        Edge[Edge Functions]
    end

    subgraph "Supabase"
        DB[(PostgreSQL)]
        AuthSvc[Auth Service]
        Storage[File Storage]
        Realtime[Realtime]
    end

    subgraph "External"
        Stripe[Stripe API]
        SendGrid[SendGrid]
        S3[AWS S3]
    end

    Frontend --> API
    Frontend --> Edge
    API --> DB
    API --> AuthSvc
    API --> Storage
    API --> Stripe
    API --> SendGrid
    Frontend --> Realtime
    Storage --> S3
```

### 6. State Flow Diagram

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Loading: Fetch Data
    Loading --> Success: Data Received
    Loading --> Error: Request Failed
    Success --> Idle: Reset
    Error --> Loading: Retry
    Error --> Idle: Dismiss
```

### 7. User Flow Diagram

```mermaid
graph TD
    Start([User visits site]) --> Auth{Authenticated?}
    Auth -->|No| Login[Login Page]
    Auth -->|Yes| Dashboard[Dashboard]
    Login --> Register[Register]
    Login --> Forgot[Forgot Password]
    Register --> Verify[Email Verification]
    Verify --> Dashboard
    Forgot --> Reset[Reset Password]
    Reset --> Login
    Dashboard --> Profile[View Profile]
    Dashboard --> Tasks[Manage Tasks]
    Dashboard --> Settings[Settings]
    Profile --> Edit[Edit Profile]
    Edit --> Dashboard
```

## Diagram Guidelines

### Clarity
- Use descriptive labels
- Group related components
- Show direction of data flow
- Include technology names where helpful

### Consistency
- Use same shapes for same types
- Consistent color coding
- Standard arrow directions (top-to-bottom, left-to-right)

### Appropriate Detail
- High-level for overview
- Detailed for specific flows
- Don't include everything in one diagram

## Mermaid Syntax Reference

### Shapes
```
[Rectangle] for services
([Stadium]) for start/end
{Diamond} for decisions
[(Database)] for data stores
{{Hexagon}} for preparation
```

### Arrows
```
--> Solid arrow
-.-> Dashed arrow
==> Thick arrow
--text--> Arrow with label
```

### Subgraphs
```mermaid
subgraph "Group Name"
    A --> B
end
```

## Output Format

When creating diagrams, provide:
1. The Mermaid code block
2. Brief explanation of the diagram
3. Key architectural decisions shown
