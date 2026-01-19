---
name: env-config-manager
description: |
  Manage environment variables and configuration across different environments.
  Handles secrets, validation, and environment-specific settings.
---

# Environment Config Manager Skill

## Purpose
Manage application configuration securely across different environments.

## Environment File Structure

```
project/
├── .env.example        # Template (committed)
├── .env                # Local development (gitignored)
├── .env.local          # Local overrides (gitignored)
├── .env.development    # Development defaults
├── .env.production     # Production defaults (non-sensitive)
└── .env.test           # Test environment
```

## .env.example Template

```bash
# .env.example
# Copy to .env and fill in values

# ===========================================
# Application
# ===========================================
NODE_ENV=development
PORT=3000
APP_URL=http://localhost:3000

# ===========================================
# Database
# ===========================================
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# ===========================================
# Authentication
# ===========================================
# Generate with: openssl rand -base64 32
JWT_SECRET=
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=
JWT_REFRESH_EXPIRES_IN=7d

# ===========================================
# External Services
# ===========================================
# Redis (optional)
REDIS_URL=redis://localhost:6379

# Email (Resend)
RESEND_API_KEY=

# File Storage (S3-compatible)
S3_ENDPOINT=
S3_BUCKET=
S3_ACCESS_KEY=
S3_SECRET_KEY=

# ===========================================
# Feature Flags
# ===========================================
ENABLE_ANALYTICS=false
ENABLE_RATE_LIMITING=true
```

## Environment Validation with Zod

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  // App
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().default(3000),
  APP_URL: z.string().url(),

  // Database
  DATABASE_URL: z.string().url(),

  // Auth
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
  JWT_EXPIRES_IN: z.string().default('15m'),
  JWT_REFRESH_SECRET: z.string().min(32),
  JWT_REFRESH_EXPIRES_IN: z.string().default('7d'),

  // Optional services
  REDIS_URL: z.string().url().optional(),
  RESEND_API_KEY: z.string().optional(),

  // Feature flags
  ENABLE_ANALYTICS: z.coerce.boolean().default(false),
  ENABLE_RATE_LIMITING: z.coerce.boolean().default(true),
});

// Parse and validate
const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('❌ Invalid environment variables:');
  console.error(parsed.error.flatten().fieldErrors);
  process.exit(1);
}

export const env = parsed.data;

// Type export for autocomplete
export type Env = z.infer<typeof envSchema>;
```

## Client-Side Environment

### Next.js Public Variables
```typescript
// lib/env.client.ts
import { z } from 'zod';

// Only NEXT_PUBLIC_ variables are available on client
const clientEnvSchema = z.object({
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url().optional(),
  NEXT_PUBLIC_ANALYTICS_ID: z.string().optional(),
});

export const clientEnv = clientEnvSchema.parse({
  NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  NEXT_PUBLIC_ANALYTICS_ID: process.env.NEXT_PUBLIC_ANALYTICS_ID,
});
```

## Configuration Object Pattern

```typescript
// config/index.ts
import { env } from '@/lib/env';

export const config = {
  app: {
    name: 'MyApp',
    url: env.APP_URL,
    port: env.PORT,
    isDev: env.NODE_ENV === 'development',
    isProd: env.NODE_ENV === 'production',
    isTest: env.NODE_ENV === 'test',
  },

  db: {
    url: env.DATABASE_URL,
    poolSize: env.NODE_ENV === 'production' ? 20 : 5,
  },

  auth: {
    jwtSecret: env.JWT_SECRET,
    jwtExpiresIn: env.JWT_EXPIRES_IN,
    refreshSecret: env.JWT_REFRESH_SECRET,
    refreshExpiresIn: env.JWT_REFRESH_EXPIRES_IN,
  },

  redis: env.REDIS_URL
    ? {
        url: env.REDIS_URL,
        enabled: true,
      }
    : {
        enabled: false,
      },

  email: env.RESEND_API_KEY
    ? {
        apiKey: env.RESEND_API_KEY,
        enabled: true,
        from: 'noreply@myapp.com',
      }
    : {
        enabled: false,
      },

  features: {
    analytics: env.ENABLE_ANALYTICS,
    rateLimiting: env.ENABLE_RATE_LIMITING,
  },
} as const;
```

## Docker Environment

### docker-compose.yml
```yaml
services:
  app:
    build: .
    env_file:
      - .env.production
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD}@db:5432/app
    secrets:
      - jwt_secret

secrets:
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

### Environment Variables in Dockerfile
```dockerfile
# Build-time arguments
ARG NODE_ENV=production

# Runtime environment
ENV NODE_ENV=${NODE_ENV}

# Don't set secrets in Dockerfile!
# Pass them at runtime via -e or env_file
```

## CI/CD Environment

### GitHub Actions Secrets
```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    environment: production
    steps:
      - name: Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
        run: |
          echo "Deploying with secrets..."
```

### Setting Secrets
```bash
# GitHub CLI
gh secret set JWT_SECRET --body "your-secret-value"
gh secret set DATABASE_URL --body "postgresql://..."

# Or use environment-specific secrets
gh secret set JWT_SECRET --env production --body "prod-secret"
```

## Vercel Environment

```bash
# Vercel CLI
vercel env add JWT_SECRET production
vercel env add DATABASE_URL production

# Or use vercel.json for non-sensitive
{
  "env": {
    "APP_URL": "https://myapp.vercel.app"
  }
}
```

## Security Best Practices

### Never Commit Secrets
```gitignore
# .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
secrets/
```

### Rotate Secrets
```typescript
// Support multiple secrets during rotation
const JWT_SECRETS = [
  env.JWT_SECRET,
  env.JWT_SECRET_PREVIOUS, // Old secret for validation during rotation
].filter(Boolean);

function verifyToken(token: string) {
  for (const secret of JWT_SECRETS) {
    try {
      return jwt.verify(token, secret);
    } catch {
      continue;
    }
  }
  throw new Error('Invalid token');
}
```

### Secret Generation
```bash
# Generate secure random strings
openssl rand -base64 32

# Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

## Quality Checklist

- [ ] .env.example committed with all keys
- [ ] Environment validation at startup
- [ ] No secrets in code or Dockerfile
- [ ] Different configs per environment
- [ ] Client variables properly prefixed
- [ ] Secrets in CI/CD configured
- [ ] Production secrets rotated
- [ ] Documentation for each variable
