# TODO: Authentication Implementation

> **📌 Tracked in:** [GitHub Issue #1](../../issues/1)
> **Want to help?** Comment on the issue and submit a PR!

This document outlines the tasks needed to add authentication to ChatGPT apps built with this starter.

## Why Authentication?

For production ChatGPT apps that:
- Access user-specific data from external APIs
- Require API keys or tokens
- Need to identify users
- Have rate limiting or access controls

## Implementation Tasks

### 1. Choose Authentication Strategy

**Options:**
- [ ] OAuth 2.0 (for third-party services like GitHub, Google, etc.)
- [ ] API Key validation (for custom APIs)
- [ ] JWT tokens (for custom authentication)
- [ ] Session-based authentication

**Decision:** _[To be determined based on use case]_

### 2. Server-Side Authentication

**File:** `example-pizza-server/src/server.ts`

- [ ] Add authentication middleware
  ```typescript
  // Example: Validate API key
  function validateApiKey(req: IncomingMessage): boolean {
    const apiKey = req.headers['x-api-key'];
    return apiKey === process.env.API_KEY;
  }
  ```

- [ ] Protect MCP endpoints
  - [ ] Add auth check to SSE endpoint (`/mcp`)
  - [ ] Add auth check to message endpoint (`/mcp/messages`)
  - [ ] Return 401 Unauthorized for invalid auth

- [ ] Handle authentication errors gracefully
  ```typescript
  if (!isAuthenticated(req)) {
    res.writeHead(401, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Unauthorized' }));
    return;
  }
  ```

### 3. Environment Variables

**File:** `example-pizza-server/.env` (create this file)

- [ ] Create `.env` file for secrets
  ```
  API_KEY=your-api-key-here
  OAUTH_CLIENT_ID=your-client-id
  OAUTH_CLIENT_SECRET=your-client-secret
  ```

- [ ] Add `.env` to `.gitignore`
- [ ] Document required environment variables in README
- [ ] Create `.env.example` template

### 4. OAuth 2.0 Flow (if applicable)

- [ ] Install OAuth library
  ```bash
  cd example-pizza-server
  pnpm add oauth4webapi
  ```

- [ ] Implement OAuth endpoints
  - [ ] `/auth/login` - Initiate OAuth flow
  - [ ] `/auth/callback` - Handle OAuth callback
  - [ ] `/auth/logout` - Clear session

- [ ] Store OAuth tokens securely
  - [ ] Use session storage or database
  - [ ] Implement token refresh logic
  - [ ] Handle token expiration

### 5. User Session Management

- [ ] Choose session storage strategy
  - In-memory (development only)
  - Redis (recommended for production)
  - Database

- [ ] Implement session middleware
  ```typescript
  import session from 'express-session';

  app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
  }));
  ```

- [ ] Associate MCP sessions with user sessions

### 6. Rate Limiting

- [ ] Add rate limiting middleware
  ```bash
  pnpm add express-rate-limit
  ```

- [ ] Configure rate limits per user/API key
  ```typescript
  import rateLimit from 'express-rate-limit';

  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each user to 100 requests per windowMs
  });
  ```

- [ ] Apply rate limits to tool endpoints

### 7. Secure API Calls

If your app calls external APIs:

- [ ] Store API keys in environment variables
- [ ] Never expose API keys to client-side widget code
- [ ] Implement server-side API proxy
  ```typescript
  // Server makes API calls, widget receives results
  async function fetchUserData(userId: string) {
    const response = await fetch(`https://api.example.com/users/${userId}`, {
      headers: {
        'Authorization': `Bearer ${process.env.API_TOKEN}`
      }
    });
    return response.json();
  }
  ```

### 8. ChatGPT Integration

- [ ] Document authentication setup in README
- [ ] Add authentication requirements to tool descriptions
- [ ] Handle authentication errors gracefully in widgets
- [ ] Provide clear error messages to users

### 9. Testing

- [ ] Test authentication flow end-to-end
- [ ] Verify unauthorized access is blocked
- [ ] Test token refresh logic
- [ ] Test rate limiting
- [ ] Test error handling

### 10. Security Checklist

- [ ] API keys/secrets not committed to git
- [ ] HTTPS required in production
- [ ] CORS configured correctly
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (if using database)
- [ ] XSS prevention in widget rendering
- [ ] CSRF protection for state-changing operations

## Example: GitHub OAuth

Here's a minimal OAuth implementation example:

```typescript
// example-pizza-server/src/auth.ts
import { generateAuthURL, exchangeCodeForToken } from './oauth-helpers';

export async function handleGitHubAuth(req: IncomingMessage, res: ServerResponse) {
  const url = new URL(req.url!, `http://${req.headers.host}`);

  if (url.pathname === '/auth/github') {
    // Redirect to GitHub OAuth
    const authUrl = generateAuthURL({
      clientId: process.env.GITHUB_CLIENT_ID!,
      redirectUri: `${process.env.BASE_URL}/auth/github/callback`,
      scope: 'read:user repo',
    });

    res.writeHead(302, { Location: authUrl });
    res.end();
    return true;
  }

  if (url.pathname === '/auth/github/callback') {
    const code = url.searchParams.get('code');
    if (!code) {
      res.writeHead(400).end('Missing code');
      return true;
    }

    // Exchange code for token
    const token = await exchangeCodeForToken(code);

    // Store token in session
    // ... session logic ...

    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<html><body>Authentication successful! You can close this window.</body></html>');
    return true;
  }

  return false;
}
```

## Resources

- [OAuth 2.0 Guide](https://oauth.net/2/)
- [GitHub OAuth Documentation](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps)
- [JWT.io](https://jwt.io/)
- [OWASP Security Cheat Sheet](https://cheatsheetseries.owasp.org/)

## Notes

- Start simple: API key validation is often enough for internal tools
- Only add OAuth if you need user-specific data from third-party services
- Always test authentication thoroughly before deploying
- Document the authentication setup process for users

---

**Status:** TODO - Authentication not yet implemented
**Priority:** High for production apps
**Estimated Effort:** 4-8 hours depending on complexity
