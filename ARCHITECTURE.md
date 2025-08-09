# Architecture Documentation - RealmKit SaaS Starter

## 🏗️ System Architecture Overview

This document provides a comprehensive view of the system architecture, design patterns, and technical decisions that make this SaaS starter robust and maintainable.

---

## 📋 Architecture Principles

### 1. **Separation of Concerns**
- **Presentation Layer**: UI components and pages
- **API Layer**: HTTP endpoints and request handling
- **Business Logic Layer**: Core application logic and rules
- **Data Access Layer**: Database interactions and queries

### 2. **Type Safety First**
- TypeScript throughout the entire stack
- Strict type checking enabled
- Runtime validation with Zod schemas
- Database types generated from Prisma schema

### 3. **AI-Optimized Design**
- Clear, predictable patterns
- Consistent naming conventions  
- Self-documenting code structure
- Explicit imports and dependencies

### 4. **Scalability Considerations**
- Stateless application design
- Database connection pooling
- Efficient caching strategies
- Horizontal scaling capabilities

---

## 🏛️ System Design Pattern: Layered Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                         │
├─────────────────────────────────────────────────────────────────┤
│  Next.js App Router  │  React Components  │  Client State      │
│  • Pages/Layouts     │  • UI Components   │  • Zustand Store   │
│  • Server Components│  • Form Handlers   │  • React Query     │
│  • Route Groups      │  • Event Handlers  │  • Local State     │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API LAYER                               │
├─────────────────────────────────────────────────────────────────┤
│       Next.js API Routes        │      External APIs           │
│  • Authentication endpoints     │  • Stripe API                │
│  • CRUD operations             │  • Email services            │
│  • Webhook handlers            │  • OAuth providers           │
│  • File upload endpoints       │  • Third-party services      │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                   BUSINESS LOGIC LAYER                         │
├─────────────────────────────────────────────────────────────────┤
│    Service Classes        │     Utilities & Helpers            │
│  • User Management        │  • Validation schemas              │
│  • Billing Operations     │  • Email templates                 │
│  • Content Management     │  • File processing                 │
│  • Admin Functions        │  • Security utilities              │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                   DATA ACCESS LAYER                            │
├─────────────────────────────────────────────────────────────────┤
│      Prisma ORM          │        External Storage             │
│  • Database Models       │  • File uploads (S3)               │
│  • Query optimization    │  • Cache layer (Redis)             │
│  • Transaction handling  │  • Search engine (if needed)       │
│  • Migration management  │  • CDN integration                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Directory Structure Deep Dive

### App Router Organization (Next.js 14)
```
app/
├── globals.css                   # Global styles and Tailwind
├── layout.tsx                    # Root layout with providers
├── page.tsx                      # Homepage
├── loading.tsx                   # Global loading UI
├── error.tsx                     # Global error boundary
├── not-found.tsx                 # 404 page
│
├── (auth)/                       # Authentication route group
│   ├── layout.tsx                # Auth-specific layout
│   ├── login/
│   │   ├── page.tsx              # Login page
│   │   └── loading.tsx           # Login loading state
│   ├── register/
│   │   ├── page.tsx              # Registration page
│   │   └── actions.ts            # Server actions for forms
│   ├── forgot-password/
│   │   └── page.tsx              # Password reset
│   └── reset-password/
│       └── page.tsx              # Password reset form
│
├── (dashboard)/                  # Protected dashboard routes
│   ├── layout.tsx                # Dashboard layout with sidebar
│   ├── middleware.ts             # Route-level auth middleware
│   ├── dashboard/
│   │   ├── page.tsx              # Main dashboard
│   │   ├── loading.tsx           # Dashboard loading
│   │   └── components/           # Dashboard-specific components
│   ├── settings/
│   │   ├── page.tsx              # User settings
│   │   ├── profile/page.tsx      # Profile management
│   │   ├── billing/page.tsx      # Billing settings
│   │   └── security/page.tsx     # Security settings
│   └── projects/
│       ├── page.tsx              # Projects list
│       ├── [id]/
│       │   ├── page.tsx          # Project details
│       │   └── edit/page.tsx     # Edit project
│       └── new/page.tsx          # Create new project
│
├── admin/                        # Admin-only routes
│   ├── layout.tsx                # Admin layout
│   ├── page.tsx                  # Admin dashboard
│   ├── users/
│   │   ├── page.tsx              # User management
│   │   └── [id]/page.tsx         # User details
│   ├── analytics/
│   │   └── page.tsx              # System analytics
│   └── settings/
│       └── page.tsx              # System settings
│
└── api/                          # API routes
    ├── auth/
    │   ├── [...nextauth]/route.ts # NextAuth.js configuration
    │   ├── register/route.ts      # User registration
    │   └── reset-password/route.ts# Password reset
    ├── users/
    │   ├── route.ts               # Users CRUD
    │   ├── [id]/route.ts          # Individual user operations
    │   └── profile/route.ts       # Current user profile
    ├── billing/
    │   ├── route.ts               # Billing operations
    │   ├── checkout/route.ts      # Stripe checkout
    │   └── webhook/route.ts       # Stripe webhooks
    ├── projects/
    │   ├── route.ts               # Projects CRUD
    │   └── [id]/route.ts          # Individual project operations
    ├── upload/
    │   └── route.ts               # File upload handling
    └── health/
        └── route.ts               # Health check endpoint
```

