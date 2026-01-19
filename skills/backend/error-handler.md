---
name: error-handler
description: |
  Implement robust error handling patterns for APIs and services.
  Creates consistent error responses and proper error classification.
---

# Error Handler Skill

## Purpose
Implement consistent, informative, and secure error handling.

## Error Class Hierarchy

```typescript
// errors/index.ts

// Base application error
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public code: string = 'INTERNAL_ERROR',
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = 'AppError';
    Error.captureStackTrace(this, this.constructor);
  }
}

// 400 - Bad Request
export class BadRequestError extends AppError {
  constructor(message: string = 'Bad request', code: string = 'BAD_REQUEST') {
    super(message, 400, code);
  }
}

// 400 - Validation Error
export class ValidationError extends AppError {
  constructor(
    public details: Record<string, string[]>,
    message: string = 'Validation failed'
  ) {
    super(message, 400, 'VALIDATION_ERROR');
  }

  toJSON() {
    return {
      error: this.message,
      code: this.code,
      details: this.details,
    };
  }
}

// 401 - Unauthorized
export class UnauthorizedError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

// 403 - Forbidden
export class ForbiddenError extends AppError {
  constructor(message: string = 'Forbidden') {
    super(message, 403, 'FORBIDDEN');
  }
}

// 404 - Not Found
export class NotFoundError extends AppError {
  constructor(resource: string = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

// 409 - Conflict
export class ConflictError extends AppError {
  constructor(message: string = 'Resource already exists') {
    super(message, 409, 'CONFLICT');
  }
}

// 429 - Rate Limited
export class RateLimitError extends AppError {
  constructor(message: string = 'Too many requests') {
    super(message, 429, 'RATE_LIMITED');
  }
}

// 500 - Internal Error
export class InternalError extends AppError {
  constructor(message: string = 'Internal server error') {
    super(message, 500, 'INTERNAL_ERROR', false);
  }
}
```

## Error Response Format

```typescript
// Standard error response
interface ErrorResponse {
  error: string;           // Human-readable message
  code: string;            // Machine-readable code
  details?: unknown;       // Additional info (validation, etc.)
  requestId?: string;      // For tracking
}

// Examples:
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "details": {
    "email": ["Invalid email format"],
    "password": ["Must be at least 8 characters"]
  }
}

{
  "error": "User not found",
  "code": "NOT_FOUND",
  "requestId": "req_abc123"
}
```

## Global Error Handler

### Next.js API Routes
```typescript
// lib/api-handler.ts
import { NextRequest, NextResponse } from 'next/server';
import { AppError, ValidationError } from '@/errors';
import { ZodError } from 'zod';
import { Prisma } from '@prisma/client';

type Handler = (request: NextRequest, context: any) => Promise<NextResponse>;

export function withErrorHandler(handler: Handler): Handler {
  return async (request: NextRequest, context: any) => {
    try {
      return await handler(request, context);
    } catch (error) {
      return handleError(error, request);
    }
  };
}

function handleError(error: unknown, request: NextRequest): NextResponse {
  const requestId = crypto.randomUUID();

  // Log the error
  console.error({
    requestId,
    error: error instanceof Error ? error.message : 'Unknown error',
    stack: error instanceof Error ? error.stack : undefined,
    path: request.nextUrl.pathname,
    method: request.method,
  });

  // Handle known errors
  if (error instanceof ValidationError) {
    return NextResponse.json(error.toJSON(), { status: error.statusCode });
  }

  if (error instanceof AppError) {
    return NextResponse.json(
      {
        error: error.message,
        code: error.code,
        requestId,
      },
      { status: error.statusCode }
    );
  }

  // Handle Zod validation errors
  if (error instanceof ZodError) {
    return NextResponse.json(
      {
        error: 'Validation failed',
        code: 'VALIDATION_ERROR',
        details: error.flatten().fieldErrors,
        requestId,
      },
      { status: 400 }
    );
  }

  // Handle Prisma errors
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    return handlePrismaError(error, requestId);
  }

  // Unknown errors - don't leak details in production
  const message = process.env.NODE_ENV === 'production'
    ? 'Internal server error'
    : error instanceof Error ? error.message : 'Unknown error';

  return NextResponse.json(
    {
      error: message,
      code: 'INTERNAL_ERROR',
      requestId,
    },
    { status: 500 }
  );
}

function handlePrismaError(
  error: Prisma.PrismaClientKnownRequestError,
  requestId: string
): NextResponse {
  switch (error.code) {
    case 'P2002': // Unique constraint violation
      return NextResponse.json(
        {
          error: 'A record with this value already exists',
          code: 'CONFLICT',
          requestId,
        },
        { status: 409 }
      );

    case 'P2025': // Record not found
      return NextResponse.json(
        {
          error: 'Record not found',
          code: 'NOT_FOUND',
          requestId,
        },
        { status: 404 }
      );

    case 'P2003': // Foreign key constraint
      return NextResponse.json(
        {
          error: 'Related record not found',
          code: 'BAD_REQUEST',
          requestId,
        },
        { status: 400 }
      );

    default:
      return NextResponse.json(
        {
          error: 'Database error',
          code: 'DB_ERROR',
          requestId,
        },
        { status: 500 }
      );
  }
}
```

### Using the Error Handler
```typescript
// app/api/users/route.ts
import { withErrorHandler } from '@/lib/api-handler';
import { NotFoundError, ValidationError } from '@/errors';

export const GET = withErrorHandler(async (request) => {
  const user = await prisma.user.findUnique({ where: { id } });

  if (!user) {
    throw new NotFoundError('User');
  }

  return NextResponse.json(user);
});

export const POST = withErrorHandler(async (request) => {
  const body = await request.json();

  // Validation errors
  const result = schema.safeParse(body);
  if (!result.success) {
    throw new ValidationError(result.error.flatten().fieldErrors);
  }

  // Create user...
});
```

## Error Logging

```typescript
// lib/logger.ts
interface ErrorLog {
  level: 'error' | 'warn' | 'info';
  message: string;
  code?: string;
  stack?: string;
  requestId?: string;
  userId?: string;
  path?: string;
  method?: string;
  metadata?: Record<string, unknown>;
  timestamp: string;
}

export function logError(error: Error, context?: Partial<ErrorLog>) {
  const log: ErrorLog = {
    level: 'error',
    message: error.message,
    stack: error.stack,
    timestamp: new Date().toISOString(),
    ...context,
  };

  // In production, send to logging service
  if (process.env.NODE_ENV === 'production') {
    // sendToLoggingService(log);
  }

  console.error(JSON.stringify(log));
}
```

## Client-Side Error Handling

```typescript
// lib/api-client.ts
export class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
    public details?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

async function handleResponse<T>(response: Response): Promise<T> {
  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    throw new ApiError(
      response.status,
      body.code || 'UNKNOWN_ERROR',
      body.error || 'Request failed',
      body.details
    );
  }
  return response.json();
}

// Usage in components
try {
  await api.post('/users', data);
} catch (error) {
  if (error instanceof ApiError) {
    if (error.code === 'VALIDATION_ERROR') {
      setFormErrors(error.details);
    } else {
      toast.error(error.message);
    }
  }
}
```

## Quality Checklist

- [ ] Consistent error response format
- [ ] Error codes are documented
- [ ] Stack traces hidden in production
- [ ] Errors are logged with context
- [ ] Validation errors include field details
- [ ] Database errors are translated
- [ ] Request IDs for tracking
- [ ] Client can handle all error types
