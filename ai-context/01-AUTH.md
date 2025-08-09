# Authentication Module - AI Context

## 🔐 **Authentication Patterns with NextAuth.js**

Load this context when working with authentication features.

---

## 🏗️ **NextAuth.js Architecture**

### **Configuration Pattern**
```typescript
// lib/auth.ts
import { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { PrismaAdapter } from '@next-auth/prisma-adapter';

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(db),
  session: { strategy: 'jwt' },
  
  providers: [
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
  ],
  
  pages: {
    signIn: '/login',
    signUp: '/register',
  },
  
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role;
      }
      return token;
    },
    
    async session({ session, token }) {
      if (session.user && token.sub) {
        session.user.id = token.sub;
        session.user.role = token.role as string;
      }
      return session;
    },
  },
};
```

---

## 🔒 **Authentication Service Patterns**

### **User Authentication Service**
```typescript
// services/user.service.ts
export class UserService {
  /**
   * Authenticates user with email/password
   */
  static async authenticate(
    email: string,
    password: string
  ): Promise<User | null> {
    // Find user by email
    const user = await db.user.findUnique({
      where: { email: email.toLowerCase() },
    });
    
    if (!user) return null;
    
    // Verify password
    const isValid = await compare(password, user.password);
    if (!isValid) return null;
    
    // Return user without password
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
    };
  }

  /**
   * Creates new user with hashed password
   */
  static async create(data: CreateUserRequest): Promise<User> {
    // Validate input
    const validated = CreateUserSchema.parse(data);
    
    // Check if user exists
    const existing = await db.user.findUnique({
      where: { email: validated.email.toLowerCase() },
    });
    
    if (existing) {
      throw new Error('User already exists');
    }
    
    // Hash password
    const hashedPassword = await hash(validated.password, 12);
    
    // Create user
    const user = await db.user.create({
      data: {
        email: validated.email.toLowerCase(),
        password: hashedPassword,
        name: validated.name,
      },
    });
    
    // Return without password
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
      createdAt: user.createdAt,
    };
  }
}
```

---

## 🛡️ **Route Protection Patterns**

### **API Route Protection**
```typescript
// app/api/protected/route.ts
export async function GET(request: NextRequest) {
  // Always check authentication first
  const session = await auth();
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // Optional role check
  if (session.user.role !== 'ADMIN') {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }
  
  // Continue with business logic
  const data = await SomeService.getData(session.user.id);
  return NextResponse.json({ data });
}
```

### **Page Protection (Server Component)**
```typescript
// app/(dashboard)/settings/page.tsx
export default async function SettingsPage() {
  const session = await auth();
  
  if (!session?.user) {
    redirect('/login');
  }
  
  // Fetch user data
  const userData = await UserService.getById(session.user.id);
  
  return <SettingsClient user={userData} />;
}
```

### **Client-Side Protection**
```typescript
// components/protected-component.tsx
'use client';
import { useSession } from 'next-auth/react';

export function ProtectedComponent() {
  const { data: session, status } = useSession();
  
  if (status === 'loading') {
    return <div>Loading...</div>;
  }
  
  if (!session) {
    return <div>Please sign in</div>;
  }
  
  return <div>Protected content for {session.user?.name}</div>;
}
```

---

## 📝 **Authentication Forms**

### **Login Form Pattern**
```typescript
// components/forms/login-form.tsx
'use client';

const LoginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(1, 'Password required'),
});

export function LoginForm() {
  const router = useRouter();
  const [error, setError] = useState<string | null>(null);
  
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(LoginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    setError(null);
    
    try {
      const result = await signIn('credentials', {
        email: data.email,
        password: data.password,
        redirect: false,
      });

      if (result?.error) {
        setError('Invalid email or password');
        return;
      }

      router.push('/dashboard');
      router.refresh();
    } catch (error) {
      setError('Something went wrong');
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
          <p className="text-red-500 text-sm">{errors.email.message}</p>
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
          <p className="text-red-500 text-sm">{errors.password.message}</p>
        )}
      </div>

      {error && (
        <p className="text-red-500 text-sm">{error}</p>
      )}

      <Button type="submit" disabled={isSubmitting} className="w-full">
        {isSubmitting ? 'Signing in...' : 'Sign in'}
      </Button>
    </form>
  );
}
```

### **Registration API Route**
```typescript
// app/api/auth/register/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // Validate input
    const userData = CreateUserSchema.parse(body);
    
    // Create user (service handles validation and hashing)
    const user = await UserService.create(userData);
    
    return NextResponse.json(
      { user, message: 'User created successfully' },
      { status: 201 }
    );
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', errors: error.flatten().fieldErrors },
        { status: 400 }
      );
    }
    
    if (error instanceof Error && error.message === 'User already exists') {
      return NextResponse.json(
        { error: error.message },
        { status: 400 }
      );
    }
    
    console.error('Registration error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## 🔐 **Password Management**

### **Password Hashing**
```typescript
import { hash, compare } from 'bcryptjs';