### Component Organization Strategy
```
components/
├── ui/                           # Base UI components (shadcn/ui)
│   ├── button.tsx                # Button component
│   ├── input.tsx                 # Input component
│   ├── dialog.tsx                # Modal/dialog
│   ├── toast.tsx                 # Notification toasts
│   ├── data-table.tsx            # Reusable data table
│   └── ...                       # Other base components
│
├── forms/                        # Form components
│   ├── login-form.tsx            # Login form with validation
│   ├── register-form.tsx         # Registration form
│   ├── profile-form.tsx          # Profile update form
│   ├── billing-form.tsx          # Billing/payment forms
│   └── project-form.tsx          # Project creation/editing
│
├── layout/                       # Layout components
│   ├── header.tsx                # Main header
│   ├── sidebar.tsx               # Dashboard sidebar
│   ├── footer.tsx                # Site footer
│   ├── breadcrumbs.tsx           # Navigation breadcrumbs
│   └── user-nav.tsx              # User navigation menu
│
├── charts/                       # Data visualization
│   ├── analytics-chart.tsx       # Analytics dashboard charts
│   ├── revenue-chart.tsx         # Revenue tracking
│   └── user-growth-chart.tsx     # User growth metrics
│
├── marketing/                    # Marketing/landing page components
│   ├── hero-section.tsx          # Hero section
│   ├── features-section.tsx      # Features showcase
│   ├── pricing-section.tsx       # Pricing plans
│   └── testimonials.tsx          # Customer testimonials
│
└── providers/                    # Context providers
    ├── auth-provider.tsx         # Authentication context
    ├── theme-provider.tsx        # Theme/dark mode
    ├── query-provider.tsx        # React Query setup
    └── toast-provider.tsx        # Toast notifications
```

---

## 🔧 Core Services Architecture

### Service Layer Pattern
All business logic is encapsulated in service classes that are:
- **Stateless**: No instance variables, pure functions
- **Testable**: Easy to unit test with mocked dependencies
- **Reusable**: Can be called from API routes, server actions, or background jobs
- **Type-safe**: Full TypeScript coverage with proper error handling

### User Management Service
```typescript
// services/user.service.ts
export class UserService {
  /**
   * Creates a new user with encrypted password
   * Handles validation, uniqueness check, and database transaction
   */
  static async create(data: CreateUserRequest): Promise<User> {
    // Input validation
    const validated = CreateUserSchema.parse(data);
    
    // Business logic
    const existingUser = await db.user.findUnique({
      where: { email: validated.email }
    });
    
    if (existingUser) {
      throw new BusinessError('User already exists', 'DUPLICATE_EMAIL');
    }
    
    // Data transformation
    const hashedPassword = await hash(validated.password, 12);
    
    // Database transaction
    return await db.$transaction(async (tx) => {
      const user = await tx.user.create({
        data: {
          email: validated.email,
          password: hashedPassword,
          name: validated.name,
        }
      });
      
      // Create default subscription
      await tx.subscription.create({
        data: {
          userId: user.id,
          plan: 'FREE',
          status: 'ACTIVE',
        }
      });
      
      return {
        id: user.id,
        email: user.email,
        name: user.name,
        createdAt: user.createdAt,
      };
    });
  }
}
```

