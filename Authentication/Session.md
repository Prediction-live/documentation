## What is a Session?
A session is an opaque, server-side record stored in the database that enables long-lived authentication without exposing sensitive tokens. It's a refresh token mechanism.

Prediction.Live uses the classical two tokens system, one for access, `sb-access` cookie, the other one for refresh, `sb-refresh` cookie.

Supabase ID tokens (the ones being used to authenticate throughout the app) are minted using the ID token from Privy. 

### Two-Token System
1. Access token (`sb-access` cookie)
	- Short-lived Supabase JWT (15 minutes)
	- Used for API requests and RLS
	- Contains user identity claims
2. Refresh token (`sb-session` cookie)
	- Opaque session ID (random 32-byte hex string)
	- Long-lived (7 days)
	- Stored in the database, not in the token itself
### How It Works
```
User Signs In
    ↓
1. Privy ID token verified → User upserted
    ↓
2. Supabase JWT minted (15 min TTL) → Set as `sb-access` cookie
    ↓
3. Session created in DB → Opaque ID set as `sb-session` cookie
    ↓
User is authenticated
```
When the access token expires:
```
Access Token Expires (after 15 min)
    ↓
Client calls /api/auth/refresh
    ↓
1. Validates `sb-session` cookie against DB
    ↓
2. Rotates session (deletes old, creates new)
    ↓
3. Mints new Supabase JWT (15 min)
    ↓
4. Sets new cookies
    ↓
User stays authenticated seamlessly
```

## Why Sessions Are Important

### 1. Security
- Opaque tokens: The session ID reveals nothing about the user
- Server-controlled: Sessions can be revoked instantly (logout, breach)
- HttpOnly cookies: Not accessible to JavaScript, reducing XSS risk
- Short-lived access tokens: Limits exposure if compromised
### 2. User experience
- Long-lived sessions: Users stay logged in for 7 days
- Automatic refresh: No re-login needed when the access token expires
- Seamless experience: Works across browser tabs/devices
### 3. Security features
```ts
// Session rotation on refresh (prevents replay attacks)
rotateSession(oldSessionId, userId, fingerprint)

// Logout invalidates all sessions
invalidateUserSessions(userId)  // Logs out from all devices

// Fingerprinting (optional) tracks device/browser
fingerprint: request.headers.get('user-agent')
```

### 4. Compliance with security rules
- No JWTs stored in the database (only opaque session IDs)
- Server-side only: All session operations use service role
- RLS enabled: Sessions table has policies for user access
### 5. Architecture benefits
- Separation of concerns: Access tokens for auth, sessions for refresh
- Scalable: Can add features like device tracking, suspicious activity detection
- Audit trail: Sessions table can track login history

## Current Implementation Details

From the code:
```ts
// Sessions live for 7 days
const SESSION_TTL_DAYS = 7

// Session ID is cryptographically random (32 bytes = 64 hex chars)
const sessionId = randomBytes(32).toString('hex')

// Sessions table structure:
// - id: TEXT (opaque session ID)
// - user_id: UUID (links to user)
// - expires_at: TIMESTAMPTZ
// - fingerprint: TEXT (optional, for device tracking)
```
## Summary

Sessions provide a secure, long-lived authentication mechanism that:
- Keeps users logged in without frequent re-authentication
- Allows instant revocation of access (logout)
- Maintains security with short-lived access tokens
- Provides a foundation for features like device management and security monitoring

Without sessions, users would need to re-authenticate every 15 minutes, which is not practical for a real-time platform.