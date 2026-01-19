---
name: seeder-creator
description: |
  Create database seed data for development and testing.
  Generates realistic test data with proper relationships.
---

# Seeder Creator Skill

## Purpose
Create seed data for development, testing, and demos.

## Prisma Seed Setup

### Configuration
```json
// package.json
{
  "prisma": {
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

### Basic Seed File
```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  console.log('🌱 Seeding database...');

  // Clean existing data (optional, dev only)
  await prisma.comment.deleteMany();
  await prisma.post.deleteMany();
  await prisma.user.deleteMany();

  // Create admin user
  const admin = await prisma.user.create({
    data: {
      email: 'admin@example.com',
      name: 'Admin User',
      passwordHash: await bcrypt.hash('admin123', 12),
      role: 'ADMIN',
      emailVerified: new Date(),
    },
  });

  // Create regular users
  const users = await Promise.all([
    prisma.user.create({
      data: {
        email: 'alice@example.com',
        name: 'Alice Johnson',
        passwordHash: await bcrypt.hash('password123', 12),
        role: 'USER',
        emailVerified: new Date(),
      },
    }),
    prisma.user.create({
      data: {
        email: 'bob@example.com',
        name: 'Bob Smith',
        passwordHash: await bcrypt.hash('password123', 12),
        role: 'USER',
      },
    }),
  ]);

  // Create posts
  const posts = await Promise.all([
    prisma.post.create({
      data: {
        title: 'Getting Started with Prisma',
        slug: 'getting-started-with-prisma',
        content: 'Prisma is a modern database toolkit...',
        excerpt: 'Learn how to use Prisma ORM',
        status: 'PUBLISHED',
        publishedAt: new Date(),
        authorId: admin.id,
      },
    }),
    prisma.post.create({
      data: {
        title: 'Building APIs with Next.js',
        slug: 'building-apis-with-nextjs',
        content: 'Next.js provides powerful API routes...',
        excerpt: 'Create REST APIs in Next.js',
        status: 'PUBLISHED',
        publishedAt: new Date(),
        authorId: users[0].id,
      },
    }),
    prisma.post.create({
      data: {
        title: 'Draft Post',
        slug: 'draft-post',
        content: 'This is a draft...',
        status: 'DRAFT',
        authorId: users[1].id,
      },
    }),
  ]);

  // Create comments
  await prisma.comment.createMany({
    data: [
      {
        content: 'Great article!',
        postId: posts[0].id,
        authorId: users[0].id,
      },
      {
        content: 'Very helpful, thanks!',
        postId: posts[0].id,
        authorId: users[1].id,
      },
      {
        content: 'I learned a lot from this.',
        postId: posts[1].id,
        authorId: admin.id,
      },
    ],
  });

  console.log('✅ Seed completed!');
  console.log({
    users: await prisma.user.count(),
    posts: await prisma.post.count(),
    comments: await prisma.comment.count(),
  });
}

