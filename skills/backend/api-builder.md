---
name: api-builder
description: |
  Build RESTful API endpoints with proper structure, validation, and documentation.
  Creates consistent, well-organized API routes.
---

# API Builder Skill

## Purpose
Create well-structured, consistent REST API endpoints.

## API Structure

### Next.js App Router
```
app/
├── api/
│   ├── auth/
│   │   ├── login/route.ts
│   │   ├── register/route.ts
│   │   └── logout/route.ts
│   ├── users/
│   │   ├── route.ts           # GET (list), POST (create)
│   │   └── [id]/
│   │       └── route.ts       # GET, PUT, DELETE
│   └── posts/
│       ├── route.ts
│       └── [id]/route.ts
```

### Express.js
```
src/
├── routes/
│   ├── index.ts
│   ├── auth.routes.ts
│   ├── users.routes.ts
│   └── posts.routes.ts
├── controllers/
│   ├── auth.controller.ts
│   ├── users.controller.ts
│   └── posts.controller.ts
├── services/
│   ├── auth.service.ts
│   ├── users.service.ts
│   └── posts.service.ts
└── middleware/
    ├── auth.ts
    └── validate.ts
```

## RESTful Conventions

| Method | Endpoint | Action | Response |
|--------|----------|--------|----------|
| GET | /api/resources | List all | 200 + array |
| GET | /api/resources/:id | Get one | 200 + object |
| POST | /api/resources | Create | 201 + object |
| PUT | /api/resources/:id | Replace | 200 + object |
| PATCH | /api/resources/:id | Update | 200 + object |
| DELETE | /api/resources/:id | Delete | 204 no content |

## Next.js API Routes

### Collection Route (GET, POST)
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { createUserSchema } from '@/lib/validations/user';
import { auth } from '@/lib/auth';

// GET /api/users - List users
export async function GET(request: NextRequest) {
  try {
    // Auth check
    const session = await auth();
    if (!session) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Parse query params
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const search = searchParams.get('search') || '';

    // Query with pagination
    const skip = (page - 1) * limit;
    const where = search
      ? { OR: [{ name: { contains: search } }, { email: { contains: search } }] }
      : {};

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip,
        take: limit,
        select: { id: true, email: true, name: true, createdAt: true },
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count({ where }),
    ]);

    return NextResponse.json({
      data: users,
      meta: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    });
  } catch (error) {
    console.error('GET /api/users error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

// POST /api/users - Create user
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    // Validate input
    const result = createUserSchema.safeParse(body);
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.flatten() },
        { status: 400 }
      );
    }

    const { email, name, password } = result.data;

    // Check for existing user
    const existing = await prisma.user.findUnique({ where: { email } });
    if (existing) {
      return NextResponse.json({ error: 'Email already registered' }, { status: 409 });
    }

    // Hash password and create
    const passwordHash = await bcrypt.hash(password, 12);
    const user = await prisma.user.create({
      data: { email, name, passwordHash },
      select: { id: true, email: true, name: true, createdAt: true },
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    console.error('POST /api/users error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

### Resource Route (GET, PUT, DELETE)
```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { updateUserSchema } from '@/lib/validations/user';

interface Params {
  params: { id: string };
}

// GET /api/users/:id
export async function GET(request: NextRequest, { params }: Params) {
  try {
    const user = await prisma.user.findUnique({
      where: { id: params.id },
      select: { id: true, email: true, name: true, createdAt: true },
    });

    if (!user) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }

    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

// PUT /api/users/:id
export async function PUT(request: NextRequest, { params }: Params) {
  try {
    const body = await request.json();

    const result = updateUserSchema.safeParse(body);
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.flatten() },
        { status: 400 }
      );
    }

    const user = await prisma.user.update({
      where: { id: params.id },
      data: result.data,
      select: { id: true, email: true, name: true, createdAt: true },
    });

    return NextResponse.json(user);
  } catch (error) {
    if (error.code === 'P2025') {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

// DELETE /api/users/:id
export async function DELETE(request: NextRequest, { params }: Params) {
  try {
    await prisma.user.delete({ where: { id: params.id } });
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    if (error.code === 'P2025') {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

## Response Formats

### Success Response
```typescript
// Single resource
{ "id": "...", "name": "...", ... }

// Collection with pagination
{
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}
```

### Error Response
```typescript
{
  "error": "Human readable message",
  "code": "MACHINE_READABLE_CODE",
  "details": { ... } // Optional validation details
}
```

## Input Validation

```typescript
// lib/validations/user.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email('Invalid email'),
  name: z.string().min(2, 'Name too short').max(100),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

export const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
});

export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
```

## Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Not allowed |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 500 | Server Error | Unexpected error |

## Quality Checklist

- [ ] RESTful naming conventions
- [ ] Input validation on all writes
- [ ] Proper status codes
- [ ] Consistent response format
- [ ] Error handling
- [ ] Auth checks where needed
- [ ] Pagination for lists
- [ ] No sensitive data in responses
