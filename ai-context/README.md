# AI Context Modules - Index

## 🎯 **Modular Semantic Layer for AI Assistants**

This directory contains modular AI context files that provide specific patterns and knowledge for building features in the SaaS Starter Realm. **Load only the context files you need** to avoid overwhelming the AI assistant.

---

## 📁 **Context Files Overview**

### **00-CORE.md** ⚡ *(Always Required)*
- **Purpose**: Essential patterns every AI assistant must know
- **When to Load**: For ALL SaaS development tasks
- **Contains**: Service layer architecture, API patterns, file organization, security rules
- **Size**: ~350 lines
- **Load Priority**: Required

### **01-AUTH.md** 🔐
- **Purpose**: Authentication & authorization patterns with NextAuth.js
- **When to Load**: When working with login, signup, password reset, role-based access
- **Contains**: NextAuth config, user service, route protection, RBAC, password management
- **Size**: ~577 lines
- **Load with**: Core only

### **02-PAYMENTS.md** 💳
- **Purpose**: Stripe payment integration patterns
- **When to Load**: When implementing subscriptions, billing, payment features
- **Contains**: Stripe setup, checkout sessions, webhooks, billing dashboard, feature gates
- **Size**: ~694 lines
- **Load with**: Core + Auth (for protected billing routes)

### **03-EMAIL.md** 📧
- **Purpose**: Email & notification system patterns
- **When to Load**: When implementing email sending, notifications, templates
- **Contains**: Resend integration, email templates, notification system, real-time alerts
- **Size**: ~580 lines
- **Load with**: Core (+ Auth for user emails)

### **04-UPLOADS.md** 📁
- **Purpose**: File upload & storage patterns
- **When to Load**: When implementing file uploads, image processing, storage
- **Contains**: S3 integration, presigned URLs, image processing, upload UI
- **Size**: ~670 lines
- **Load with**: Core + Auth (for user file ownership)

### **05-ADMIN.md** 👑
- **Purpose**: Admin panel & management patterns
- **When to Load**: When building admin dashboards, user management, analytics
- **Contains**: Admin routes, user management, system analytics, audit logs
- **Size**: ~620 lines
- **Load with**: Core + Auth (for admin roles)

### **06-LANDING.md** 🚀
- **Purpose**: Landing page & marketing site patterns
- **When to Load**: When building public-facing marketing pages, hero sections, pricing
- **Contains**: Hero sections, features, pricing, testimonials, FAQ, CTA patterns
- **Size**: ~680 lines
- **Load with**: Core only (public pages don't need auth)

---

## 🎯 **Loading Strategy**

### **For New Features** 
```
Always Load: 00-CORE.md
+ Load specific modules based on feature requirements
```

### **Common Combinations**
```typescript
// Landing Page Only
[00-CORE, 06-LANDING]

// Basic SaaS App
[00-CORE, 01-AUTH]

// SaaS with Payments  
[00-CORE, 01-AUTH, 02-PAYMENTS]

// SaaS with File Uploads
[00-CORE, 01-AUTH, 04-UPLOADS] 

// Complete SaaS with Marketing
[00-CORE, 06-LANDING, 01-AUTH, 02-PAYMENTS, 03-EMAIL, 04-UPLOADS, 05-ADMIN]
```

---

## 📊 **Context File Statistics**

| Module | Lines | Load Time | Dependencies |
|--------|-------|-----------|--------------|
| Core | ~350 | Fast | None |
| Auth | ~577 | Fast | Core |
| Payments | ~694 | Medium | Core, Auth |
| Email | ~580 | Fast | Core |
| Uploads | ~670 | Medium | Core, Auth |
| Admin | ~620 | Medium | Core, Auth |
| Landing | ~680 | Medium | Core |
| **Total** | ~4,171 | - | - |

---

## 🔧 **Usage Examples**

### **Building a Login Feature**
```bash
# Load these contexts:
- 00-CORE.md (architecture patterns)
- 01-AUTH.md (authentication patterns)
```

### **Adding Stripe Billing**
```bash  
# Load these contexts:
- 00-CORE.md (architecture patterns)
- 01-AUTH.md (user authentication)
- 02-PAYMENTS.md (Stripe integration)
```

### **Building Landing Page**
```bash
# Load these contexts:
- 00-CORE.md (architecture patterns)
- 06-LANDING.md (marketing page patterns)
```

### **Building Admin Dashboard**
```bash
# Load these contexts:
- 00-CORE.md (architecture patterns) 
- 01-AUTH.md (admin role protection)
- 05-ADMIN.md (admin UI patterns)
```

---

## 🧠 **AI Assistant Instructions**

### **Context Loading Rules**
1. **Always start with 00-CORE.md** - Contains essential patterns
2. **Load additional modules only when needed** - Avoid context bloat
3. **Check dependencies** - Some modules require others (e.g., Payments needs Auth)
4. **Unload unused contexts** - When switching tasks, only keep relevant contexts

### **Feature Development Workflow**
1. **Identify feature requirements** (auth, payments, uploads, etc.)
2. **Load 00-CORE.md + relevant modules**
3. **Follow patterns from loaded contexts**
4. **Test according to patterns in contexts**
5. **Update contexts if new patterns emerge**

### **Context Selection Guide**
- **Building API endpoints?** → Core only
- **Creating landing page?** → Core + Landing
- **Adding user registration?** → Core + Auth
- **Implementing subscriptions?** → Core + Auth + Payments  
- **Adding file uploads?** → Core + Auth + Uploads
- **Building admin panel?** → Core + Auth + Admin
- **Sending emails?** → Core + Email (+ Auth for user emails)

---

## 📝 **Maintenance Notes**

### **Adding New Modules**
1. Create new context file (06-FEATURE.md)
2. Follow existing format and patterns
3. Update this README with new module info
4. Keep modules focused and independent

### **Updating Existing Modules**
1. Maintain backward compatibility
2. Update version info in module header
3. Test patterns with AI assistant
4. Update README if new dependencies added

---

**This modular approach allows AI assistants to load only the knowledge they need, improving performance and reducing context confusion.**