# Authentication Flow

**Category:** Security
**Last Updated:** 2026-02-26
**Status:** Active

## Overview

The authentication system uses JWT (JSON Web Tokens) to verify user identity across requests. It provides a stateless authentication mechanism that works across web and mobile platforms.

The flow consists of:
1. User login with credentials
2. Server validates and issues JWT
3. Client includes token in subsequent requests
4. Server validates token for protected resources

## Key Concepts

### JSON Web Token (JWT)
A compact, URL-safe token format containing JSON-encoded claims. Structure: `header.payload.signature`

**Header**: Algorithm and token type
**Payload**: User claims (userId, expiration, etc.)
**Signature**: Cryptographic signature ensuring integrity

### httpOnly Cookies
Browser cookies that cannot be accessed via JavaScript, providing XSS protection for token storage.

### Bearer Token
Authorization header format: `Authorization: Bearer <token>`

## Architecture

```
┌─────────────┐                 ┌─────────────┐                 ┌──────────────┐
│   Client    │                 │   Server    │                 │   Database   │
│  (Web/Mobile)│                │  (Express)  │                 │  (PostgreSQL)│
└──────┬──────┘                 └──────┬──────┘                 └──────┬───────┘
       │                               │                               │
       │  POST /auth/login             │                               │
       │  {email, password}            │                               │
       ├──────────────────────────────►│                               │
       │                               │  Query user by email          │
       │                               ├──────────────────────────────►│
       │                               │                               │
       │                               │  User record                  │
       │                               │◄──────────────────────────────┤
       │                               │                               │
       │                               │ Compare password hash         │
       │                               │ (bcrypt.compare)              │
       │                               │                               │
       │                               │ Generate JWT                  │
       │                               │ (jwt.sign)                    │
       │                               │                               │
       │  Set-Cookie: token=...        │                               │
       │  {user, token}                │                               │
       │◄──────────────────────────────┤                               │
       │                               │                               │
       │  GET /api/users/me            │                               │
       │  Authorization: Bearer token  │                               │
       ├──────────────────────────────►│                               │
       │                               │ Verify JWT                    │
       │                               │ (jwt.verify)                  │
       │                               │                               │
       │                               │  Query user data              │
       │                               ├──────────────────────────────►│
       │                               │                               │
       │                               │  User data                    │
       │                               │◄──────────────────────────────┤
       │                               │                               │
       │  {user}                       │                               │
       │◄──────────────────────────────┤                               │
       │                               │                               │
```

## Patterns & Practices

### Pattern 1: Protected Route
**When to use:** Any endpoint requiring authentication

**How to implement:**
```typescript
import { authMiddleware } from '../middleware/auth.middleware';

router.get('/users/me', authMiddleware, async (req, res) => {
  // req.user is populated by middleware
  const user = await getUserById(req.user.userId);
  res.json({ user });
});
```

### Pattern 2: Optional Authentication
**When to use:** Endpoints that personalize based on auth but don't require it

**How to implement:**
```typescript
import { optionalAuthMiddleware } from '../middleware/auth.middleware';

router.get('/posts', optionalAuthMiddleware, async (req, res) => {
  const posts = await getPosts();
  // req.user exists if authenticated
  if (req.user) {
    // Return personalized version
  }
  res.json({ posts });
});
```

### Pattern 3: Role-Based Access
**When to use:** Different user types with varying permissions

**How to implement:**
```typescript
const requireRole = (role: string) => {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
};

router.delete('/users/:id', authMiddleware, requireRole('admin'), handler);
```

## Code Examples

### Login Flow
```typescript
// auth.routes.ts
router.post('/login', async (req, res) => {
  const { email, password } = req.body;

  // Find user
  const user = await User.findOne({ where: { email } });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Verify password
  const isValid = await bcrypt.compare(password, user.passwordHash);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate token
  const token = authService.generateToken(user.id);

  // Set httpOnly cookie
  res.cookie('token', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  });

  res.json({ user: user.toJSON(), token });
});
```

