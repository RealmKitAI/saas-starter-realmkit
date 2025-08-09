# Testing Strategy - RealmKit SaaS Starter

## 🧪 Comprehensive Testing Approach

This realm includes a complete testing strategy with multiple layers of testing to ensure reliability and maintainability.

---

## 🏗️ Testing Architecture

### Testing Pyramid
```
       🔺 E2E Tests (Playwright)
      /   \  - Critical user flows
     /     \  - Browser automation
    /       \  - Real environment
   ───────────
  🔷 Integration Tests (Jest + MSW)
 /             \  - API endpoints
/               \  - Database operations  
\               /  - External services
 \             /
  ─────────────
    🟦 Unit Tests (Jest + RTL)
   /                         \
  /    - Services             \
 /     - Utilities             \
/      - Components            \
─────────────────────────────────
```

---

## 📋 Test Configuration

### Jest Configuration
```javascript
// jest.config.js
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/tests/setup/jest.setup.ts'],
  testEnvironment: 'jest-environment-jsdom',
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
  collectCoverageFrom: [
    'app/**/*.{ts,tsx}',
    'components/**/*.{ts,tsx}',
    'lib/**/*.{ts,tsx}',
    'services/**/*.{ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/$1',
  },
};

module.exports = createJestConfig(customJestConfig);
```

### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## 🔧 Testing Setup Files

### Jest Setup
```typescript
// tests/setup/jest.setup.ts
import '@testing-library/jest-dom';
import { loadEnvConfig } from '@next/env';

// Load environment variables
loadEnvConfig(process.cwd());

// Mock Next.js router
jest.mock('next/navigation', () => ({
  useRouter: () => ({
    push: jest.fn(),
    replace: jest.fn(),
    back: jest.fn(),
    refresh: jest.fn(),
  }),
  usePathname: () => '/test-path',
  useSearchParams: () => new URLSearchParams(),
}));

// Mock NextAuth
jest.mock('next-auth', () => ({
  default: jest.fn(),
}));

jest.mock('next-auth/react', () => ({
  useSession: jest.fn(() => ({
    data: null,
    status: 'unauthenticated',
  })),
  signIn: jest.fn(),
  signOut: jest.fn(),
}));

// Global test utilities
global.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn(),
}));

// Suppress console.error in tests unless needed
const originalError = console.error;
beforeAll(() => {
  console.error = (...args) => {
    if (
      typeof args[0] === 'string' &&
      args[0].includes('Warning: ReactDOM.render is no longer supported')
    ) {
      return;
    }
    originalError.call(console, ...args);
  };
});

afterAll(() => {
  console.error = originalError;
});
```

### Test Database Setup
```typescript
// tests/setup/database.setup.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.TEST_DATABASE_URL,
    },
  },
});

export async function setupTestDatabase() {
  // Clean database before tests
  await prisma.$transaction([
    prisma.session.deleteMany(),
    prisma.account.deleteMany(),
    prisma.subscription.deleteMany(),
    prisma.user.deleteMany(),
  ]);
}

export async function teardownTestDatabase() {
  // Clean up after tests
  await prisma.$transaction([
    prisma.session.deleteMany(),
    prisma.account.deleteMany(),
    prisma.subscription.deleteMany(),
    prisma.user.deleteMany(),
  ]);
  
  await prisma.$disconnect();
}

// Test data factories
export const createTestUser = (overrides = {}) => ({
  email: 'test@example.com',
  name: 'Test User',
  password: 'hashedpassword123',
  ...overrides,
});

export const createTestSubscription = (userId: string, overrides = {}) => ({
  userId,
  plan: 'FREE',
  status: 'ACTIVE',
  ...overrides,
});
```

---

## 🏠 Unit Testing Patterns