### Billing Service Architecture
```typescript
// services/billing.service.ts
export class BillingService {
  /**
   * Creates Stripe checkout session for subscription upgrade
   */
  static async createCheckoutSession({
    userId,
    priceId,
    successUrl,
    cancelUrl,
  }: CreateCheckoutSessionRequest): Promise<{ url: string }> {
    // Get user for Stripe customer
    const user = await UserService.getById(userId);
    if (!user) {
      throw new BusinessError('User not found', 'USER_NOT_FOUND');
    }
    
    // Create or get Stripe customer
    let customerId = user.stripeCustomerId;
    if (!customerId) {
      const customer = await stripe.customers.create({
        email: user.email,
        name: user.name,
        metadata: { userId },
      });
      
      customerId = customer.id;
      
      // Update user with customer ID
      await UserService.updateStripeCustomer(userId, customerId);
    }
    
    // Create checkout session
    const session = await stripe.checkout.sessions.create({
      customer: customerId,
      payment_method_types: ['card'],
      billing_address_collection: 'required',
      line_items: [{ price: priceId, quantity: 1 }],
      mode: 'subscription',
      success_url: successUrl,
      cancel_url: cancelUrl,
      metadata: { userId },
    });
    
    if (!session.url) {
      throw new BusinessError('Failed to create checkout session', 'STRIPE_ERROR');
    }
    
    return { url: session.url };
  }
}
```

---

## 🗄️ Database Architecture

### Schema Design Philosophy
- **Explicit relationships**: Clear foreign keys and constraints
- **Audit trails**: CreatedAt/updatedAt on all entities
- **Soft deletes**: Mark records as deleted rather than physical deletion
- **Indexing strategy**: Optimize for common query patterns
- **Data integrity**: Use database constraints and validations

### Core Entity Relationships
```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│    User     │    │ Subscription │    │   Invoice   │
├─────────────┤    ├──────────────┤    ├─────────────┤
│ id (PK)     │◄──►│ id (PK)      │◄──►│ id (PK)     │
│ email       │    │ userId (FK)  │    │ userId (FK) │
│ name        │    │ plan         │    │ amount      │
│ password    │    │ status       │    │ status      │
│ role        │    │ currentPeriod│    │ stripeId    │
│ stripeId    │    │ stripeId     │    │ createdAt   │
│ createdAt   │    │ createdAt    │    └─────────────┘
│ updatedAt   │    │ updatedAt    │
│ deletedAt   │    └──────────────┘
└─────────────┘
       │
       ▼
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   Project   │    │   Session    │    │   Account   │
├─────────────┤    ├──────────────┤    ├─────────────┤
│ id (PK)     │    │ id (PK)      │    │ id (PK)     │
│ userId (FK) │    │ userId (FK)  │    │ userId (FK) │
│ title       │    │ token        │    │ provider    │
│ description │    │ expiresAt    │    │ providerID  │
│ status      │    │ createdAt    │    │ accessToken │
│ settings    │    └──────────────┘    │ refreshToken│
│ createdAt   │                       │ createdAt   │
│ updatedAt   │                       └─────────────┘
│ deletedAt   │
└─────────────┘
```

### Prisma Schema Example
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id               String    @id @default(cuid())
  email            String    @unique
  name             String?
  password         String
  role             UserRole  @default(USER)
  emailVerified    DateTime?
  image            String?
  stripeCustomerId String?   @unique
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt
  deletedAt        DateTime?

  // Relationships
  accounts      Account[]
  sessions      Session[]
  subscriptions Subscription[]
  projects      Project[]
  invoices      Invoice[]

  // Indexes for performance
  @@index([email])
  @@index([stripeCustomerId])
  @@index([createdAt])
  @@map("users")
}

