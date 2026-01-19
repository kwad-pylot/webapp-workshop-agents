---
name: middleware-creator
description: |
  Create middleware functions for request processing, validation, and cross-cutting concerns.
  Handles auth, logging, rate limiting, and request transformation.
---

# Middleware Creator Skill

## Purpose
Create reusable middleware for common request processing tasks.

## Common Middleware Types

| Middleware | Purpose |
|------------|---------|
| Authentication | Verify JWT/session |
| Authorization | Check permissions |
| Validation | Validate request body |
| Rate Limiting | Prevent abuse |
| Logging | Request/response logging |
| Error Handling | Catch and format errors |
| CORS | Cross-origin requests |

## Next.js Middleware

### Global Middleware
```typescript
// middleware.ts (root level)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Get pathname
  const { pathname } = request.nextUrl;

  // Public paths that don't require auth
  const publicPaths = ['/login', '/register', '/api/auth'];
  const isPublicPath = publicPaths.some(path => pathname.startsWith(path));

  // Get token from cookie
  const token = request.cookies.get('accessToken')?.value;

  // Redirect unauthenticated users
  if (!isPublicPath && !token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Redirect authenticated users away from login
  if (isPublicPath && token && pathname !== '/api/auth') {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - _next/static (static files)
     * - _next/image (image optimization)
     * - favicon.ico (favicon)
     * - public files
     */
    '/((?!_next/static|_next/image|favicon.ico|public).*)',
  ],
};
```

### API Route Middleware Wrapper
```typescript
// lib/middleware/with-auth.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyAccessToken } from '@/lib/auth/jwt';

type Handler = (
  request: NextRequest & { user: { userId: string; email: string } },
  context: { params: Record<string, string> }
) => Promise<NextResponse>;

export function withAuth(handler: Handler) {
  return async (request: NextRequest, context: { params: Record<string, string> }) => {
    const authHeader = request.headers.get('authorization');

    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json(
        { error: 'Authorization header required' },
        { status: 401 }
      );
    }

    try {
      const token = authHeader.slice(7);
      const user = verifyAccessToken(token);

      // Extend request with user
      const extendedRequest = Object.assign(request, { user });
      return handler(extendedRequest, context);
    } catch (error) {
      return NextResponse.json(
        { error: 'Invalid or expired token' },
        { status: 401 }
      );
    }
  };
}

// Usage in route:
// export const GET = withAuth(async (request) => {
//   console.log(request.user.userId);
//   return NextResponse.json({ ... });
// });
```

### Validation Middleware
```typescript
// lib/middleware/with-validation.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

export function withValidation<T extends z.ZodSchema>(schema: T) {
  return function (handler: (request: NextRequest & { validatedBody: z.infer<T> }, context: any) => Promise<NextResponse>) {
    return async (request: NextRequest, context: any) => {
      try {
        const body = await request.json();
        const validated = schema.parse(body);

        const extendedRequest = Object.assign(request, { validatedBody: validated });
        return handler(extendedRequest, context);
      } catch (error) {
        if (error instanceof z.ZodError) {
          return NextResponse.json(
            { error: 'Validation failed', details: error.flatten() },
            { status: 400 }
          );
        }
        return NextResponse.json(
          { error: 'Invalid JSON body' },
          { status: 400 }
        );
      }
    };
  };
}

// Usage:
// export const POST = withValidation(createUserSchema)(async (request) => {
//   const data = request.validatedBody; // Type-safe!
//   return NextResponse.json({ ... });
// });
```

### Composable Middleware
```typescript
// lib/middleware/compose.ts
import { NextRequest, NextResponse } from 'next/server';

type Middleware = (
  request: NextRequest,
  context: any,
  next: () => Promise<NextResponse>
) => Promise<NextResponse>;

type Handler = (request: NextRequest, context: any) => Promise<NextResponse>;

export function compose(...middlewares: Middleware[]) {
  return function (handler: Handler) {
    return async (request: NextRequest, context: any): Promise<NextResponse> => {
      let index = 0;

      async function next(): Promise<NextResponse> {
        if (index < middlewares.length) {
          const middleware = middlewares[index++];
          return middleware(request, context, next);
        }
        return handler(request, context);
      }

      return next();
    };
  };
}

// Usage:
// export const POST = compose(
//   authMiddleware,
//   validationMiddleware(schema),
//   rateLimitMiddleware
// )(async (request) => {
//   return NextResponse.json({ ... });
// });
```

## Express.js Middleware

### Rate Limiter
```typescript
// middleware/rate-limit.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { redis } from '@/lib/redis';

export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: { error: 'Too many requests, please try again later' },
  standardHeaders: true,
  legacyHeaders: false,
});

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts per 15 min
  message: { error: 'Too many login attempts' },
  store: new RedisStore({
    sendCommand: (...args: string[]) => redis.call(...args),
  }),
});
```

### Request Logger
```typescript
// middleware/logger.ts
import { Request, Response, NextFunction } from 'express';

export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('user-agent'),
      ip: req.ip,
    }));
  });

  next();
}
```

### Error Handler
```typescript
// middleware/error-handler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '@/errors/app-error';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Log error
  console.error({
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
  });

  // Known errors
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
    });
  }

  // Prisma errors
  if (err.name === 'PrismaClientKnownRequestError') {
    return res.status(400).json({
      error: 'Database error',
      code: 'DB_ERROR',
    });
  }

  // Unexpected errors
  const message = process.env.NODE_ENV === 'production'
    ? 'Internal server error'
    : err.message;

  res.status(500).json({ error: message });
}
```

### CORS Configuration
```typescript
// middleware/cors.ts
import cors from 'cors';

export const corsMiddleware = cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400, // 24 hours
});
```

## Quality Checklist

- [ ] Middleware is reusable
- [ ] Error handling doesn't leak info
- [ ] Rate limiting configured
- [ ] Logging doesn't include sensitive data
- [ ] CORS properly configured
- [ ] Auth middleware validates tokens
- [ ] Validation returns helpful errors
