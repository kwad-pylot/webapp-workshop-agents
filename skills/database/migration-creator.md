---
name: migration-creator
description: |
  Create safe, reversible database migrations for schema changes.
  Handles both Prisma migrations and raw SQL migrations.
---

# Migration Creator Skill

## Purpose
Create safe database migrations that can be applied and rolled back.

## Prisma Migrations

### Creating Migrations
```bash
# Create migration from schema changes
npx prisma migrate dev --name add_user_profile

# Create migration without applying
npx prisma migrate dev --create-only --name add_posts_table

# Apply migrations in production
npx prisma migrate deploy

# Reset database (dev only!)
npx prisma migrate reset
```

### Migration Files
```
prisma/
├── migrations/
│   ├── 20240101000000_init/
│   │   └── migration.sql
│   ├── 20240102000000_add_posts/
│   │   └── migration.sql
│   └── migration_lock.toml
└── schema.prisma
```

### Generated Migration Example
```sql
-- prisma/migrations/20240101000000_init/migration.sql

-- CreateEnum
CREATE TYPE "Role" AS ENUM ('USER', 'ADMIN');

-- CreateTable
CREATE TABLE "users" (
    "id" TEXT NOT NULL,
    "email" TEXT NOT NULL,
    "password_hash" TEXT NOT NULL,
    "name" TEXT,
    "role" "Role" NOT NULL DEFAULT 'USER',
    "created_at" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "users_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "users_email_key" ON "users"("email");
```

## Custom Migration for Data Changes

```sql
-- prisma/migrations/20240201000000_backfill_slugs/migration.sql

-- Backfill slugs for existing posts
UPDATE posts
SET slug = LOWER(REPLACE(REPLACE(title, ' ', '-'), '''', ''))
WHERE slug IS NULL;

-- Now make slug required
ALTER TABLE posts ALTER COLUMN slug SET NOT NULL;
```

## Raw SQL Migrations (without Prisma)

### Directory Structure
```
migrations/
├── 001_create_users.up.sql
├── 001_create_users.down.sql
├── 002_create_posts.up.sql
├── 002_create_posts.down.sql
└── ...
```

### Up Migration
```sql
-- migrations/001_create_users.up.sql
BEGIN;

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

COMMIT;
```

### Down Migration
```sql
-- migrations/001_create_users.down.sql
BEGIN;

DROP TABLE IF EXISTS users;

COMMIT;
```

## Safe Migration Patterns

### Adding a Column
```sql
-- Safe: Add nullable column
ALTER TABLE users ADD COLUMN bio TEXT;

-- Safe: Add column with default
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;

-- UNSAFE: Adding NOT NULL without default on existing table
-- ALTER TABLE users ADD COLUMN required_field TEXT NOT NULL;

-- Safe way to add NOT NULL:
-- 1. Add nullable
ALTER TABLE users ADD COLUMN status VARCHAR(50);
-- 2. Backfill
UPDATE users SET status = 'active' WHERE status IS NULL;
-- 3. Add constraint
ALTER TABLE users ALTER COLUMN status SET NOT NULL;
-- 4. Add default for new rows
ALTER TABLE users ALTER COLUMN status SET DEFAULT 'active';
```

### Removing a Column
```sql
-- Safe: Remove column (ensure app doesn't use it first!)
ALTER TABLE users DROP COLUMN IF EXISTS legacy_field;
```

### Renaming a Column
```sql
-- Option 1: Direct rename (causes downtime)
ALTER TABLE users RENAME COLUMN old_name TO new_name;

-- Option 2: Zero-downtime approach
-- 1. Add new column
ALTER TABLE users ADD COLUMN new_name VARCHAR(255);
-- 2. Backfill
UPDATE users SET new_name = old_name;
-- 3. Update app to write to both
-- 4. Update app to read from new
-- 5. Update app to only write to new
-- 6. Drop old column
ALTER TABLE users DROP COLUMN old_name;
```

### Adding an Index
```sql
-- Blocking (locks table)
CREATE INDEX idx_posts_title ON posts(title);

-- Non-blocking (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_posts_title ON posts(title);
```

### Adding a Foreign Key
```sql
-- Add column
ALTER TABLE posts ADD COLUMN category_id UUID;

-- Add foreign key constraint
ALTER TABLE posts
ADD CONSTRAINT fk_posts_category
FOREIGN KEY (category_id) REFERENCES categories(id)
ON DELETE SET NULL;

-- Add index for performance
CREATE INDEX idx_posts_category ON posts(category_id);
```

## Data Migrations

```sql
-- migrations/005_migrate_user_status.up.sql
BEGIN;

-- Create new enum
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'suspended');

-- Add new column
ALTER TABLE users ADD COLUMN status user_status;

-- Migrate data
UPDATE users SET status =
  CASE
    WHEN is_active = true THEN 'active'::user_status
    WHEN is_banned = true THEN 'suspended'::user_status
    ELSE 'inactive'::user_status
  END;

-- Set default and NOT NULL
ALTER TABLE users ALTER COLUMN status SET DEFAULT 'active';
ALTER TABLE users ALTER COLUMN status SET NOT NULL;

-- Remove old columns (after app deployment)
-- ALTER TABLE users DROP COLUMN is_active;
-- ALTER TABLE users DROP COLUMN is_banned;

COMMIT;
```

## Migration Testing

```bash
# Test migration locally
# 1. Backup current state
pg_dump $DATABASE_URL > backup.sql

# 2. Apply migration
npx prisma migrate dev

# 3. Test application

# 4. If issues, restore
psql $DATABASE_URL < backup.sql
```

## Supabase Migrations

```typescript
// Using Supabase MCP
await applyMigration({
  project_id: 'your-project-id',
  name: 'add_user_profiles',
  query: `
    CREATE TABLE profiles (
      id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
      bio TEXT,
      website VARCHAR(255),
      avatar_url TEXT,
      created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
    );

    -- Enable RLS
    ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

    -- Policy: Users can read all profiles
    CREATE POLICY "Profiles are viewable by everyone"
      ON profiles FOR SELECT
      USING (true);

    -- Policy: Users can update own profile
    CREATE POLICY "Users can update own profile"
      ON profiles FOR UPDATE
      USING (auth.uid() = id);
  `
});
```

## Quality Checklist

- [ ] Migration is reversible (has down script)
- [ ] Uses transactions for atomicity
- [ ] Non-blocking index creation when possible
- [ ] Data backfill before adding NOT NULL
- [ ] Tested on copy of production data
- [ ] Foreign keys have indexes
- [ ] Large table changes are batched
- [ ] Migration name is descriptive