### Testing Services
```typescript
// services/__tests__/user.service.test.ts
import { UserService } from '../user.service';
import { db } from '@/lib/db';
import { hash } from 'bcryptjs';

// Mock dependencies
jest.mock('@/lib/db');
jest.mock('bcryptjs');

const mockDb = db as jest.Mocked<typeof db>;
const mockHash = hash as jest.MockedFunction<typeof hash>;

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

      mockDb.user.findUnique.mockResolvedValue(null);
      mockHash.mockResolvedValue('hashed_password');
      mockDb.user.create.mockResolvedValue({
        id: 'user_123',
        email: userData.email,
        name: userData.name,
        password: 'hashed_password',
        role: 'USER',
        createdAt: new Date(),
        updatedAt: new Date(),
      });

      // Act
      const result = await UserService.create(userData);

      // Assert
      expect(result).toEqual({
        id: 'user_123',
        email: userData.email,
        name: userData.name,
        createdAt: expect.any(Date),
      });

      expect(mockDb.user.findUnique).toHaveBeenCalledWith({
        where: { email: userData.email },
      });
      
      expect(mockHash).toHaveBeenCalledWith(userData.password, 12);
      
      expect(mockDb.user.create).toHaveBeenCalledWith({
        data: {
          email: userData.email,
          password: 'hashed_password',
          name: userData.name,
        },
      });
    });

    it('should throw error if user already exists', async () => {
      // Arrange
      const userData = {
        email: 'existing@example.com',
        password: 'password123',
      };

      mockDb.user.findUnique.mockResolvedValue({
        id: 'existing_user',
        email: userData.email,
        name: 'Existing User',
        password: 'hashed',
        role: 'USER',
        createdAt: new Date(),
        updatedAt: new Date(),
      });

      // Act & Assert
      await expect(UserService.create(userData)).rejects.toThrow(
        'User already exists'
      );
    });

    it('should validate required fields', async () => {
      // Act & Assert
      await expect(
        UserService.create({ email: '', password: 'test' })
      ).rejects.toThrow('Email and password are required');

      await expect(
        UserService.create({ email: 'test@example.com', password: '' })
      ).rejects.toThrow('Email and password are required');
    });
  });

  describe('authenticate', () => {
    it('should authenticate valid credentials', async () => {
      // Arrange
      const credentials = {
        email: 'test@example.com',
        password: 'password123',
      };

      const mockUser = {
        id: 'user_123',
        email: credentials.email,
        name: 'Test User',
        password: 'hashed_password',
        role: 'USER',
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      mockDb.user.findUnique.mockResolvedValue(mockUser);
      (bcrypt.compare as jest.Mock).mockResolvedValue(true);

      // Act
      const result = await UserService.authenticate(
        credentials.email,
        credentials.password
      );

      // Assert
      expect(result).toEqual({
        id: mockUser.id,
        email: mockUser.email,
        name: mockUser.name,
        role: mockUser.role,
      });
    });

    it('should return null for invalid credentials', async () => {
      // Arrange
      mockDb.user.findUnique.mockResolvedValue(null);

      // Act
      const result = await UserService.authenticate(
        'nonexistent@example.com',
        'password'
      );

      // Assert
      expect(result).toBeNull();
    });
  });
});
```

### Testing React Components
```typescript
// components/__tests__/login-form.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from '../forms/login-form';

// Mock next/router
const mockPush = jest.fn();
jest.mock('next/navigation', () => ({
  useRouter: () => ({
    push: mockPush,
  }),
}));

// Mock next-auth
const mockSignIn = jest.fn();
jest.mock('next-auth/react', () => ({
  signIn: mockSignIn,
}));

describe('LoginForm', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render login form', () => {
    render(<LoginForm />);

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /sign in/i })).toBeInTheDocument();
  });

  it('should show validation errors for empty fields', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    const submitButton = screen.getByRole('button', { name: /sign in/i });
    await user.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
      expect(screen.getByText(/password is required/i)).toBeInTheDocument();
    });
  });

  it('should submit form with valid data', async () => {
    const user = userEvent.setup();
    mockSignIn.mockResolvedValue({ ok: true });

    render(<LoginForm />);

    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /sign in/i });

    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'password123');
    await user.click(submitButton);

    await waitFor(() => {
      expect(mockSignIn).toHaveBeenCalledWith('credentials', {
        email: 'test@example.com',
        password: 'password123',
        redirect: false,
      });
    });
  });

  it('should display error message on failed login', async () => {
    const user = userEvent.setup();
    mockSignIn.mockResolvedValue({ 
      ok: false, 
      error: 'Invalid credentials' 
    });

    render(<LoginForm />);

    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /sign in/i });

    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'wrongpassword');
    await user.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
    });
  });

  it('should disable form during submission', async () => {
    const user = userEvent.setup();
    mockSignIn.mockImplementation(() => new Promise(resolve => 
      setTimeout(() => resolve({ ok: true }), 100)
    ));

    render(<LoginForm />);

    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /sign in/i });

    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'password123');
    await user.click(submitButton);

    // Form should be disabled during submission
    expect(submitButton).toBeDisabled();
    expect(emailInput).toBeDisabled();
    expect(passwordInput).toBeDisabled();

    await waitFor(() => {
      expect(submitButton).not.toBeDisabled();
    });
  });
});
```

---

## 🔗 Integration Testing

