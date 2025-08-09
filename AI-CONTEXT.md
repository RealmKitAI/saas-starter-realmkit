# AI Assistant Context - RealmKit SaaS Starter

## 🤖 For AI Coding Assistants

This document provides complete context for AI assistants (ChatGPT, Claude, Cursor, etc.) to understand and work with this SaaS starter effectively.

---

## 🏗️ Architecture Overview

### Core Principles
1. **Layered Architecture**: Clear separation between UI, API, business logic, and data
2. **Type Safety First**: TypeScript everywhere with strict mode
3. **Single Responsibility**: Each file/function has one clear purpose
4. **Explicit over Implicit**: Clear naming, explicit imports, documented decisions
5. **Test-Driven**: Every feature has comprehensive tests

### System Design Pattern: **Service Layer Architecture**

```
┌─ Presentation Layer ──────────────────────────────────┐
│ app/ - Next.js pages and components                   │
│ components/ - Reusable UI components                  │
└───────────────────────────────────────────────────────┘
                          │
┌─ API Layer ───────────────────────────────────────────┐
│ app/api/ - Next.js API routes (thin controllers)     │
└───────────────────────────────────────────────────────┘
                          │
┌─ Business Logic Layer ────────────────────────────────┐
│ services/ - Core business logic and rules            │
└───────────────────────────────────────────────────────┘
                          │
┌─ Data Access Layer ───────────────────────────────────┐
│ lib/db.ts - Database client and queries              │
│ prisma/ - Schema, migrations, seed data              │
└───────────────────────────────────────────────────────┘
```

---

## 📁 File Organization Rules

### 1. App Router Structure (Next.js 14)
```
app/
├── (auth)/                    # Route groups for auth pages
│   ├── login/page.tsx         # Public auth pages
│   └── register/page.tsx
├── (dashboard)/               # Protected dashboard routes  
│   ├── dashboard/page.tsx     # Main dashboard
│   ├── settings/page.tsx      # User settings
│   └── billing/page.tsx       # Billing management
├── admin/                     # Admin-only routes
├── api/                       # API endpoints
└── layout.tsx                 # Root layout
```

**Rule**: Use route groups `(auth)`, `(dashboard)` for logical organization without affecting URLs.

### 2. Component Organization
```
components/
├── ui/                        # shadcn/ui base components
│   ├── button.tsx
│   ├── input.tsx
│   └── ...
├── forms/                     # Form-specific components
│   ├── login-form.tsx
│   ├── register-form.tsx
│   └── billing-form.tsx
├── layout/                    # Layout components
│   ├── header.tsx
│   ├── sidebar.tsx
│   └── footer.tsx
└── charts/                    # Data visualization
    └── analytics-chart.tsx
```

**Rule**: Components are grouped by domain, not by type.

### 3. Service Layer Pattern
```
services/
├── user.service.ts            # User management business logic
├── billing.service.ts         # Payment and subscription logic
├── email.service.ts           # Email sending and templates
└── admin.service.ts           # Admin operations
```

**Rule**: All business logic goes in services. API routes are thin controllers.

---

## 🎯 Coding Patterns and Conventions

### 1. File Naming
- **Pages**: `kebab-case` (Next.js requirement)
- **Components**: `PascalCase.tsx`  
- **Services**: `camelCase.service.ts`
- **Utilities**: `camelCase.ts`
- **Types**: `PascalCase.types.ts`

### 2. Import Organization
```typescript
// 1. External libraries
import React from 'react';
import { NextRequest } from 'next/server';

// 2. Internal utilities and config
import { db } from '@/lib/db';
import { auth } from '@/lib/auth';

// 3. Services (business logic)
import { UserService } from '@/services/user.service';

// 4. Components
import { Button } from '@/components/ui/button';

// 5. Types
import type { User } from '@/types/user.types';
```

### 3. API Route Pattern
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { UserService } from '@/services/user.service';
import { auth } from '@/lib/auth';