main()
  .catch((e) => {
    console.error('❌ Seed failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

### Running Seeds
```bash
# Run seed
npx prisma db seed

# Reset and seed
npx prisma migrate reset
```

## Faker for Realistic Data

```typescript
// prisma/seed.ts
import { faker } from '@faker-js/faker';
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

// Deterministic seed for reproducible data
faker.seed(12345);

async function main() {
  // Create many users
  const PASSWORD_HASH = await bcrypt.hash('password123', 12);

  const users = await Promise.all(
    Array.from({ length: 20 }).map(() =>
      prisma.user.create({
        data: {
          email: faker.internet.email().toLowerCase(),
          name: faker.person.fullName(),
          passwordHash: PASSWORD_HASH,
          role: faker.helpers.arrayElement(['USER', 'USER', 'USER', 'ADMIN']),
          image: faker.image.avatar(),
          createdAt: faker.date.past({ years: 1 }),
        },
      })
    )
  );

  // Create posts for each user
  const posts = [];
  for (const user of users) {
    const userPosts = await Promise.all(
      Array.from({ length: faker.number.int({ min: 0, max: 5 }) }).map(() => {
        const title = faker.lorem.sentence({ min: 3, max: 8 });
        const isPublished = faker.datatype.boolean(0.7);
        return prisma.post.create({
          data: {
            title,
            slug: faker.helpers.slugify(title).toLowerCase(),
            content: faker.lorem.paragraphs({ min: 3, max: 10 }),
            excerpt: faker.lorem.paragraph(),
            status: isPublished ? 'PUBLISHED' : 'DRAFT',
            publishedAt: isPublished ? faker.date.past({ years: 1 }) : null,
            authorId: user.id,
            createdAt: faker.date.past({ years: 1 }),
          },
        });
      })
    );
    posts.push(...userPosts);
  }

  // Create comments on published posts
  const publishedPosts = posts.filter((p) => p.status === 'PUBLISHED');
  for (const post of publishedPosts) {
    const commentCount = faker.number.int({ min: 0, max: 10 });
    await prisma.comment.createMany({
      data: Array.from({ length: commentCount }).map(() => ({
        content: faker.lorem.paragraph(),
        postId: post.id,
        authorId: faker.helpers.arrayElement(users).id,
        createdAt: faker.date.between({
          from: post.createdAt,
          to: new Date(),
        }),
      })),
    });
  }

  console.log('Seeded:', {
    users: users.length,
    posts: posts.length,
    comments: await prisma.comment.count(),
  });
}
```

## Factory Pattern

```typescript
// prisma/factories/user.factory.ts
import { faker } from '@faker-js/faker';
import { Prisma } from '@prisma/client';

export function createUserData(
  overrides?: Partial<Prisma.UserCreateInput>
): Prisma.UserCreateInput {
  return {
    email: faker.internet.email().toLowerCase(),
    name: faker.person.fullName(),
    passwordHash: '$2a$12$...', // Pre-hashed password
    role: 'USER',
    image: faker.image.avatar(),
    createdAt: faker.date.past({ years: 1 }),
    ...overrides,
  };
}

export function createPostData(
  authorId: string,
  overrides?: Partial<Prisma.PostCreateInput>
): Prisma.PostCreateInput {
  const title = faker.lorem.sentence({ min: 3, max: 8 });
  return {
    title,
    slug: faker.helpers.slugify(title).toLowerCase(),
    content: faker.lorem.paragraphs(5),
    excerpt: faker.lorem.paragraph(),
    status: 'DRAFT',
    author: { connect: { id: authorId } },
    createdAt: faker.date.past({ years: 1 }),
    ...overrides,
  };
}

// Usage in seed:
const user = await prisma.user.create({
  data: createUserData({ role: 'ADMIN' }),
});

const post = await prisma.post.create({
  data: createPostData(user.id, { status: 'PUBLISHED' }),
});
```

## Environment-Specific Seeds

```typescript
// prisma/seed.ts
async function main() {
  const env = process.env.NODE_ENV || 'development';

  // Always create admin
  await createAdminUser();

  if (env === 'development') {
    await seedDevelopmentData();
  } else if (env === 'test') {
    await seedTestData();
  } else if (env === 'staging') {
    await seedStagingData();
  }
}

async function seedDevelopmentData() {
  // Lots of varied data for development
  await createManyUsers(50);
  await createManyPosts(200);
}

async function seedTestData() {
  // Minimal, predictable data for tests
  await createFixedTestUsers();
  await createFixedTestPosts();
}

async function seedStagingData() {
  // Demo data for staging
  await createDemoUsers();
  await createDemoPosts();
}
```

## SQL Seeds

```sql
-- seeds/001_users.sql
INSERT INTO users (id, email, password_hash, name, role, created_at)
VALUES
  ('user_admin', 'admin@example.com', '$2a$12$...', 'Admin', 'ADMIN', NOW()),
  ('user_alice', 'alice@example.com', '$2a$12$...', 'Alice', 'USER', NOW()),
  ('user_bob', 'bob@example.com', '$2a$12$...', 'Bob', 'USER', NOW())
ON CONFLICT (email) DO NOTHING;

-- seeds/002_posts.sql
INSERT INTO posts (id, title, slug, content, status, author_id, created_at)
VALUES
  ('post_1', 'First Post', 'first-post', 'Content...', 'PUBLISHED', 'user_admin', NOW()),
  ('post_2', 'Second Post', 'second-post', 'Content...', 'DRAFT', 'user_alice', NOW())
ON CONFLICT (slug) DO NOTHING;
```

## Quality Checklist

- [ ] Seed is idempotent (can run multiple times)
- [ ] Uses upsert or ON CONFLICT for safety
- [ ] Respects foreign key relationships
- [ ] Creates realistic data variety
- [ ] Has deterministic mode for tests
- [ ] Handles errors gracefully
- [ ] Provides progress feedback
- [ ] Documents seed users/passwords