model Subscription {
  id                   String            @id @default(cuid())
  userId               String
  plan                 SubscriptionPlan  @default(FREE)
  status               SubscriptionStatus @default(ACTIVE)
  currentPeriodStart   DateTime?
  currentPeriodEnd     DateTime?
  cancelAtPeriodEnd    Boolean           @default(false)
  stripeSubscriptionId String?           @unique
  stripePriceId        String?
  createdAt            DateTime          @default(now())
  updatedAt            DateTime          @updatedAt

  // Relationships
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Indexes
  @@index([userId])
  @@index([status])
  @@index([stripeSubscriptionId])
  @@map("subscriptions")
}

enum UserRole {
  USER
  ADMIN
  SUPER_ADMIN
}

enum SubscriptionPlan {
  FREE
  PRO
  ENTERPRISE
}

enum SubscriptionStatus {
  ACTIVE
  CANCELED
  PAST_DUE
  PAUSED
}
```

---

## 🔐 Authentication & Authorization Architecture

### NextAuth.js Configuration
```typescript
// lib/auth.ts
import { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import GoogleProvider from 'next-auth/providers/google';
import GitHubProvider from 'next-auth/providers/github';
import { PrismaAdapter } from '@next-auth/prisma-adapter';

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(db),
  session: { strategy: 'jwt' },
  
  providers: [
    // Email/password authentication
    CredentialsProvider({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) return null;
        
        const user = await UserService.authenticate(
          credentials.email,
          credentials.password
        );
        
        return user ? {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        } : null;
      },
    }),
    
    // OAuth providers
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    
    GitHubProvider({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    }),
  ],
  
  pages: {
    signIn: '/login',
    signUp: '/register',
    error: '/auth/error',
  },
  
  callbacks: {
    async jwt({ token, user, account }) {
      // Store user info in JWT
      if (user) {
        token.role = user.role;
      }
      
      // Handle OAuth account linking
      if (account?.provider && account?.providerAccountId) {
        token.provider = account.provider;
      }
      
      return token;
    },
    
    async session({ session, token }) {
      // Send properties to client
      if (session.user && token.sub) {
        session.user.id = token.sub;
        session.user.role = token.role as string;
      }
      
      return session;
    },
  },
  
  events: {
    async signIn({ user, account, isNewUser }) {
      // Track sign-in events
      console.log(`User ${user.email} signed in via ${account?.provider}`);
      
      // Create default subscription for new users
      if (isNewUser) {
        await UserService.createDefaultSubscription(user.id);
      }
    },
  },
};
```

### Role-Based Access Control
```typescript
// lib/rbac.ts
export enum Permission {
  READ_USERS = 'read:users',
  WRITE_USERS = 'write:users',
  DELETE_USERS = 'delete:users',
  READ_ANALYTICS = 'read:analytics',
  WRITE_SETTINGS = 'write:settings',
}

export const rolePermissions = {
  [UserRole.USER]: [],
  [UserRole.ADMIN]: [
    Permission.READ_USERS,
    Permission.WRITE_USERS,
    Permission.READ_ANALYTICS,
  ],
  [UserRole.SUPER_ADMIN]: Object.values(Permission),
};

export function hasPermission(
  userRole: UserRole,
  permission: Permission
): boolean {
  return rolePermissions[userRole].includes(permission);
}

export function requirePermission(permission: Permission) {
  return async function(req: NextRequest) {
    const session = await auth();
    
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }
    
    if (!hasPermission(session.user.role as UserRole, permission)) {
      return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
    }
    
    return null; // Permission granted
  };
}
```

---

## 🎨 Frontend Architecture

### Component Design System
- **Base Components**: shadcn/ui components for consistency
- **Compound Components**: Complex UI patterns
- **Smart Components**: Components with business logic
- **Dumb Components**: Pure presentation components

### State Management Strategy
```typescript
// lib/stores/auth.store.ts - Zustand for client-side state
interface AuthState {
  user: User | null;
  isLoading: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  updateProfile: (data: ProfileData) => Promise<void>;
}