### Token Validation
```typescript
// auth.middleware.ts
export const authMiddleware = async (req, res, next) => {
  try {
    // Try cookie first, then Authorization header
    const token = req.cookies.token ||
                  req.headers.authorization?.replace('Bearer ', '');

    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as TokenPayload;

    // Attach user to request
    req.user = decoded;
    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

## Gotchas & Warnings

- ⚠️ **Token Cannot Be Revoked**: Once issued, tokens are valid until expiration. For critical logout (e.g., security breach), implement a token blacklist.

- ⚠️ **Secret Exposure**: If JWT_SECRET is compromised, all tokens can be forged. Store securely, rotate periodically.

- ⚠️ **XSS Vulnerability**: If tokens are stored in localStorage, they're accessible to JavaScript. Always use httpOnly cookies for web clients.

- ⚠️ **CSRF with Cookies**: httpOnly cookies are sent automatically, making endpoints vulnerable to CSRF. Implement CSRF tokens for state-changing operations.

- ⚠️ **Token Size**: JWTs are sent with every request. Keep payload minimal to reduce bandwidth.

## Common Issues & Solutions

### Issue: "Invalid token" on valid requests
**Solution:** Check clock synchronization between servers. JWT expiration uses timestamps.
**Prevention:** Use NTP to sync server clocks. Add clock skew tolerance to verification.

### Issue: Token works in Postman but not in browser
**Solution:** Check CORS configuration. Credentials must be included in fetch requests.
**Prevention:**
```typescript
// Server
cors({ origin: 'https://app.example.com', credentials: true })

// Client
fetch('/api/endpoint', { credentials: 'include' })
```

### Issue: User appears logged in but API returns 401
**Solution:** Token expired. Check expiration time and implement refresh mechanism.
**Prevention:** Implement refresh tokens or extend expiration time.

## Dependencies

- `jsonwebtoken` (v9.0.0) - JWT generation and validation
- `bcrypt` (v5.1.1) - Password hashing
- `cookie-parser` (v1.4.6) - Parse cookies in Express

## Configuration

```typescript
// .env
JWT_SECRET=your-256-bit-secret-here
JWT_EXPIRATION=24h
NODE_ENV=production

// config/auth.ts
export const authConfig = {
  jwtSecret: process.env.JWT_SECRET!,
  jwtExpiration: process.env.JWT_EXPIRATION || '24h',
  bcryptRounds: 10,
  cookieOptions: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict' as const,
    maxAge: 24 * 60 * 60 * 1000
  }
};
```

## Testing

```typescript
// auth.service.spec.ts
describe('AuthService', () => {
  describe('generateToken', () => {
    it('should generate valid JWT', () => {
      const token = authService.generateToken('user-123');
      const decoded = jwt.verify(token, process.env.JWT_SECRET!);

      expect(decoded.userId).toBe('user-123');
      expect(decoded.type).toBe('access');
    });
  });

  describe('validateToken', () => {
    it('should reject expired tokens', () => {
      const expiredToken = jwt.sign(
        { userId: 'user-123' },
        process.env.JWT_SECRET!,
        { expiresIn: '-1h' }
      );

      expect(() => authService.validateToken(expiredToken))
        .toThrow('Token expired');
    });
  });
});
```

## Performance Considerations

- **Token Verification**: ~0.1ms per request (negligible)
- **Password Hashing**: ~100ms (intentionally slow for security)
  - Perform asynchronously to avoid blocking
  - Consider queuing for bulk operations
- **Database Lookups**: Avoided for token validation (main benefit of JWT)

## Security Considerations

- **HTTPS Only**: Always use HTTPS in production
- **Strong Secret**: Use cryptographically random 256-bit secret
- **Short Expiration**: Balance security vs. UX (24h recommended)
- **Rate Limiting**: Implement on `/auth/login` to prevent brute force
- **Password Policy**: Enforce minimum length, complexity requirements
- **Audit Logging**: Log all authentication attempts

## Related Files

- `src/services/auth.service.ts:1` - Core authentication logic
- `src/middleware/auth.middleware.ts:1` - Request authentication
- `src/routes/auth.routes.ts:1` - Login/logout endpoints
- `tests/auth.service.spec.ts:1` - Test coverage
- `config/auth.ts:1` - Configuration

## Related Documentation

- [ADR 0002: Use JWT for Authentication](../decisions/0002-use-jwt-auth.md)
- [Session: 2026-02-26-14-45 - Implementation](../sessions/2026-02-26-14-45.md)
- [External: JWT Best Practices](https://tools.ietf.org/html/rfc8725)

## History

- **2026-02-26** - Initial implementation with JWT
- **2026-02-26** - Added httpOnly cookie support for web clients

## Future Improvements

- [ ] Implement refresh token mechanism (30-day expiration)
- [ ] Add token blacklist for forced logout scenarios
- [ ] Consider RS256 (asymmetric) signing for microservices
- [ ] Add role/permission claims to token payload
- [ ] Implement OAuth providers (Google, GitHub)
- [ ] Add two-factor authentication option

---
**Generated by Session Scribe**