### API Route Testing
```typescript
// tests/integration/api/auth/register.test.ts
import { createMocks } from 'node-mocks-http';
import { POST } from '@/app/api/auth/register/route';
import { setupTestDatabase, teardownTestDatabase, createTestUser } from '@/tests/setup/database.setup';

describe('/api/auth/register', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  beforeEach(async () => {
    await setupTestDatabase(); // Clean slate for each test
  });

  it('should register new user successfully', async () => {
    const userData = {
      email: 'newuser@example.com',
      password: 'password123',
      name: 'New User',
    };

    const request = new Request('http://localhost:3000/api/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(201);
    expect(data.user.email).toBe(userData.email);
    expect(data.user.name).toBe(userData.name);
    expect(data.user).not.toHaveProperty('password');

    // Verify user was created in database
    const createdUser = await db.user.findUnique({
      where: { email: userData.email },
    });
    expect(createdUser).toBeTruthy();
    expect(createdUser?.email).toBe(userData.email);
  });

  it('should reject duplicate email', async () => {
    // Create user first
    const existingUserData = createTestUser({
      email: 'existing@example.com',
    });
    
    await db.user.create({
      data: existingUserData,
    });

    // Try to create user with same email
    const request = new Request('http://localhost:3000/api/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: 'existing@example.com',
        password: 'password123',
        name: 'Duplicate User',
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(400);
    expect(data.error).toBe('User already exists');
  });

  it('should validate input data', async () => {
    const invalidData = {
      email: 'invalid-email',
      password: '123', // Too short
      name: '', // Empty name
    };

    const request = new Request('http://localhost:3000/api/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(invalidData),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(400);
    expect(data.errors).toBeDefined();
    expect(data.errors.email).toContain('Invalid email format');
    expect(data.errors.password).toContain('Password must be at least 8 characters');
    expect(data.errors.name).toContain('Name is required');
  });

  it('should handle database errors gracefully', async () => {
    // Mock database error
    jest.spyOn(db.user, 'create').mockRejectedValueOnce(
      new Error('Database connection failed')
    );

    const request = new Request('http://localhost:3000/api/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(500);
    expect(data.error).toBe('Internal server error');

    // Restore mock
    jest.restoreAllMocks();
  });
});
```

### Database Testing with MSW
```typescript
// tests/setup/msw.setup.ts
import { setupServer } from 'msw/node';
import { rest } from 'msw';

// Mock external API calls
export const server = setupServer(
  // Mock Stripe API
  rest.post('https://api.stripe.com/v1/checkout/sessions', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        id: 'cs_test_123',
        url: 'https://checkout.stripe.com/pay/cs_test_123',
      })
    );
  }),

  // Mock email service
  rest.post('https://api.sendgrid.com/v3/mail/send', (req, res, ctx) => {
    return res(
      ctx.status(202),
      ctx.json({ message: 'Email sent successfully' })
    );
  }),
);

// Enable API mocking before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset handlers between tests
afterEach(() => server.resetHandlers());

// Disable API mocking after all tests
afterAll(() => server.close());
```

---

## 🎭 End-to-End Testing

### User Flow Tests
```typescript
// tests/e2e/user-registration-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Registration Flow', () => {
  test('complete user registration and onboarding', async ({ page }) => {
    // Navigate to registration page
    await page.goto('/register');

    // Fill registration form
    await page.fill('input[name="email"]', 'e2e-test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="name"]', 'E2E Test User');

    // Submit form
    await page.click('button[type="submit"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');

    // Should show welcome message
    await expect(page.locator('h1')).toContainText('Welcome, E2E Test User');

    // Should show onboarding checklist
    await expect(page.locator('[data-testid="onboarding-checklist"]')).toBeVisible();

    // Complete profile setup
    await page.click('[data-testid="complete-profile-button"]');
    await page.fill('input[name="company"]', 'Test Company');
    await page.selectOption('select[name="role"]', 'developer');
    await page.click('button[type="submit"]');

    // Should update onboarding progress
    await expect(page.locator('[data-testid="profile-complete-check"]')).toBeChecked();

    // Should be able to navigate to settings
    await page.click('[data-testid="settings-link"]');
    await expect(page).toHaveURL('/settings');
    await expect(page.locator('input[name="name"]')).toHaveValue('E2E Test User');
  });

  test('should show validation errors for invalid input', async ({ page }) => {
    await page.goto('/register');

    // Try to submit empty form
    await page.click('button[type="submit"]');

    // Should show validation errors
    await expect(page.locator('[data-testid="email-error"]')).toContainText(
      'Email is required'
    );
    await expect(page.locator('[data-testid="password-error"]')).toContainText(
      'Password is required'
    );

    // Fill invalid email
    await page.fill('input[name="email"]', 'invalid-email');
    await page.blur('input[name="email"]');
    
    await expect(page.locator('[data-testid="email-error"]')).toContainText(
      'Invalid email format'
    );

    // Fill short password
    await page.fill('input[name="password"]', '123');
    await page.blur('input[name="password"]');
    
    await expect(page.locator('[data-testid="password-error"]')).toContainText(
      'Password must be at least 8 characters'
    );
  });

  test('should handle existing user registration', async ({ page }) => {
    // First, create a user (this would be done in test setup)
    await page.goto('/register');
    
    await page.fill('input[name="email"]', 'existing@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="name"]', 'Existing User');
    
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/dashboard');
    
    // Sign out
    await page.click('[data-testid="user-menu"]');
    await page.click('[data-testid="sign-out"]');
    
    // Try to register again with same email
    await page.goto('/register');
    
    await page.fill('input[name="email"]', 'existing@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="name"]', 'Duplicate User');
    
    await page.click('button[type="submit"]');
    
    // Should show error message
    await expect(page.locator('[data-testid="form-error"]')).toContainText(
      'User already exists'
    );
  });
});
```