// Always use 12 rounds for production
const SALT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return await hash(password, SALT_ROUNDS);
}

export async function verifyPassword(
  password: string,
  hashedPassword: string
): Promise<boolean> {
  return await compare(password, hashedPassword);
}
```

### **Password Reset Flow**
```typescript
// services/auth.service.ts
export class AuthService {
  /**
   * Initiates password reset process
   */
  static async initiatePasswordReset(email: string): Promise<void> {
    const user = await db.user.findUnique({
      where: { email: email.toLowerCase() },
    });
    
    if (!user) {
      // Don't reveal if email exists
      return;
    }
    
    // Generate reset token
    const token = crypto.randomUUID();
    const expiresAt = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 hours
    
    // Save reset token
    await db.passwordReset.create({
      data: {
        userId: user.id,
        token,
        expiresAt,
      },
    });
    
    // Send reset email
    await EmailService.sendPasswordReset(user.email, token);
  }

  /**
   * Resets password with token
   */
  static async resetPassword(
    token: string,
    newPassword: string
  ): Promise<void> {
    // Find valid reset token
    const resetRequest = await db.passwordReset.findFirst({
      where: {
        token,
        expiresAt: { gt: new Date() },
        usedAt: null,
      },
      include: { user: true },
    });
    
    if (!resetRequest) {
      throw new Error('Invalid or expired reset token');
    }
    
    // Hash new password
    const hashedPassword = await hash(newPassword, 12);
    
    // Update user password and mark token as used
    await db.$transaction([
      db.user.update({
        where: { id: resetRequest.userId },
        data: { password: hashedPassword },
      }),
      db.passwordReset.update({
        where: { id: resetRequest.id },
        data: { usedAt: new Date() },
      }),
    ]);
  }
}
```

---

## 👥 **Role-Based Access Control**

### **Role Definition**
```typescript
// types/auth.types.ts
export enum UserRole {
  USER = 'USER',
  ADMIN = 'ADMIN',
  SUPER_ADMIN = 'SUPER_ADMIN',
}

export enum Permission {
  READ_USERS = 'read:users',
  WRITE_USERS = 'write:users',
  DELETE_USERS = 'delete:users',
  READ_ANALYTICS = 'read:analytics',
  MANAGE_SYSTEM = 'manage:system',
}
```

### **Permission System**
```typescript
// lib/permissions.ts
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
  return async function(request: NextRequest) {
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

## 🧪 **Authentication Testing**

### **Service Testing**
```typescript
// services/__tests__/user.service.test.ts
describe('UserService.authenticate', () => {
  it('should authenticate valid credentials', async () => {
    // Mock user in database
    const mockUser = {
      id: '123',
      email: 'test@example.com',
      password: await hash('password123', 12),
      name: 'Test User',
      role: 'USER',
    };
    
    mockDb.user.findUnique.mockResolvedValue(mockUser);
    
    const result = await UserService.authenticate(
      'test@example.com',
      'password123'
    );
    
    expect(result).toEqual({
      id: '123',
      email: 'test@example.com',
      name: 'Test User',
      role: 'USER',
    });
  });

  it('should return null for invalid credentials', async () => {
    mockDb.user.findUnique.mockResolvedValue(null);
    
    const result = await UserService.authenticate(
      'nonexistent@example.com',
      'password'
    );
    
    expect(result).toBeNull();
  });
});
```

### **API Route Testing**
```typescript
// tests/integration/auth.test.ts
describe('/api/auth/register', () => {
  it('should register new user', async () => {
    const userData = {
      email: 'newuser@example.com',
      password: 'password123',
      name: 'New User',
    };

    const response = await request(app)
      .post('/api/auth/register')
      .send(userData);

    expect(response.status).toBe(201);
    expect(response.body.user.email).toBe(userData.email);
    expect(response.body.user).not.toHaveProperty('password');
  });
});
```

---

## 🎯 **Authentication Checklist**

When adding authentication features:
- [ ] Use NextAuth.js configuration pattern
- [ ] Hash passwords with bcrypt (12 rounds)
- [ ] Validate input with Zod schemas  
- [ ] Never return passwords in responses
- [ ] Check authentication in protected routes
- [ ] Handle errors gracefully
- [ ] Add proper TypeScript types
- [ ] Write tests for auth flows
- [ ] Use secure session strategy (JWT)
- [ ] Implement proper role-based access control

---

**This authentication context provides all patterns for NextAuth.js implementation. Only load when authentication features are needed.**