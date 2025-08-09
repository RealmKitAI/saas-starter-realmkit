# Authentication Module - AI Context

## Authentication System Overview
This realm uses NextAuth.js v5 (Auth.js) for authentication with a service-layer architecture. The system supports multiple authentication methods and role-based access control.

## Authentication Methods
1. **Credentials Provider**: Email/password authentication
2. **Magic Links**: Passwordless email authentication
3. **OAuth Providers**: Google and GitHub (easily extensible)

## Key Files and Locations

### Configuration
- `src/lib/auth/config.ts` - Main NextAuth configuration
- `src/lib/auth/index.ts` - Auth exports and utilities
- `.env.local` - Authentication environment variables

### API Routes
- `src/app/api/auth/[...nextauth]/route.ts` - NextAuth API handler
- `src/app/api/auth/register/route.ts` - Custom registration endpoint

### Services
- `src/services/auth/auth.service.ts` - Authentication business logic
- `src/services/user/user.service.ts` - User management operations

### Pages
- `src/app/(auth)/login/page.tsx` - Login page
- `src/app/(auth)/register/page.tsx` - Registration page
- `src/app/(auth)/forgot-password/page.tsx` - Password reset

### Components
- `src/components/auth/LoginForm.tsx` - Login form component
- `src/components/auth/RegisterForm.tsx` - Registration form component
- `src/components/auth/OAuthButtons.tsx` - Social login buttons

## Common Authentication Tasks

### 1. Protecting Pages
```typescript
// Server Component protection
import { auth } from "@/lib/auth"
import { redirect } from "next/navigation"

export default async function ProtectedPage() {
  const session = await auth()
  if (!session) {
    redirect("/login")
  }
  
  return <div>Protected content for {session.user.email}</div>
}
```

### 2. Protecting API Routes
```typescript
// app/api/protected/route.ts
import { auth } from "@/lib/auth"
import { NextResponse } from "next/server"

export async function GET() {
  const session = await auth()
  
  if (!session) {
    return new NextResponse("Unauthorized", { status: 401 })
  }
  
  // Handle authenticated request
  return NextResponse.json({ user: session.user })
}
```

### 3. Client-Side Session Access
```typescript
// Client Component
'use client'
import { useSession } from "next-auth/react"

export function UserProfile() {
  const { data: session, status } = useSession()
  
  if (status === "loading") return <div>Loading...</div>
  if (status === "unauthenticated") return <div>Not logged in</div>
  
  return <div>Welcome {session?.user?.email}</div>
}
```

### 4. Adding a New OAuth Provider
1. Install the provider:
   ```bash
   npm install @auth/[provider]-adapter
   ```

2. Update `src/lib/auth/config.ts`:
   ```typescript
   import Twitter from "@auth/core/providers/twitter"
   
   export const authConfig = {
     providers: [
       // ... existing providers
       Twitter({
         clientId: process.env.TWITTER_CLIENT_ID!,
         clientSecret: process.env.TWITTER_CLIENT_SECRET!,
       })
     ]
   }
   ```

3. Add environment variables to `.env.local`

4. Update OAuth buttons in `src/components/auth/OAuthButtons.tsx`

### 5. Implementing Role-Based Access
```typescript
// Middleware for role checking
export async function hasRole(role: string) {
  const session = await auth()
  return session?.user?.role === role
}

// In a Server Component
const isAdmin = await hasRole('admin')
if (!isAdmin) {
  redirect('/dashboard')
}

// In an API route
if (!await hasRole('admin')) {
  return new NextResponse("Forbidden", { status: 403 })
}
```

### 6. Custom Registration Flow
The registration process uses a custom endpoint to create users with additional data:

```typescript
// services/auth/auth.service.ts
export const authService = {
  async register(data: RegisterInput) {
    // Validate input
    // Hash password
    // Create user with profile
    // Send welcome email
    // Return user
  }
}
```

## Security Best Practices
1. **CSRF Protection**: Automatically handled by NextAuth
2. **Session Security**: JWT tokens with proper expiration
3. **Password Security**: Bcrypt hashing with salt rounds
4. **Rate Limiting**: Implement on auth endpoints
5. **Secure Cookies**: httpOnly, secure, sameSite settings

## Database Schema
```prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  password      String?   // Null for OAuth users
  name          String?
  role          String    @default("user")
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  accounts      Account[]
  sessions      Session[]
}

model Account {
  // OAuth account connections
}

model Session {
  // Active user sessions
}
```

## Email Verification Flow
1. User registers → Verification email sent
2. User clicks link → Email marked as verified
3. Unverified users have limited access

## Password Reset Flow
1. User requests reset → Token generated and emailed
2. User clicks link → Taken to reset form
3. New password set → User can login

## Session Management
- Sessions expire after 30 days (configurable)
- Refresh tokens rotate on use
- Multiple device sessions supported
- Session revocation available

## Troubleshooting Common Issues

### "NEXTAUTH_URL is not set"
Set the environment variable:
```env
NEXTAUTH_URL=http://localhost:3000  # Development
NEXTAUTH_URL=https://yourdomain.com  # Production
```

### OAuth redirect issues
Ensure callback URLs are correctly configured in provider dashboards:
- Google: `https://yourdomain.com/api/auth/callback/google`
- GitHub: `https://yourdomain.com/api/auth/callback/github`

### Session not persisting
Check that NEXTAUTH_SECRET is set and consistent across deployments.

## Testing Authentication
```typescript
// Mock authenticated requests in tests
const mockSession = {
  user: {
    id: "123",
    email: "test@example.com",
    role: "user"
  }
}

// Use in tests
jest.mock("@/lib/auth", () => ({
  auth: jest.fn(() => Promise.resolve(mockSession))
}))
```