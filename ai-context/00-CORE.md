# Core Architecture - AI Context

## 🎯 **Essential Patterns Every AI Assistant Must Know**

This is the **core semantic layer** that applies to all SaaS applications built with this realm, regardless of specific features enabled.

---

## 🏗️ **Architecture Pattern: Service Layer**

```
┌─ Presentation Layer (Next.js App Router)
├─ API Layer (Next.js API Routes - THIN controllers)
├─ Business Logic Layer (Services - ALL logic goes here)
├─ Data Layer (Prisma + PostgreSQL)
└─ Infrastructure (Docker + deployment)
```

**Critical Rule**: API routes are THIN controllers. All business logic goes in services.

---

## 📁 **File Organization Rules**

### **App Router Structure (Next.js 14)**
```
app/
├── (auth)/           # Route group - auth pages (public)
├── (dashboard)/      # Route group - protected pages  
├── admin/            # Admin-only routes
├── api/              # API endpoints
└── layout.tsx        # Root layout
```

**Route Groups Pattern**: Use `(groupname)` for organization without affecting URLs.

### **Service Layer Pattern** 
```
services/
├── user.service.ts      # User management logic
├── billing.service.ts   # Payment logic  
├── email.service.ts     # Email logic
└── admin.service.ts     # Admin logic
```

**Rule**: Every domain gets its own service class with static methods.

### **Component Organization**
```
components/
├── ui/          # shadcn/ui base components
├── forms/       # Form components  
├── layout/      # Layout components
└── providers/   # Context providers
```

**Rule**: Components grouped by PURPOSE, not by type.

---

## 🎯 **Coding Patterns**

### **1. Import Order (ALWAYS follow this)**
```typescript
// 1. External libraries
import React from 'react';
import { NextRequest } from 'next/server';

// 2. Internal config/utilities  
import { db } from '@/lib/db';

// 3. Services (business logic)
import { UserService } from '@/services/user.service';

// 4. Components
import { Button } from '@/components/ui/button';

// 5. Types
import type { User } from '@/types/user.types';
```

### **2. API Route Pattern (ALWAYS use this)**
```typescript
export async function GET(request: NextRequest) {
  try {
    // 1. Auth check (if needed)
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // 2. Extract/validate params
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') ?? '1');

    // 3. Call service layer (NO business logic here!)
    const result = await UserService.getPaginated({ page });

    // 4. Return response
    return NextResponse.json({ data: result });
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' }, 
      { status: 500 }
    );
  }
}
```

**Pattern**: Auth → Validate → Service Call → Response

### **3. Service Layer Pattern (ALWAYS use this)**
```typescript
// services/user.service.ts
export class UserService {
  /**
   * Creates a new user
   * @param data - User creation data
   * @returns Created user (without password)
   */
  static async create(data: CreateUserRequest): Promise<User> {
    // 1. Validate input
    const validated = CreateUserSchema.parse(data);
    
    // 2. Business logic
    const existing = await db.user.findUnique({
      where: { email: validated.email }
    });
    
    if (existing) {
      throw new Error('User already exists');
    }
    
    // 3. Data transformation
    const hashedPassword = await hash(validated.password, 12);
    
    // 4. Database operation
    const user = await db.user.create({
      data: {
        email: validated.email,
        password: hashedPassword,
        name: validated.name,
      }
    });
    
    // 5. Return safe data (no password!)
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}
```

**Pattern**: Validate → Business Logic → Database → Return Safe Data

### **4. Component Pattern (Server + Client)**
```typescript
// Server Component (data fetching)
export default async function DashboardPage() {
  const session = await auth();
  if (!session?.user) redirect('/login');
  
  // Fetch data on server
  const data = await UserService.getDashboardData(session.user.id);
  
  // Pass to client component
  return <DashboardClient data={data} />;
}

// Client Component (interactivity)  
'use client';
export function DashboardClient({ data }: { data: DashboardData }) {
  const [loading, setLoading] = useState(false);
  
  // Handle interactions here
  return <div>{/* Interactive UI */}</div>;
}
```

**Pattern**: Server Component (data) → Client Component (interactions)

---

## 🗄️ **Database Patterns**

### **Prisma Schema Conventions**
```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // Always include audit fields
  @@map("users") // table name in snake_case
}
```

### **Database Query Pattern**
```typescript
// Use transactions for related operations
const result = await db.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.profile.create({ data: { userId: user.id } });
  return user;
});
```

---

## 🎨 **UI Patterns**

### **Form Handling Pattern**
```typescript
'use client';
export function MyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    resolver: zodResolver(MyFormSchema),
  });

  const onSubmit = async (data: FormData) => {
    try {
      const response = await fetch('/api/endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      
      if (!response.ok) throw new Error('Failed');
      
      // Handle success
    } catch (error) {
      // Handle error
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
}
```

---

## 🔒 **Security Rules (NEVER VIOLATE)**

### **1. Input Validation**
```typescript
// ALWAYS validate with Zod schemas
const validated = CreateUserSchema.parse(input);
```

### **2. Password Handling**
```typescript
// ALWAYS hash passwords
const hashed = await hash(password, 12);

// NEVER return passwords in responses
const { password, ...safeUser } = user;
return safeUser;
```

### **3. Authentication**
```typescript
// ALWAYS check auth in protected routes
const session = await auth();
if (!session?.user) {
  return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
}
```

---

## 🧪 **Testing Patterns**

### **Test File Naming**
```
services/__tests__/user.service.test.ts
components/__tests__/login-form.test.tsx
tests/integration/auth.test.ts
tests/e2e/user-flow.spec.ts
```

### **Test Structure**
```typescript
describe('UserService', () => {
  describe('create', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = { /* test data */ };
      
      // Act  
      const result = await UserService.create(userData);
      
      // Assert
      expect(result.email).toBe(userData.email);
    });
  });
});
```

---

## 🎯 **AI Assistant Guidelines**

### **When Adding New Features:**
1. **Follow the Service Layer Pattern** - No business logic in API routes
2. **Use Type Safety** - TypeScript + Zod validation
3. **Follow File Organization** - Components by purpose, services by domain  
4. **Write Tests** - Unit, integration, and E2E
5. **Document Decisions** - Update relevant AI context files

### **When Debugging:**
1. **Check Service Layer** - Business logic issues are usually here
2. **Validate Input** - Most errors come from invalid input
3. **Check Database** - Verify queries and relationships
4. **Review Logs** - Errors are logged with context

### **Common Tasks:**

**Add API Endpoint:**
1. Create route in `app/api/`
2. Follow API route pattern
3. Call appropriate service method
4. Add tests

**Add New Page:**  
1. Create in appropriate app folder
2. Use Server Component for data
3. Use Client Component for interactions
4. Add tests

**Add Service Method:**
1. Add to appropriate service class
2. Follow service pattern
3. Add validation
4. Add tests

---

## 📋 **Quality Checklist**

Before calling a feature "complete":
- [ ] Follows service layer pattern
- [ ] Has proper TypeScript types
- [ ] Includes input validation
- [ ] Has error handling
- [ ] Includes tests
- [ ] Follows naming conventions
- [ ] No business logic in API routes
- [ ] No sensitive data in responses

---

**This core context applies to ALL features. Load additional context files only for features you're actually using.**