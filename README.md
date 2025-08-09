# RealmKit SaaS Starter Realm 🏰

The complete, AI-optimized SaaS foundation that scales from MVP to enterprise.

> **🎯 Get Started with RealmKit CLI** - The fastest way to create production-ready SaaS applications

## 🚀 Quick Start (Recommended)

Install your SaaS with the [RealmKit CLI](https://realmkit.com/cli) for the best experience:

```bash
# Install RealmKit CLI
npm install -g @realmkit/cli

# Create your SaaS project instantly
realmkit create my-saas-app realmkitai/saas-starter

# Start building your SaaS
cd my-saas-app
docker-compose up
```

**That's it!** Your SaaS is running at `http://localhost:3000`

### Alternative: Manual Clone

```bash
git clone https://github.com/RealmKitAI/saas-starter-realmkit.git my-saas-app
cd my-saas-app
docker-compose up
```

## 🌟 Why Use RealmKit?

- **🤖 AI-First**: Built for seamless integration with AI coding assistants
- **⚡ Lightning Fast**: Skip weeks of boilerplate setup
- **🎯 Production Ready**: Battle-tested architecture and patterns
- **🔄 Stay Updated**: Get the latest improvements with `realmkit update`
- **🏪 Discover More**: Browse 100+ realms at [RealmKit Hub](https://realmkit.com/browse)

- ✅ Authentication system
- ✅ Payment integration (Stripe)
- ✅ Admin dashboard
- ✅ User management
- ✅ Database with migrations
- ✅ Email system
- ✅ Full test suite
- ✅ Production deployment ready

---

## 🏗️ Architecture

This realm follows the **Layered SaaS Architecture** pattern, optimized for AI coding assistants:

```
┌─ Frontend Layer (Next.js + TypeScript)
├─ API Layer (Next.js API Routes)
├─ Service Layer (Business Logic)
├─ Data Layer (Prisma + PostgreSQL)
└─ Infrastructure Layer (Docker + Deployment)
```

### 📁 File Structure
```
saas-starter-realm/
├── 📄 AI-CONTEXT.md              # AI assistant guide
├── 📄 ARCHITECTURE.md            # System design
├── 📄 TESTING.md                 # Testing strategy
├── 📄 DEPLOYMENT.md              # Deploy guide
├── 
├── 🐳 docker-compose.yml         # Container setup
├── 🐳 Dockerfile                 # Production image
├── 
├── 📱 app/                       # Next.js App Router
│   ├── (auth)/                   # Auth pages group
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (dashboard)/              # Protected routes
│   │   ├── dashboard/page.tsx
│   │   ├── settings/page.tsx
│   │   └── billing/page.tsx
│   ├── admin/                    # Admin panel
│   ├── api/                      # API routes
│   │   ├── auth/[...nextauth]/
│   │   ├── users/
│   │   ├── billing/
│   │   └── webhooks/
│   ├── globals.css               # Tailwind styles
│   └── layout.tsx                # Root layout
│
├── 🧩 components/                # Reusable UI components
│   ├── ui/                       # shadcn/ui components
│   ├── forms/                    # Form components
│   ├── layout/                   # Layout components
│   └── charts/                   # Data visualization
│
├── 🔧 lib/                       # Utilities and config
│   ├── auth.ts                   # NextAuth config
│   ├── db.ts                     # Database client
│   ├── stripe.ts                 # Payment config
│   ├── email.ts                  # Email service
│   └── utils.ts                  # Helper functions
│
├── 🔐 services/                  # Business logic layer
│   ├── user.service.ts
│   ├── billing.service.ts
│   ├── admin.service.ts
│   └── email.service.ts
│
├── 🗄️ prisma/                    # Database layer
│   ├── schema.prisma             # Database schema
│   ├── migrations/               # Database migrations
│   └── seed.ts                   # Seed data
│
├── 🧪 tests/                     # Test suite
│   ├── __mocks__/                # Test mocks
│   ├── unit/                     # Unit tests
│   ├── integration/              # API tests
│   ├── e2e/                      # End-to-end tests
│   └── setup/                    # Test configuration
│
├── 📚 docs/                      # Documentation
│   ├── ai-context/               # AI assistant guides
│   ├── examples/                 # Usage examples
│   └── deployment/               # Deploy guides
│
└── 🔧 Config files
    ├── .env.example
    ├── next.config.js
    ├── tailwind.config.js
    ├── jest.config.js
    ├── playwright.config.ts
    └── tsconfig.json
```

---

## 🤖 AI-Optimized Features

This realm is designed to work perfectly with coding assistants like ChatGPT, Claude, and Cursor:

### 📋 Semantic Layer
- **AI-CONTEXT.md**: Complete guide for AI assistants
- **Clear naming conventions**: Self-documenting code
- **Type-safe everything**: TypeScript throughout
- **Consistent patterns**: Same approach everywhere

### 🎯 AI-Friendly Patterns
- **Service layer pattern**: Business logic separated
- **Single responsibility**: Each file has one job
- **Explicit imports**: No barrel exports confusion
- **Documented decisions**: Why, not just what

### 🧪 Test-Driven Development
- **Comprehensive test suite**: Unit, integration, E2E
- **Test utilities**: Helpers for common scenarios
- **Mock strategies**: Consistent mocking patterns
- **CI/CD ready**: Automated testing pipeline

---

## 🛠️ Tech Stack

```yaml
Frontend:
  - Next.js 14 (App Router)
  - TypeScript
  - Tailwind CSS
  - shadcn/ui components
  - React Hook Form
  
Backend:
  - Next.js API Routes
  - Prisma ORM
  - PostgreSQL
  - NextAuth.js
  
Payments:
  - Stripe (subscriptions)
  - Webhook handling
  - Invoice generation
  
Testing:
  - Jest (unit tests)
  - React Testing Library
  - Playwright (E2E)
  - Mock Service Worker
  
Infrastructure:
  - Docker Compose
  - GitHub Actions
  - Deployment configs
```

---

## 🚀 Quick Start Guide

### 1. Clone and Setup
```bash
git clone <this-repo> my-saas
cd my-saas
cp .env.example .env.local
```

### 2. Configure Environment
```bash
# Edit .env.local with your values
NEXTAUTH_SECRET=your-secret-here
STRIPE_SECRET_KEY=sk_test_...
DATABASE_URL=postgresql://...
```

### 3. Start Development
```bash
# Using Docker (recommended)
docker-compose up

# Or local development
npm install
npm run dev
```

### 4. Run Tests
```bash
# All tests
npm test

# Specific test types
npm run test:unit
npm run test:integration  
npm run test:e2e
```

### 5. Deploy
```bash
# Railway
railway up

# Or any Docker platform
docker build -t my-saas .
```

---

## 📖 Documentation

- 📄 **[AI-CONTEXT.md](./AI-CONTEXT.md)** - Complete AI assistant guide
- 🏗️ **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System design and patterns
- 🧪 **[TESTING.md](./TESTING.md)** - Testing strategy and examples
- 🚀 **[DEPLOYMENT.md](./DEPLOYMENT.md)** - Deploy anywhere guide
- 🔧 **[CUSTOMIZATION.md](./CUSTOMIZATION.md)** - How to modify and extend

---

## 🎯 What You Get

### Core Features
- **Authentication**: Login, register, password reset, social login
- **User Management**: Profiles, settings, admin panel
- **Billing**: Stripe subscriptions, invoices, usage tracking  
- **Email**: Transactional emails, templates, queues
- **Admin Dashboard**: User management, analytics, system health

### Developer Experience
- **Type Safety**: TypeScript everywhere with strict mode
- **Hot Reload**: Instant feedback during development
- **Database UI**: Built-in database browser
- **API Documentation**: Auto-generated OpenAPI specs
- **Error Handling**: Comprehensive error boundaries

### Production Ready
- **Performance**: Optimized builds, caching, CDN
- **Security**: CSRF protection, rate limiting, input validation
- **Monitoring**: Health checks, logging, error tracking
- **Scalability**: Database indexing, query optimization

---

## 🧪 Testing Strategy

This realm includes a comprehensive testing strategy:

### Unit Tests (Jest + RTL)
```typescript
// Example: services/__tests__/user.service.test.ts
describe('UserService', () => {
  it('should create user with valid data', async () => {
    const user = await UserService.create({
      email: 'test@example.com',
      password: 'password123'
    });
    
    expect(user.email).toBe('test@example.com');
    expect(user.password).not.toBe('password123'); // hashed
  });
});
```

### Integration Tests (API Testing)
```typescript
// Example: tests/integration/auth.test.ts
describe('/api/auth', () => {
  it('should authenticate valid user', async () => {
    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: 'user@test.com', password: 'password' });
      
    expect(response.status).toBe(200);
    expect(response.body.user.email).toBe('user@test.com');
  });
});
```

### E2E Tests (Playwright)
```typescript
// Example: tests/e2e/user-flow.spec.ts
test('complete user registration flow', async ({ page }) => {
  await page.goto('/register');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

---

## 🌟 Next Steps

1. **Customize branding** in `app/layout.tsx`
2. **Add your features** following the established patterns  
3. **Configure payments** in `lib/stripe.ts`
4. **Set up monitoring** with your preferred service
5. **Deploy to production** using the deployment guides

---

## 🏪 Explore More Realms

**Discover hundreds of AI-optimized project templates:**

- 🚀 **Browse All Realms**: [realmkit.com/browse](https://realmkit.com/browse)
- 📱 **Mobile Apps**: React Native, Flutter, and native iOS/Android starters
- 🤖 **AI Applications**: RAG systems, chatbots, and ML model servers  
- 🛒 **E-commerce**: Full-featured online stores with payments and inventory
- 🎮 **Games**: Unity, Godot, and web game templates
- 📊 **Analytics**: Data dashboards, reporting tools, and visualization apps

**Popular Realms:**
```bash
realmkit create my-app realmkitai/nextjs-ai-chatbot    # AI-powered chatbot
realmkit create my-store realmkitai/ecommerce-pro     # Complete e-commerce  
realmkit create my-api realmkitai/fastapi-postgres    # High-performance API
```

---

## 🤝 Join the RealmKit Community

**This realm is part of the RealmKit ecosystem. Build together, ship faster:**

- 🌟 **Star this realm** on [RealmKit Hub](https://realmkit.com/realms/realmkitai/saas-starter)
- 💡 **Share improvements** - Fork, enhance, and publish your version
- 🗣️ **Join discussions** - Connect with other builders in our community
- 📢 **Publish your realm** - Share your templates with `realmkit publish`

**Contributing:**
1. Fork this realm and make improvements
2. Test your changes thoroughly  
3. Publish back with `realmkit publish --link-github`
4. Earn reputation and help others build faster!

---

## 📞 Get Help & Connect

- 📚 **Documentation**: [RealmKit Docs](https://realmkit.com/docs)
- 🐛 **Issues**: [GitHub Issues](https://github.com/RealmKitAI/saas-starter-realmkit/issues)
- 💬 **Community**: [RealmKit Discord](https://discord.gg/realmkit)
- 🎓 **Tutorials**: [RealmKit Academy](https://realmkit.com/learn)
- 📧 **Email**: support@realmkit.com

---

<div align="center">

**🏰 Built with [RealmKit](https://realmkit.com) - The AI-First Development Platform**

*Skip the boilerplate. Ship faster. Build better.*

[![RealmKit Hub](https://img.shields.io/badge/Explore-RealmKit%20Hub-6366f1?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEyIDJMMTMuMDkgOC4yNkwyMCA5TDEzLjA5IDE1Ljc0TDEyIDIyTDEwLjkxIDE1Ljc0TDQgOUwxMC45MSA4LjI2TDEyIDJaIiBmaWxsPSJ3aGl0ZSIvPgo8L3N2Zz4K)](https://realmkit.com/browse)
[![CLI](https://img.shields.io/badge/Install-RealmKit%20CLI-22c55e?style=for-the-badge&logo=terminal)](https://realmkit.com/cli)
[![Discord](https://img.shields.io/badge/Join-Community-5865f2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/realmkit)

*This realm follows RealmKit standards for AI-assisted development.<br/>
Every architectural decision is documented, every pattern is consistent, and every feature is tested.*

</div>