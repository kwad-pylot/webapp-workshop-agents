---
name: integration-test-writer
description: |
  Write integration and E2E tests for APIs, components with external dependencies,
  and complete user flows.
---

# Integration Test Writer Skill

## Purpose
Test how multiple units work together, including API endpoints, database operations, and user flows.

## API Integration Tests

### Setup with Supertest
```typescript
// tests/api/setup.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

const prisma = new PrismaClient();

export async function setupTestDatabase() {
  // Run migrations on test database
  execSync('npx prisma migrate deploy', {
    env: { ...process.env, DATABASE_URL: process.env.TEST_DATABASE_URL },
  });
}

export async function cleanupTestDatabase() {
  // Clean tables in correct order (respect foreign keys)
  await prisma.comment.deleteMany();
  await prisma.post.deleteMany();
  await prisma.user.deleteMany();
}

export { prisma };
```

### User API Tests
```typescript
// tests/api/users.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '@/app';
import { prisma, cleanupTestDatabase } from './setup';
import bcrypt from 'bcryptjs';

describe('Users API', () => {
  let authToken: string;
  let testUser: { id: string; email: string };

  beforeAll(async () => {
    await cleanupTestDatabase();

    // Create test user
    const passwordHash = await bcrypt.hash('password123', 12);
    testUser = await prisma.user.create({
      data: {
        email: 'test@example.com',
        name: 'Test User',
        passwordHash,
      },
    });

    // Get auth token
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'password123' });
    authToken = loginRes.body.accessToken;
  });

  afterAll(async () => {
    await cleanupTestDatabase();
    await prisma.$disconnect();
  });

  describe('GET /api/users', () => {
    it('returns 401 without authentication', async () => {
      const res = await request(app).get('/api/users');

      expect(res.status).toBe(401);
      expect(res.body.error).toBe('Unauthorized');
    });

    it('returns paginated users list', async () => {
      const res = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.status).toBe(200);
      expect(res.body.data).toBeInstanceOf(Array);
      expect(res.body.meta).toHaveProperty('page');
      expect(res.body.meta).toHaveProperty('total');
    });

    it('supports pagination parameters', async () => {
      const res = await request(app)
        .get('/api/users?page=1&limit=5')
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.status).toBe(200);
      expect(res.body.meta.page).toBe(1);
      expect(res.body.meta.limit).toBe(5);
    });

    it('supports search parameter', async () => {
      const res = await request(app)
        .get('/api/users?search=test')
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.status).toBe(200);
      expect(res.body.data.some((u: any) => u.email.includes('test'))).toBe(true);
    });
  });

  describe('POST /api/users', () => {
    it('creates a new user with valid data', async () => {
      const userData = {
        email: 'newuser@example.com',
        name: 'New User',
        password: 'securePassword123',
      };

      const res = await request(app)
        .post('/api/users')
        .send(userData);

      expect(res.status).toBe(201);
      expect(res.body.email).toBe(userData.email);
      expect(res.body.name).toBe(userData.name);
      expect(res.body.password).toBeUndefined();
      expect(res.body.passwordHash).toBeUndefined();
    });

    it('returns 400 for invalid email', async () => {
      const res = await request(app)
        .post('/api/users')
        .send({ email: 'invalid', password: 'password123' });

      expect(res.status).toBe(400);
      expect(res.body.error).toBe('Validation failed');
      expect(res.body.details.email).toBeDefined();
    });

    it('returns 409 for duplicate email', async () => {
      const res = await request(app)
        .post('/api/users')
        .send({ email: 'test@example.com', password: 'password123' });

      expect(res.status).toBe(409);
    });
  });

  describe('GET /api/users/:id', () => {
    it('returns user by ID', async () => {
      const res = await request(app)
        .get(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.status).toBe(200);
      expect(res.body.id).toBe(testUser.id);
      expect(res.body.email).toBe(testUser.email);
    });

    it('returns 404 for non-existent user', async () => {
      const res = await request(app)
        .get('/api/users/nonexistent-id')
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.status).toBe(404);
    });
  });

  describe('PATCH /api/users/:id', () => {
    it('updates user profile', async () => {
      const res = await request(app)
        .patch(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Updated Name' });

      expect(res.status).toBe(200);
      expect(res.body.name).toBe('Updated Name');
    });

    it('returns 401 for unauthorized update', async () => {
      // Create another user
      const otherUser = await prisma.user.create({
        data: {
          email: 'other@example.com',
          passwordHash: await bcrypt.hash('password', 12),
        },
      });

      const res = await request(app)
        .patch(`/api/users/${otherUser.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Hacked' });

      expect(res.status).toBe(401);
    });
  });
});
```

## Component Integration Tests

```tsx
// tests/components/UserList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, beforeAll, afterAll, afterEach } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserList } from '@/components/UserList';