export const useAuthStore = create<AuthState>((set, get) => ({
  user: null,
  isLoading: false,
  
  login: async (credentials) => {
    set({ isLoading: true });
    try {
      const result = await signIn('credentials', {
        ...credentials,
        redirect: false,
      });
      
      if (result?.error) {
        throw new Error(result.error);
      }
      
      // Refresh session
      const session = await getSession();
      set({ user: session?.user ?? null });
    } finally {
      set({ isLoading: false });
    }
  },
  
  logout: async () => {
    await signOut({ redirect: false });
    set({ user: null });
  },
  
  updateProfile: async (data) => {
    const response = await fetch('/api/users/profile', {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    
    if (!response.ok) {
      throw new Error('Failed to update profile');
    }
    
    const updatedUser = await response.json();
    set({ user: updatedUser });
  },
}));
```

### Form Handling Pattern
```typescript
// components/forms/profile-form.tsx
interface ProfileFormProps {
  user: User;
  onSuccess?: (user: User) => void;
}

export function ProfileForm({ user, onSuccess }: ProfileFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm<ProfileFormData>({
    resolver: zodResolver(ProfileFormSchema),
    defaultValues: {
      name: user.name ?? '',
      email: user.email,
    },
  });

  const onSubmit = async (data: ProfileFormData) => {
    try {
      const response = await fetch('/api/users/profile', {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const error = await response.json();
        if (error.field) {
          setError(error.field, { message: error.message });
          return;
        }
        throw new Error(error.message);
      }

      const updatedUser = await response.json();
      onSuccess?.(updatedUser);
      toast.success('Profile updated successfully');
    } catch (error) {
      toast.error('Failed to update profile');
      console.error('Profile update error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          {...register('name')}
          disabled={isSubmitting}
        />
        {errors.name && (
          <p className="text-red-500 text-sm mt-1">
            {errors.name.message}
          </p>
        )}
      </div>

      <div>
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          {...register('email')}
          disabled={isSubmitting}
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">
            {errors.email.message}
          </p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save Changes'}
      </Button>
    </form>
  );
}
```

---

## 💳 Payment Architecture (Stripe Integration)

### Subscription Flow Architecture
```
┌─ User ─────────────────┐    ┌─ Application ──────────┐    ┌─ Stripe ─────────────┐
│                        │    │                        │    │                      │
│ 1. Selects plan        │───►│ 2. Creates checkout    │───►│ 3. Processes payment │
│                        │    │    session             │    │                      │
│ 6. Accesses premium    │◄───│ 5. Activates features  │◄───│ 4. Sends webhook     │
│    features            │    │                        │    │                      │
└────────────────────────┘    └────────────────────────┘    └──────────────────────┘
```

### Webhook Handling Architecture
```typescript
// app/api/billing/webhook/route.ts
import { stripe } from '@/lib/stripe';
import { BillingService } from '@/services/billing.service';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  let event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (error) {
    console.error('Webhook signature verification failed:', error);
    return new Response('Webhook Error', { status: 400 });
  }

  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await BillingService.handleCheckoutCompleted(event.data.object);
        break;
        
      case 'invoice.payment_succeeded':
        await BillingService.handlePaymentSucceeded(event.data.object);
        break;
        
      case 'customer.subscription.updated':
        await BillingService.handleSubscriptionUpdated(event.data.object);
        break;
        
      case 'customer.subscription.deleted':
        await BillingService.handleSubscriptionCanceled(event.data.object);
        break;
        
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return new Response('Webhook handled', { status: 200 });
  } catch (error) {
    console.error('Webhook handler error:', error);
    return new Response('Webhook Error', { status: 500 });
  }
}
```

---

## 🚀 Performance Optimization Strategies

### 1. **Next.js Optimizations**
- **App Router**: Efficient routing with layouts
- **Server Components**: Reduce client-side JavaScript
- **Static Generation**: Pre-render when possible
- **Edge Runtime**: Fast execution at edge locations

### 2. **Database Optimizations**
- **Connection Pooling**: Reuse database connections
- **Query Optimization**: Efficient queries with proper indexes
- **Caching Strategy**: Redis for frequently accessed data
- **Read Replicas**: Separate read/write operations

### 3. **Caching Strategy**
```typescript
// lib/cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

export class CacheService {
  private static getKey(prefix: string, identifier: string): string {
    return `${prefix}:${identifier}`;
  }

  static async get<T>(
    prefix: string,
    identifier: string
  ): Promise<T | null> {
    const key = this.getKey(prefix, identifier);
    const cached = await redis.get(key);
    return cached as T | null;
  }

  static async set<T>(
    prefix: string,
    identifier: string,
    data: T,
    ttlSeconds: number = 3600
  ): Promise<void> {
    const key = this.getKey(prefix, identifier);
    await redis.setex(key, ttlSeconds, JSON.stringify(data));
  }

  static async invalidate(
    prefix: string,
    identifier: string
  ): Promise<void> {
    const key = this.getKey(prefix, identifier);
    await redis.del(key);
  }
}

// Usage in services
export class UserService {
  static async getById(id: string): Promise<User | null> {
    // Try cache first
    const cached = await CacheService.get<User>('user', id);
    if (cached) return cached;

    // Fetch from database
    const user = await db.user.findUnique({
      where: { id },
      select: { password: false }, // Exclude sensitive data
    });

    if (user) {
      // Cache for 1 hour
      await CacheService.set('user', id, user, 3600);
    }

    return user;
  }
}
```

### 4. **Image and Asset Optimization**
- **Next.js Image**: Automatic optimization and lazy loading
- **WebP Format**: Modern image formats
- **CDN Integration**: Serve assets from edge locations
- **Compression**: Gzip/Brotli compression

---

## 🔒 Security Architecture

### 1. **Input Validation & Sanitization**
```typescript
// lib/validation.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z
    .string()
    .email('Invalid email format')
    .max(255, 'Email too long')
    .transform(email => email.toLowerCase().trim()),
    
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .max(128, 'Password too long')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain special character'),
    
  name: z
    .string()
    .min(1, 'Name is required')
    .max(100, 'Name too long')
    .transform(name => name.trim()),
});
```

### 2. **Rate Limiting**
```typescript
// lib/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

// Different rate limits for different operations
export const loginRateLimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(5, '15 m'), // 5 attempts per 15 minutes
  analytics: true,
});

