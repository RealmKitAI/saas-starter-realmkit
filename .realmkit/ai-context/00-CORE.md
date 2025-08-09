# Modern SaaS Starter - Core AI Context

## Realm Overview
This is a production-ready SaaS application built with Next.js 14 App Router, TypeScript, PostgreSQL, and Prisma ORM. It follows a service-layer architecture pattern with clear separation of concerns.

## Technology Stack
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS + shadcn/ui components
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: NextAuth.js v5 (Auth.js)
- **Payments**: Stripe subscriptions
- **Email**: Resend for transactional emails
- **Deployment**: Optimized for Vercel

## Architecture Philosophy
This realm follows these core principles:
1. **Service Layer Pattern**: All business logic lives in services, not in components or API routes
2. **Type Safety**: Strict TypeScript throughout with proper type definitions
3. **Server-First**: Use Server Components by default, Client Components only when needed
4. **Error Boundaries**: Consistent error handling at service and API layers
5. **Performance**: Optimized for Core Web Vitals with proper caching strategies

## Project Structure
```
src/
├── app/                      # Next.js App Router
│   ├── (auth)/              # Authentication pages (public)
│   │   ├── login/           # Login page
│   │   ├── register/        # Registration page
│   │   └── forgot-password/ # Password reset
│   ├── (dashboard)/         # Protected dashboard pages
│   │   └── dashboard/       # Dashboard routes
│   │       ├── page.tsx     # Dashboard home
│   │       ├── billing/     # Billing management
│   │       ├── settings/    # User settings
│   │       └── admin/       # Admin panel (role-based)
│   ├── (marketing)/         # Public marketing pages
│   │   ├── page.tsx         # Homepage
│   │   ├── pricing/         # Pricing page
│   │   └── about/          # About page
│   └── api/                 # API routes
│       ├── auth/           # Auth endpoints
│       ├── users/          # User management
│       ├── billing/        # Stripe operations
│       └── webhooks/       # External webhooks
├── components/              # React components
│   ├── ui/                 # shadcn/ui base components
│   ├── dashboard/          # Dashboard-specific components
│   ├── marketing/          # Marketing page components
│   └── shared/             # Shared components
├── lib/                    # Utilities and configurations
│   ├── auth/              # NextAuth configuration
│   ├── db/                # Database client singleton
│   ├── email/             # Email templates and client
│   └── utils/             # Helper functions
├── services/               # Business logic layer
│   ├── auth/              # Authentication service
│   ├── user/              # User management service
│   ├── billing/           # Stripe billing service
│   └── email/             # Email service
├── types/                  # TypeScript type definitions
└── config/                 # Application configuration
```

## Development Workflow

### Starting a New Feature
1. **Plan the data model**: Update `prisma/schema.prisma` if needed
2. **Create the service**: Add business logic to `src/services/`
3. **Add API routes**: Create endpoints in `src/app/api/`
4. **Build UI components**: Server Components in `app/`, Client Components in `components/`
5. **Add tests**: Unit tests for services, integration tests for APIs

### Common Commands
```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript compiler
npm test            # Run tests
npx prisma studio   # Open database GUI
```

## Key Conventions

### File Naming
- Components: PascalCase (e.g., `UserProfile.tsx`)
- Pages: lowercase with hyphens (e.g., `forgot-password/page.tsx`)
- Services: camelCase with .service suffix (e.g., `user.service.ts`)
- API routes: RESTful naming (e.g., `/api/users/[id]`)

### Component Patterns
```typescript
// Server Component (default)
export default async function UserList() {
  const users = await userService.getAll()
  return <div>{/* render users */}</div>
}

// Client Component (when needed)
'use client'
export function InteractiveForm() {
  const [state, setState] = useState()
  return <form>{/* interactive elements */}</form>
}
```

### Service Layer Pattern
```typescript
// services/user/user.service.ts
export const userService = {
  async create(data: CreateUserInput): Promise<User> {
    // Validation
    // Business logic
    // Database operation
    // Return typed result
  },
  
  async findById(id: string): Promise<User | null> {
    // Implementation
  }
}

// API route uses service
// app/api/users/route.ts
export async function POST(request: Request) {
  try {
    const data = await request.json()
    const user = await userService.create(data)
    return NextResponse.json(user)
  } catch (error) {
    return handleError(error)
  }
}
```

### Error Handling
- Services throw typed errors
- API routes catch and return appropriate HTTP responses
- Client components use error boundaries
- Always log errors for debugging

## Available AI Context Modules
Load these modules when working on specific features:

- **01-AUTH.md**: Authentication system, user sessions, protected routes
- **02-PAYMENTS.md**: Stripe integration, subscription management, webhooks
- **03-EMAIL.md**: Email templates, transactional emails, email service
- **04-ADMIN.md**: Admin dashboard, user management, system settings
- **05-LANDING.md**: Marketing pages, SEO optimization, conversion tracking

## Quick Start for AI Assistants
When asked to add features to this realm:
1. Check existing patterns in similar features
2. Use the service layer for business logic
3. Follow the established file structure
4. Maintain TypeScript type safety
5. Use Server Components unless client interactivity is needed
6. Add proper error handling at all layers
7. Update relevant documentation

## Performance Guidelines
- Use `loading.tsx` for better perceived performance
- Implement proper caching strategies (React Cache, Next.js caching)
- Optimize images with Next.js Image component
- Lazy load heavy client components
- Use database indexes for common queries

This realm is designed to be a solid foundation for building production-ready SaaS applications quickly while maintaining code quality and scalability.