---
name: auth-implementer
description: |
  Implement secure authentication systems including JWT, sessions, and OAuth.
  Handles login, registration, password reset, and token management.
---

# Auth Implementer Skill

## Purpose
Implement secure, robust authentication systems.

## Authentication Options

| Method | Use Case | Pros | Cons |
|--------|----------|------|------|
| JWT | APIs, mobile | Stateless, scalable | Can't revoke easily |
| Sessions | Traditional web | Easy revocation | Server state |
| NextAuth | Next.js apps | Easy OAuth, flexible | Next.js specific |
| Clerk/Auth0 | Quick setup | Full-featured | Cost, vendor lock |

## JWT Implementation

### Token Generation
```typescript
// lib/auth/jwt.ts
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_EXPIRES_IN = '15m';
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;
const REFRESH_EXPIRES_IN = '7d';

interface TokenPayload {
  userId: string;
  email: string;
}

export function generateTokens(payload: TokenPayload) {
  const accessToken = jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRES_IN,
  });

  const refreshToken = jwt.sign(
    { userId: payload.userId },
    REFRESH_SECRET,
    { expiresIn: REFRESH_EXPIRES_IN }
  );

  return { accessToken, refreshToken };
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, JWT_SECRET) as TokenPayload;
}

export function verifyRefreshToken(token: string): { userId: string } {
  return jwt.verify(token, REFRESH_SECRET) as { userId: string };
}
```

### Auth Service
```typescript
// services/auth.service.ts
import bcrypt from 'bcryptjs';
import { prisma } from '@/lib/prisma';
import { generateTokens, verifyRefreshToken } from '@/lib/auth/jwt';

export class AuthService {
  async register(email: string, password: string, name?: string) {
    // Check existing
    const existing = await prisma.user.findUnique({ where: { email } });
    if (existing) {
      throw new Error('Email already registered');
    }

    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);

    // Create user
    const user = await prisma.user.create({
      data: { email, passwordHash, name },
    });

    // Generate tokens
    const tokens = generateTokens({ userId: user.id, email: user.email });

    return {
      user: { id: user.id, email: user.email, name: user.name },
      ...tokens,
    };
  }

  async login(email: string, password: string) {
    // Find user
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      throw new Error('Invalid credentials');
    }

    // Verify password
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) {
      throw new Error('Invalid credentials');
    }

    // Generate tokens
    const tokens = generateTokens({ userId: user.id, email: user.email });

    // Store refresh token hash (optional, for revocation)
    await prisma.refreshToken.create({
      data: {
        token: tokens.refreshToken,
        userId: user.id,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    });

    return {
      user: { id: user.id, email: user.email, name: user.name },
      ...tokens,
    };
  }

  async refresh(refreshToken: string) {
    // Verify token
    const payload = verifyRefreshToken(refreshToken);

    // Check if token exists in DB (not revoked)
    const storedToken = await prisma.refreshToken.findFirst({
      where: { token: refreshToken, userId: payload.userId },
    });
    if (!storedToken) {
      throw new Error('Invalid refresh token');
    }

    // Get user
    const user = await prisma.user.findUnique({ where: { id: payload.userId } });
    if (!user) {
      throw new Error('User not found');
    }

    // Generate new tokens
    const tokens = generateTokens({ userId: user.id, email: user.email });

    // Rotate refresh token
    await prisma.refreshToken.delete({ where: { id: storedToken.id } });
    await prisma.refreshToken.create({
      data: {
        token: tokens.refreshToken,
        userId: user.id,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    });

    return tokens;
  }

  async logout(refreshToken: string) {
    await prisma.refreshToken.deleteMany({ where: { token: refreshToken } });
  }

  async forgotPassword(email: string) {
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) return; // Don't reveal if email exists

    // Generate reset token
    const resetToken = crypto.randomBytes(32).toString('hex');
    const hashedToken = await bcrypt.hash(resetToken, 10);

    await prisma.passwordReset.create({
      data: {
        userId: user.id,
        token: hashedToken,
        expiresAt: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
      },
    });

    // Send email (implement with your email provider)
    // await sendPasswordResetEmail(email, resetToken);
  }

  async resetPassword(token: string, newPassword: string) {
    // Find valid reset token
    const resetRecords = await prisma.passwordReset.findMany({
      where: { expiresAt: { gt: new Date() } },
      include: { user: true },
    });

    let validRecord = null;
    for (const record of resetRecords) {
      if (await bcrypt.compare(token, record.token)) {
        validRecord = record;
        break;
      }
    }

    if (!validRecord) {
      throw new Error('Invalid or expired reset token');
    }

    // Update password
    const passwordHash = await bcrypt.hash(newPassword, 12);
    await prisma.user.update({
      where: { id: validRecord.userId },
      data: { passwordHash },
    });

    // Delete used token
    await prisma.passwordReset.delete({ where: { id: validRecord.id } });
  }
}
```

### Auth API Routes
```typescript
// app/api/auth/register/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { AuthService } from '@/services/auth.service';
import { registerSchema } from '@/lib/validations/auth';

const authService = new AuthService();

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    const result = registerSchema.safeParse(body);
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.flatten() },
        { status: 400 }
      );
    }

    const { email, password, name } = result.data;
    const data = await authService.register(email, password, name);

    // Set refresh token in HTTP-only cookie
    const response = NextResponse.json({
      user: data.user,
      accessToken: data.accessToken,
    }, { status: 201 });

    response.cookies.set('refreshToken', data.refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 7 * 24 * 60 * 60, // 7 days
      path: '/',
    });

    return response;
  } catch (error) {
    if (error.message === 'Email already registered') {
      return NextResponse.json({ error: error.message }, { status: 409 });
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

## Auth Middleware
```typescript
// middleware/auth.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyAccessToken } from '@/lib/auth/jwt';

export function withAuth(handler: Function) {
  return async (request: NextRequest, context: any) => {
    const authHeader = request.headers.get('authorization');

    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const token = authHeader.substring(7);

    try {
      const payload = verifyAccessToken(token);
      // Attach user to request (via headers for Next.js)
      const headers = new Headers(request.headers);
      headers.set('x-user-id', payload.userId);
      headers.set('x-user-email', payload.email);

      return handler(request, context);
    } catch (error) {
      return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
    }
  };
}
```

## Security Best Practices

1. **Password Storage**: Always use bcrypt with 12+ rounds
2. **Token Security**: Use HTTP-only cookies for refresh tokens
3. **Rate Limiting**: Limit login attempts (5 per 15 min)
4. **HTTPS**: Always use HTTPS in production
5. **CORS**: Configure properly for your domains
6. **Token Expiry**: Short access tokens (15min), longer refresh (7d)
7. **Logout**: Invalidate refresh tokens on logout

## Quality Checklist

- [ ] Passwords hashed with bcrypt
- [ ] JWT secrets are strong and env-based
- [ ] Refresh tokens in HTTP-only cookies
- [ ] Rate limiting on auth endpoints
- [ ] No password in responses
- [ ] Token rotation on refresh
- [ ] Proper error messages (no info leak)