export const apiRateLimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(100, '1 h'), // 100 requests per hour
  analytics: true,
});

export async function checkRateLimit(
  limiter: Ratelimit,
  identifier: string
): Promise<boolean> {
  const { success } = await limiter.limit(identifier);
  return success;
}
```

### 3. **CSRF Protection**
- **NextAuth.js**: Built-in CSRF protection
- **SameSite Cookies**: Prevent cross-site request forgery
- **Origin Validation**: Verify request origins
- **State Parameters**: OAuth state validation

### 4. **Data Privacy & Compliance**
- **Password Hashing**: bcrypt with salt rounds
- **Data Encryption**: Encrypt sensitive data at rest
- **PII Handling**: Proper handling of personal information
- **GDPR Compliance**: Data export and deletion capabilities

---

## 📊 Monitoring & Observability

### Application Monitoring
```typescript
// lib/monitoring.ts
import * as Sentry from '@sentry/nextjs';

// Error tracking setup
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
});

// Custom metrics
export class MetricsService {
  static trackUserRegistration(userId: string) {
    Sentry.addBreadcrumb({
      message: 'User registered',
      category: 'auth',
      level: 'info',
      data: { userId },
    });
  }

  static trackSubscriptionChange(
    userId: string,
    from: string,
    to: string
  ) {
    Sentry.addBreadcrumb({
      message: 'Subscription changed',
      category: 'billing',
      level: 'info',
      data: { userId, from, to },
    });
  }
}
```

### Health Checks
```typescript
// app/api/health/route.ts
export async function GET() {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      stripe: await checkStripe(),
    },
  };

  const isHealthy = Object.values(health.checks).every(check => 
    check.status === 'healthy'
  );

  return Response.json(health, {
    status: isHealthy ? 200 : 503,
  });
}
```

---

This architecture documentation provides a comprehensive view of how all components work together to create a robust, scalable, and maintainable SaaS application. The design prioritizes developer experience, security, performance, and AI-assisted development while maintaining production readiness.