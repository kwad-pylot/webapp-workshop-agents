---
name: schema-designer
description: |
  Design normalized database schemas with proper relationships, constraints, and indexes.
  Creates efficient, maintainable data models.
---

# Schema Designer Skill

## Purpose
Design efficient, normalized database schemas that support application requirements.

## Design Process

1. **Identify Entities**: What things do we need to store?
2. **Define Attributes**: What data does each entity have?
3. **Establish Relationships**: How do entities relate?
4. **Add Constraints**: What rules must the data follow?
5. **Create Indexes**: What queries need to be fast?

## Prisma Schema Design

### Basic Schema Structure
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Enums
enum Role {
  USER
  ADMIN
  MODERATOR
}

enum Status {
  DRAFT
  PUBLISHED
  ARCHIVED
}

// Models
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String    @map("password_hash")
  name          String?
  role          Role      @default(USER)
  emailVerified DateTime? @map("email_verified")
  image         String?
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  posts         Post[]
  comments      Comment[]
  sessions      Session[]

  @@map("users")
}

model Post {
  id          String   @id @default(cuid())
  title       String   @db.VarChar(255)
  slug        String   @unique
  content     String?  @db.Text
  excerpt     String?  @db.VarChar(500)
  status      Status   @default(DRAFT)
  publishedAt DateTime? @map("published_at")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  // Foreign keys
  authorId    String   @map("author_id")
  author      User     @relation(fields: [authorId], references: [id], onDelete: Cascade)

  // Relations
  comments    Comment[]
  tags        Tag[]

  // Indexes
  @@index([authorId])
  @@index([status, publishedAt(sort: Desc)])
  @@index([slug])

  @@map("posts")
}

model Comment {
  id        String   @id @default(cuid())
  content   String   @db.Text
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  // Foreign keys
  authorId  String   @map("author_id")
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  postId    String   @map("post_id")
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)

  // Self-referential for replies
  parentId  String?  @map("parent_id")
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  replies   Comment[] @relation("CommentReplies")

  @@index([postId])
  @@index([authorId])
  @@index([parentId])

  @@map("comments")
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique @db.VarChar(50)
  slug  String @unique @db.VarChar(50)
  posts Post[]

  @@map("tags")
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique @map("session_token")
  userId       String   @map("user_id")
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("sessions")
}
```

## Relationship Patterns

### One-to-Many
```prisma
model User {
  id    String @id @default(cuid())
  posts Post[] // One user has many posts
}

model Post {
  id       String @id @default(cuid())
  authorId String
  author   User   @relation(fields: [authorId], references: [id])
}
```

### Many-to-Many (Implicit)
```prisma
model Post {
  id   String @id
  tags Tag[]  // Prisma creates join table
}

model Tag {
  id    String @id
  posts Post[]
}
```

### Many-to-Many (Explicit)
```prisma
model Post {
  id       String     @id
  postTags PostTag[]
}

model Tag {
  id       String     @id
  postTags PostTag[]
}

model PostTag {
  postId    String
  tagId     String
  assignedAt DateTime @default(now())
  assignedBy String?

  post Post @relation(fields: [postId], references: [id])
  tag  Tag  @relation(fields: [tagId], references: [id])

  @@id([postId, tagId])
}
```

### Self-Referential
```prisma
// Categories with parent/child
model Category {
  id       String     @id @default(cuid())
  name     String
  parentId String?
  parent   Category?  @relation("CategoryChildren", fields: [parentId], references: [id])
  children Category[] @relation("CategoryChildren")
}

// Following system
model User {
  id        String   @id
  followers Follow[] @relation("Following")
  following Follow[] @relation("Followers")
}

model Follow {
  followerId  String
  followingId String
  createdAt   DateTime @default(now())

  follower  User @relation("Following", fields: [followerId], references: [id])
  following User @relation("Followers", fields: [followingId], references: [id])

  @@id([followerId, followingId])
}
```

## Raw SQL Schema

```sql
-- users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  role VARCHAR(50) DEFAULT 'USER' CHECK (role IN ('USER', 'ADMIN', 'MODERATOR')),
  email_verified TIMESTAMP WITH TIME ZONE,
  image TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

-- posts table
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  content TEXT,
  excerpt VARCHAR(500),
  status VARCHAR(50) DEFAULT 'DRAFT' CHECK (status IN ('DRAFT', 'PUBLISHED', 'ARCHIVED')),
  published_at TIMESTAMP WITH TIME ZONE,
  author_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_status_published ON posts(status, published_at DESC);
CREATE INDEX idx_posts_slug ON posts(slug);

-- tags table
CREATE TABLE tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(50) UNIQUE NOT NULL,
  slug VARCHAR(50) UNIQUE NOT NULL
);

-- post_tags join table
CREATE TABLE post_tags (
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (post_id, tag_id)
);

CREATE INDEX idx_post_tags_tag ON post_tags(tag_id);

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER posts_updated_at
  BEFORE UPDATE ON posts
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

## Normalization Guidelines

### 1NF - First Normal Form
- No repeating groups
- Each cell contains single value

### 2NF - Second Normal Form
- 1NF + No partial dependencies
- All non-key attributes depend on entire primary key

### 3NF - Third Normal Form
- 2NF + No transitive dependencies
- Non-key attributes depend only on primary key

### When to Denormalize
- Read-heavy workloads
- Expensive JOIN operations
- Caching computed values (with refresh strategy)

## Quality Checklist

- [ ] Primary keys defined (prefer UUID/CUID)
- [ ] Foreign keys with proper ON DELETE
- [ ] Unique constraints where needed
- [ ] NOT NULL on required fields
- [ ] Indexes on foreign keys
- [ ] Indexes on frequently queried columns
- [ ] Timestamps (created_at, updated_at)
- [ ] Soft delete if needed
- [ ] Appropriate data types and sizes
- [ ] Schema follows naming conventions