### Authentication Flow Tests
```typescript
// tests/e2e/authentication-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication Flow', () => {
  test('login and logout flow', async ({ page }) => {
    // Go to login page
    await page.goto('/login');

    // Fill login form
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');

    // Submit login
    await page.click('button[type="submit"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');

    // Should show user info
    await expect(page.locator('[data-testid="user-name"]')).toContainText(
      'Test User'
    );

    // Logout
    await page.click('[data-testid="user-menu"]');
    await page.click('[data-testid="sign-out"]');

    // Should redirect to home
    await expect(page).toHaveURL('/');

    // Should not be able to access dashboard
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login');
  });

  test('should protect authenticated routes', async ({ page }) => {
    const protectedRoutes = ['/dashboard', '/settings', '/billing', '/admin'];

    for (const route of protectedRoutes) {
      await page.goto(route);
      await expect(page).toHaveURL('/login');
    }
  });

  test('should remember user after page refresh', async ({ page }) => {
    // Login first
    await page.goto('/login');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    await expect(page).toHaveURL('/dashboard');

    // Refresh page
    await page.reload();

    // Should still be logged in
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-name"]')).toContainText(
      'Test User'
    );
  });
});
```

---

## 🚀 CI/CD Testing Pipeline

### GitHub Actions Configuration
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_USER: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Setup test database
        run: |
          npx prisma migrate deploy
        env:
          TEST_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Run unit tests
        run: npm run test:unit
        env:
          TEST_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Run integration tests
        run: npm run test:integration
        env:
          TEST_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3

  e2e-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_USER: postgres  
          POSTGRES_DB: e2e_test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Setup test database
        run: |
          npx prisma migrate deploy
          npx prisma db seed
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/e2e_test_db

      - name: Build application
        run: npm run build
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/e2e_test_db

      - name: Run E2E tests
        run: npm run test:e2e
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/e2e_test_db

      - name: Upload Playwright report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## 📊 Test Scripts

### Package.json Scripts
```json
{
  "scripts": {
    "test": "npm run test:unit && npm run test:integration && npm run test:e2e",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "playwright test",
    "test:watch": "jest --watch --testPathPattern=unit",
    "test:coverage": "jest --coverage",
    "test:ui": "playwright test --ui",
    "test:debug": "playwright test --debug"
  }
}
```

---

## 📈 Testing Best Practices

### 1. Test Organization
- **Unit tests**: Test individual functions/components in isolation
- **Integration tests**: Test API endpoints and database operations
- **E2E tests**: Test complete user workflows

### 2. Test Data Management
- Use factories for creating test data
- Clean database between tests
- Use separate test database
- Mock external services

### 3. Test Naming Convention
```typescript
describe('UserService', () => {
  describe('create', () => {
    it('should create user with valid data', () => {
      // Test implementation
    });
    
    it('should throw error when email already exists', () => {
      // Test implementation
    });
    
    it('should validate required fields', () => {
      // Test implementation
    });
  });
});
```

### 4. Coverage Goals
- **Overall coverage**: 80%+
- **Critical paths**: 100% (auth, payments, security)
- **Business logic**: 90%+ (services)
- **UI components**: 70%+ (focus on logic)

### 5. Performance Testing
```typescript
// Example performance test
test('user creation should complete within 500ms', async () => {
  const start = Date.now();
  
  await UserService.create({
    email: 'perf-test@example.com',
    password: 'password123',
    name: 'Performance Test',
  });
  
  const duration = Date.now() - start;
  expect(duration).toBeLessThan(500);
});
```

---

This comprehensive testing strategy ensures that the RealmKit SaaS Starter is reliable, maintainable, and production-ready. Every feature is tested at multiple levels, and the testing infrastructure supports continuous development and deployment.