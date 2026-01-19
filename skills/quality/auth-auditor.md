---
name: auth-auditor
description: |
  Audit authentication and authorization implementations for security issues.
  Reviews JWT handling, session management, and access control.
---

# Auth Auditor Skill

## Purpose
Review authentication systems for security vulnerabilities and best practices.

## Authentication Audit Checklist

### Password Security
- [ ] Passwords hashed with bcrypt (cost factor 12+) or Argon2
- [ ] Password requirements enforced (length, complexity)
- [ ] Password breach checking (Have I Been Pwned)
- [ ] No password hints or security questions
- [ ] Secure password reset flow

### Token Management
- [ ] JWT secret is strong (256+ bits of entropy)
- [ ] Access tokens have short expiry (15-60 minutes)
- [ ] Refresh tokens have reasonable expiry (7-30 days)
- [ ] Refresh token rotation implemented
- [ ] Tokens can be revoked

### Session Security
- [ ] Sessions have timeout
- [ ] Session ID regenerated after login
- [ ] Secure cookie flags (HttpOnly, Secure, SameSite)
- [ ] Session terminated on logout
- [ ] Concurrent session limits (if needed)

### Multi-Factor Authentication
- [ ] MFA option available
- [ ] TOTP implementation correct
- [ ] Backup codes generated
- [ ] Recovery flow secure

## Common Vulnerabilities

### Weak JWT Implementation

```typescript
// VULNERABLE: Weak secret
const token = jwt.sign(payload, 'secret123');

// VULNERABLE: No algorithm specified
const decoded = jwt.verify(token, secret);

// VULNERABLE: Accepting 'none' algorithm
const decoded = jwt.verify(token, secret, { algorithms: ['none', 'HS256'] });

// SECURE: Strong secret, explicit algorithm
const JWT_SECRET = process.env.JWT_SECRET; // 32+ random bytes
const token = jwt.sign(payload, JWT_SECRET, { algorithm: 'HS256' });
const decoded = jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] });
```

### Insecure Token Storage

```typescript
// VULNERABLE: Storing in localStorage (XSS risk)
localStorage.setItem('accessToken', token);

// VULNERABLE: Storing refresh token in localStorage
localStorage.setItem('refreshToken', refreshToken);

// SECURE: Access token in memory, refresh in HttpOnly cookie
// Server sets cookie:
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  path: '/api/auth/refresh',
  maxAge: 7 * 24 * 60 * 60 * 1000,
});

// Client stores access token in memory only
let accessToken: string | null = null;
```

### Missing Token Expiry

```typescript
// VULNERABLE: No expiry
const token = jwt.sign({ userId });

// SECURE: Short expiry
const token = jwt.sign({ userId }, secret, { expiresIn: '15m' });
```

### Insecure Password Reset

```typescript
// VULNERABLE: Predictable token
const resetToken = `reset-${userId}`;

// VULNERABLE: Token doesn't expire
await db.passwordReset.create({ token, userId });

// VULNERABLE: Same endpoint reveals if email exists
if (!user) {
  return res.status(404).json({ error: 'User not found' });
}

// SECURE Implementation
const resetToken = crypto.randomBytes(32).toString('hex');
const hashedToken = await bcrypt.hash(resetToken, 10);

await db.passwordReset.create({
  data: {
    token: hashedToken,
    userId,
    expiresAt: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
  },
});

// Same response for existing/non-existing email
return res.json({ message: 'If email exists, reset link sent' });
```

## Authorization Audit

### Access Control Checks

```typescript
// VULNERABLE: No ownership check
app.get('/api/orders/:id', async (req, res) => {
  const order = await db.order.findById(req.params.id);
  res.json(order);  // Anyone can view any order!
});

// SECURE: Check ownership
app.get('/api/orders/:id', authenticate, async (req, res) => {
  const order = await db.order.findFirst({
    where: {
      id: req.params.id,
      userId: req.user.id,  // Only owner's orders
    },
  });

  if (!order) {
    return res.status(404).json({ error: 'Order not found' });
  }

  res.json(order);
});
```

### Role-Based Access Control

```typescript
// middleware/authorize.ts
type Role = 'user' | 'admin' | 'moderator';

function authorize(...allowedRoles: Role[]) {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
}

// Usage
app.delete('/api/users/:id', authenticate, authorize('admin'), deleteUser);
```

### Permission-Based Access Control

```typescript
type Permission = 'read:users' | 'write:users' | 'delete:users' | 'read:posts';

const rolePermissions: Record<Role, Permission[]> = {
  user: ['read:posts'],
  moderator: ['read:posts', 'read:users'],
  admin: ['read:posts', 'read:users', 'write:users', 'delete:users'],
};

function hasPermission(role: Role, permission: Permission): boolean {
  return rolePermissions[role]?.includes(permission) ?? false;
}

function requirePermission(permission: Permission) {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!hasPermission(req.user.role, permission)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}
```

## Auth Audit Report Template

```markdown
# Authentication Audit Report

## Summary
Overall assessment of the authentication system.

## Password Security
| Check | Status | Notes |
|-------|--------|-------|
| Bcrypt hashing | ✅ Pass | Using cost factor 12 |
| Password requirements | ⚠️ Weak | Minimum 6 chars, no complexity |
| Breach checking | ❌ Missing | Should check Have I Been Pwned |

## Token Security
| Check | Status | Notes |
|-------|--------|-------|
| JWT algorithm | ✅ Pass | Using HS256 |
| Access token expiry | ✅ Pass | 15 minutes |
| Refresh token rotation | ❌ Missing | Should rotate on use |

## Session Security
| Check | Status | Notes |
|-------|--------|-------|
| HttpOnly cookies | ✅ Pass | |
| Secure flag | ⚠️ Dev only | Not set in development |
| SameSite | ✅ Pass | Strict |

## Authorization
| Check | Status | Notes |
|-------|--------|-------|
| Ownership checks | ⚠️ Partial | Missing on 3 endpoints |
| Role enforcement | ✅ Pass | |
| Admin protection | ✅ Pass | |

## Critical Issues
1. **Password Reset Token Predictable** - Using timestamp-based tokens
2. **Missing Rate Limiting** - No brute force protection on login

## Recommendations
1. Implement rate limiting on auth endpoints
2. Add password complexity requirements
3. Implement refresh token rotation
```

## Quality Checklist

- [ ] Password hashing is strong
- [ ] Tokens are properly managed
- [ ] Sessions are secure
- [ ] Access control enforced everywhere
- [ ] No sensitive data in tokens
- [ ] Logout properly invalidates session
- [ ] Password reset is secure
- [ ] Rate limiting in place
