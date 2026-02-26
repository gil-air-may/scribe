# ADR 0002: Use JWT for Authentication

**Date:** 2026-02-26
**Status:** Accepted
**Deciders:** Development Team
**Tags:** authentication, security, api, architecture

## Context

The application needs a robust authentication mechanism that:
- Supports both web and mobile clients
- Enables horizontal scaling without shared session storage
- Works with a microservices architecture (future consideration)
- Provides adequate security for user data
- Minimizes server-side state management

Current constraints:
- Team has experience with both session and token-based auth
- Infrastructure will be deployed on Kubernetes (stateless preferred)
- Mobile app development planned for Q2
- Budget constraints favor simpler infrastructure

## Decision

We will implement JWT (JSON Web Tokens) for authentication.

Tokens will:
- Be signed with HS256 algorithm
- Include userId and token type in payload
- Expire after 24 hours
- Be stored in httpOnly cookies for web clients
- Be sent in Authorization headers for mobile clients

## Rationale

### Technical Factors

1. **Statelessness**: No need for centralized session storage (Redis, database)
   - Reduces infrastructure complexity
   - Eliminates session store as single point of failure
   - Simplifies scaling decisions

2. **Cross-Platform Support**: Same token format works for web, mobile, and third-party integrations
   - Mobile apps can store tokens securely in device storage
   - Web clients can use httpOnly cookies
   - APIs can verify tokens without database lookups

3. **Microservices Ready**: When we split into microservices, JWT can be verified independently by each service
   - No shared session state required
   - Each service validates tokens with shared secret
   - Easier to implement API gateway pattern

4. **Standard & Well-Supported**: JWT is an industry standard (RFC 7519)
   - Mature libraries in all languages
   - Good tooling and debugging support
   - Security best practices well-documented

### Business Factors

- Lower operational costs (no Redis/session store needed initially)
- Faster time to market (simpler architecture)
- Future-proof for planned mobile app

## Consequences

### Positive

- **Scalability**: Horizontal scaling without session synchronization
- **Performance**: No database/cache lookup for every authenticated request
- **Flexibility**: Tokens can be validated offline, useful for background jobs
- **Simplicity**: Less infrastructure to manage and monitor
- **Mobile Ready**: Same auth mechanism works across all client types

### Negative

- **Token Revocation**: Cannot invalidate tokens before expiration
  - Mitigation: Short expiration times (24h)
  - Future: Implement token blacklist for critical revocations

- **Token Size**: JWTs are larger than session IDs
  - Impact: ~200 bytes vs ~32 bytes for session ID
  - Mitigation: Minimal payload, only essential claims

- **Secret Management**: JWT secret must be protected
  - Impact: Compromise means all tokens can be forged
  - Mitigation: Use strong secret, rotate periodically, consider asymmetric signing in future

- **Logout Complexity**: True "logout" requires additional mechanism
  - Mitigation: Clear token client-side, rely on expiration
  - Future: Implement token blacklist if needed

### Neutral

- Need to implement refresh token mechanism for longer sessions
- Must handle token expiration gracefully in client applications
- Requires careful CORS configuration for web clients

## Alternatives Considered

### Session-Based Authentication

**Pros:**
- Immediate revocation capability
- Smaller session identifiers
- Server has full control over sessions

**Cons:**
- Requires session store (Redis, database)
- Complex to scale (sticky sessions or shared store)
- Less suitable for mobile apps
- Additional infrastructure cost

**Why not chosen:** Scaling complexity and infrastructure requirements conflict with our stateless architecture goals.

### OAuth 2.0 Only (No Custom Auth)

**Pros:**
- Delegates authentication to providers (Google, GitHub)
- No password management required
- Users may prefer social login

**Cons:**
- Requires internet connectivity for auth
- Dependent on third-party services
- Some users prefer local accounts
- More complex initial setup

**Why not chosen:** Will implement as additional option, but need native auth as foundation. Planned for Phase 2.

### API Keys

**Pros:**
- Simple to implement
- Easy to revoke
- Good for service-to-service auth

**Cons:**
- No user context or expiration
- Less secure for user authentication
- Not suitable for interactive sessions

**Why not chosen:** Better suited for service accounts, not user authentication.

## Implementation Notes

### Token Structure
```json
{
  "userId": "uuid-here",
  "type": "access",
  "iat": 1709042700,
  "exp": 1709129100
}
```

### Security Considerations
- Store JWT_SECRET in environment variables only
- Use httpOnly cookies for web to prevent XSS
- Implement CSRF protection for cookie-based auth
- Always use HTTPS in production
- Consider rate limiting on auth endpoints

### Future Enhancements
1. Implement refresh tokens (30-day expiration)
2. Add token blacklist for forced logout
3. Consider asymmetric signing (RS256) for multi-service architecture
4. Add claims for roles/permissions
5. Implement token rotation on refresh

## Validation

Success will be measured by:
- Zero scaling issues related to authentication (6 month horizon)
- Successful mobile app integration in Q2
- Less than 0.1% auth-related errors in production
- No security incidents related to authentication

Review date: 2026-08-26 (6 months)

## Related Decisions

- [ADR 0001: Use TypeScript for Backend](0001-use-typescript.md) - Affects implementation libraries
- Future ADR: Refresh Token Strategy (planned)
- Future ADR: OAuth Provider Integration (planned)

## References

- [RFC 7519 - JWT Standard](https://tools.ietf.org/html/rfc7519)
- [JWT.io - Token Debugger](https://jwt.io/)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- Internal: [Session Discussion - 2026-02-20](../sessions/2026-02-20-10-15.md)

---
**Generated by Session Scribe**