const mockUsers = [
  { id: '1', name: 'Alice', email: 'alice@example.com' },
  { id: '2', name: 'Bob', email: 'bob@example.com' },
];

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json({
      data: mockUsers,
      meta: { page: 1, limit: 10, total: 2, totalPages: 1 },
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>
  );
}

describe('UserList', () => {
  it('shows loading state', () => {
    renderWithProviders(<UserList />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('renders users after loading', async () => {
    renderWithProviders(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument();
      expect(screen.getByText('Bob')).toBeInTheDocument();
    });
  });

  it('shows error state on failure', async () => {
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json({ error: 'Server error' }, { status: 500 });
      })
    );

    renderWithProviders(<UserList />);

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });

  it('shows empty state when no users', async () => {
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json({
          data: [],
          meta: { page: 1, limit: 10, total: 0, totalPages: 0 },
        });
      })
    );

    renderWithProviders(<UserList />);

    await waitFor(() => {
      expect(screen.getByText(/no users/i)).toBeInTheDocument();
    });
  });
});
```

## E2E Tests with Playwright

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can sign up', async ({ page }) => {
    const email = `test-${Date.now()}@example.com`;

    await page.goto('/signup');

    await page.fill('[name="email"]', email);
    await page.fill('[name="password"]', 'SecurePassword123!');
    await page.fill('[name="confirmPassword"]', 'SecurePassword123!');
    await page.click('button[type="submit"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome')).toBeVisible();
  });

  test('user can log in and log out', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');

    // Log out
    await page.click('[data-testid="user-menu"]');
    await page.click('text=Log out');

    await expect(page).toHaveURL('/login');
  });

  test('shows validation errors', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="email"]', 'invalid');
    await page.click('button[type="submit"]');

    await expect(page.getByText('Invalid email')).toBeVisible();
  });
});

// e2e/posts.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Posts', () => {
  test.beforeEach(async ({ page }) => {
    // Log in before each test
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/dashboard');
  });

  test('user can create a post', async ({ page }) => {
    await page.click('text=New Post');

    await page.fill('[name="title"]', 'My Test Post');
    await page.fill('[name="content"]', 'This is the content of my test post.');
    await page.click('button:has-text("Publish")');

    await expect(page.getByText('Post published!')).toBeVisible();
    await expect(page.getByText('My Test Post')).toBeVisible();
  });

  test('user can edit a post', async ({ page }) => {
    await page.goto('/posts');
    await page.click('[data-testid="post-menu-1"]');
    await page.click('text=Edit');

    await page.fill('[name="title"]', 'Updated Title');
    await page.click('button:has-text("Save")');

    await expect(page.getByText('Post updated!')).toBeVisible();
    await expect(page.getByText('Updated Title')).toBeVisible();
  });
});
```

## Quality Checklist

- [ ] Tests cover happy path
- [ ] Tests cover error cases
- [ ] Database cleaned between tests
- [ ] Auth handled properly
- [ ] Mocks reset between tests
- [ ] E2E tests are stable
- [ ] Tests are independent
- [ ] CI/CD runs tests automatically