// GET /api/users
export async function GET(request: NextRequest) {
  try {
    // 1. Authentication check
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // 2. Extract parameters
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') ?? '1');

    // 3. Call service layer
    const users = await UserService.getPaginated({ page });

    // 4. Return response
    return NextResponse.json({ users });
  } catch (error) {
    console.error('GET /api/users error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

**Pattern**: Auth → Validate → Service → Response

### 4. Service Layer Pattern
```typescript
// services/user.service.ts
import { db } from '@/lib/db';
import { hash } from 'bcryptjs';
import type { CreateUserRequest, User } from '@/types/user.types';

export class UserService {
  /**
   * Creates a new user with hashed password
   */
  static async create(data: CreateUserRequest): Promise<User> {
    // 1. Validate input
    if (!data.email || !data.password) {
      throw new Error('Email and password are required');
    }

    // 2. Check if user exists
    const existingUser = await db.user.findUnique({
      where: { email: data.email }
    });
    if (existingUser) {
      throw new Error('User already exists');
    }

    // 3. Hash password
    const hashedPassword = await hash(data.password, 12);

    // 4. Create user
    const user = await db.user.create({
      data: {
        email: data.email,
        password: hashedPassword,
        name: data.name,
      }
    });

    // 5. Return user (without password)
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }

  /**
   * Gets paginated users for admin
   */
  static async getPaginated({ 
    page = 1, 
    limit = 20 
  }: { 
    page?: number; 
    limit?: number; 
  }): Promise<{ users: User[]; total: number }> {
    const skip = (page - 1) * limit;

    const [users, total] = await Promise.all([
      db.user.findMany({
        skip,
        take: limit,
        select: {
          id: true,
          email: true,
          name: true,
          createdAt: true,
          // Never select password
        },
        orderBy: {
          createdAt: 'desc',
        },
      }),
      db.user.count(),
    ]);

    return { users, total };
  }
}
```

**Pattern**: Validate → Business Logic → Database → Return

### 5. Component Pattern (Server + Client)
```typescript
// app/(dashboard)/dashboard/page.tsx (Server Component)
import { auth } from '@/lib/auth';
import { UserService } from '@/services/user.service';
import { DashboardClient } from './dashboard-client';

export default async function DashboardPage() {
  // Server-side data fetching
  const session = await auth();
  if (!session?.user) {
    redirect('/login');
  }

  const userData = await UserService.getById(session.user.id);

  // Pass data to client component
  return <DashboardClient user={userData} />;
}
```

```typescript
// app/(dashboard)/dashboard/dashboard-client.tsx (Client Component)
'use client';

import { useState } from 'react';
import type { User } from '@/types/user.types';

interface DashboardClientProps {
  user: User;
}

export function DashboardClient({ user }: DashboardClientProps) {
  const [isLoading, setIsLoading] = useState(false);

  const handleUpdateProfile = async () => {
    setIsLoading(true);
    try {
      // API call to update user
      await fetch('/api/users/profile', {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: user.name }),
      });
    } catch (error) {
      console.error('Update failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="p-6">
      <h1>Welcome, {user.name}</h1>
      {/* Dashboard content */}
    </div>
  );
}
```

**Pattern**: Server Component (data) → Client Component (interactivity)

---

## 🗄️ Database Patterns

### 1. Prisma Schema Conventions
```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  role      UserRole @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relationships
  accounts Account[]
  sessions Session[]
  subscriptions Subscription[]

  @@map("users") // Table name in snake_case
}

enum UserRole {
  USER
  ADMIN
}
```

### 2. Database Query Patterns
```typescript
// Always use transactions for related operations
const result = await db.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email, password: hashedPassword },
  });

  await tx.subscription.create({
    data: {
      userId: user.id,
      plan: 'FREE',
    },
  });

  return user;
});
```

---

## 🧪 Testing Patterns

### 1. Unit Test Structure
```typescript
// services/__tests__/user.service.test.ts
import { UserService } from '../user.service';
import { db } from '@/lib/db';

// Mock database
jest.mock('@/lib/db', () => ({
  db: {
    user: {
      create: jest.fn(),
      findUnique: jest.fn(),
      findMany: jest.fn(),
      count: jest.fn(),
    },
  },
}));

const mockDb = db as jest.Mocked<typeof db>;

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('create', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      };

      mockDb.user.findUnique.mockResolvedValue(null); // No existing user
      mockDb.user.create.mockResolvedValue({
        id: '123',
        email: userData.email,
        name: userData.name,
        password: 'hashed_password',
        createdAt: new Date(),
      });

      // Act
      const result = await UserService.create(userData);

      // Assert
      expect(result.email).toBe(userData.email);
      expect(result.name).toBe(userData.name);
      expect(result).not.toHaveProperty('password'); // Sensitive data filtered
      expect(mockDb.user.create).toHaveBeenCalledWith({
        data: {
          email: userData.email,
          password: expect.any(String), // Hashed
          name: userData.name,
        },
      });
    });

    it('should throw error if user already exists', async () => {
      // Arrange
      mockDb.user.findUnique.mockResolvedValue({
        id: '123',
        email: 'existing@example.com',
      });

      // Act & Assert
      await expect(
        UserService.create({
          email: 'existing@example.com',
          password: 'password123',
        })
      ).rejects.toThrow('User already exists');
    });
  });
});
```

### 2. Integration Test Pattern
```typescript
// tests/integration/auth.test.ts
import { createMocks } from 'node-mocks-http';
import handler from '@/app/api/auth/register/route';

describe('/api/auth/register', () => {
  beforeEach(async () => {
    // Clean database before each test
    await db.user.deleteMany({});
  });

  it('should register new user', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: {
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      },
    });

    await handler(req, res);

    expect(res._getStatusCode()).toBe(201);
    
    const data = JSON.parse(res._getData());
    expect(data.user.email).toBe('test@example.com');
    expect(data.user).not.toHaveProperty('password');

    // Verify user was created in database
    const user = await db.user.findUnique({
      where: { email: 'test@example.com' },
    });
    expect(user).toBeTruthy();
  });
});
```

---

## 🔒 Security Patterns

### 1. Input Validation
```typescript
import { z } from 'zod';

// Define schemas for validation
export const CreateUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
});

// Use in API routes
export async function POST(request: NextRequest) {
  const body = await request.json();
  
  // Validate input
  const validatedData = CreateUserSchema.parse(body);
  
  // Continue with validated data
  const user = await UserService.create(validatedData);
}
```

### 2. Authentication Patterns
```typescript
// lib/auth.ts - NextAuth configuration
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { UserService } from '@/services/user.service';

export const { handlers, auth } = NextAuth({
  providers: [
    CredentialsProvider({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }

        const user = await UserService.authenticate(
          credentials.email,
          credentials.password
        );

        return user ? { 
          id: user.id, 
          email: user.email, 
          name: user.name 
        } : null;
      },
    }),
  ],
  pages: {
    signIn: '/login',
    signUp: '/register',
  },
  session: { strategy: 'jwt' },
});
```

---

## 🎨 UI/UX Patterns

### 1. Form Handling
```typescript
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { CreateUserSchema } from '@/lib/validations';

interface RegisterFormProps {
  onSuccess: (user: User) => void;
}

export function RegisterForm({ onSuccess }: RegisterFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    resolver: zodResolver(CreateUserSchema),
  });

  const onSubmit = async (data: CreateUserRequest) => {
    try {
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        throw new Error('Registration failed');
      }

      const result = await response.json();
      onSuccess(result.user);
    } catch (error) {
      console.error('Registration error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Input
          {...register('email')}
          type="email"
          placeholder="Email"
          disabled={isSubmitting}
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">
            {errors.email.message}
          </p>
        )}
      </div>

      <div>
        <Input
          {...register('password')}
          type="password"
          placeholder="Password"
          disabled={isSubmitting}
        />
        {errors.password && (
          <p className="text-red-500 text-sm mt-1">
            {errors.password.message}
          </p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting} className="w-full">
        {isSubmitting ? 'Creating account...' : 'Create account'}
      </Button>
    </form>
  );
}
```

### 2. Loading and Error States
```typescript
'use client';

import { useState, useEffect } from 'react';
import { Skeleton } from '@/components/ui/skeleton';
import { Alert, AlertDescription } from '@/components/ui/alert';

export function UsersList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error('Failed to fetch users');
        
        const data = await response.json();
        setUsers(data.users);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (loading) {
    return (
      <div className="space-y-4">
        {Array.from({ length: 3 }).map((_, i) => (
          <Skeleton key={i} className="h-20 w-full" />
        ))}
      </div>
    );
  }

  if (error) {
    return (
      <Alert variant="destructive">
        <AlertDescription>{error}</AlertDescription>
      </Alert>
    );
  }

  return (
    <div className="space-y-4">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

---

## 💳 Payment Integration Patterns

### 1. Stripe Setup
```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export const PLANS = {
  FREE: {
    name: 'Free',
    price: 0,
    features: ['5 projects', 'Basic support'],
  },
  PRO: {
    name: 'Pro',
    price: 2900, // $29.00 in cents
    priceId: 'price_1234567890',
    features: ['Unlimited projects', 'Priority support', 'Advanced features'],
  },
} as const;
```

### 2. Subscription Management
```typescript
// services/billing.service.ts
import { stripe } from '@/lib/stripe';

export class BillingService {
  static async createCheckoutSession({
    userId,
    priceId,
    successUrl,
    cancelUrl,
  }: {
    userId: string;
    priceId: string;
    successUrl: string;
    cancelUrl: string;
  }) {
    const session = await stripe.checkout.sessions.create({
      customer_email: user.email,
      payment_method_types: ['card'],
      billing_address_collection: 'required',
      line_items: [
        {
          price: priceId,
          quantity: 1,
        },
      ],
      mode: 'subscription',
      success_url: successUrl,
      cancel_url: cancelUrl,
      metadata: {
        userId,
      },
    });

    return { url: session.url };
  }

  static async handleWebhook(
    body: string,
    signature: string
  ): Promise<void> {
    const event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );

    switch (event.type) {
      case 'checkout.session.completed':
        await this.handleCheckoutCompleted(event.data.object);
        break;
      case 'invoice.payment_succeeded':
        await this.handlePaymentSucceeded(event.data.object);
        break;
      case 'customer.subscription.deleted':
        await this.handleSubscriptionDeleted(event.data.object);
        break;
    }
  }
}
```

---

## 🚀 Deployment Patterns

### 1. Environment Configuration
```bash
# .env.example
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/saas_starter"

# NextAuth.js
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-here"

# Stripe
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."
STRIPE_PUBLISHABLE_KEY="pk_test_..."

# Email (optional)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT="587"
SMTP_USER="your-email@gmail.com"
SMTP_PASS="your-app-password"

# Redis (optional - for caching)
REDIS_URL="redis://localhost:6379"
```

### 2. Docker Configuration
```dockerfile
# Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma

USER nextjs
EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

---

## 🎯 AI Assistant Guidelines

### When Working with This Codebase:

1. **Follow the Service Layer Pattern**
   - Never put business logic in API routes
   - Always call services from API routes
   - Keep services pure and testable

2. **Maintain Type Safety**
   - Use TypeScript strictly
   - Define interfaces for all data structures
   - Validate input with Zod schemas

3. **Follow File Organization**
   - Respect the established folder structure
   - Use consistent naming conventions
   - Group related functionality together

4. **Write Tests First**
   - Every new feature needs tests
   - Use the established testing patterns
   - Mock external dependencies consistently

5. **Security First**
   - Always validate input
   - Never expose sensitive data
   - Use proper authentication checks
   - Sanitize database queries

6. **Performance Considerations**
   - Use Server Components when possible
   - Implement proper caching strategies
   - Optimize database queries
   - Handle loading states properly

7. **Error Handling**
   - Use try-catch blocks consistently
   - Log errors appropriately
   - Return meaningful error messages
   - Handle edge cases gracefully

### Common Tasks and How to Approach Them:

**Adding a New API Endpoint:**
1. Create route file in `app/api/`
2. Add authentication check
3. Validate input with Zod
4. Call appropriate service method
5. Return structured response
6. Write integration test

**Adding a New Page:**
1. Create page file in appropriate app folder
2. Use Server Component for data fetching
3. Create Client Component for interactivity
4. Add proper TypeScript interfaces
5. Handle loading and error states
6. Write E2E test

**Adding a New Database Model:**
1. Update Prisma schema
2. Create and run migration
3. Update service layer methods
4. Add validation schemas
5. Update TypeScript types
6. Write unit tests

This codebase is designed to be AI-friendly. Every pattern is consistent, every decision is documented, and every feature follows the same architectural principles.