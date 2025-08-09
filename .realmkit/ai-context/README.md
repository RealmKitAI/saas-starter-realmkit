# AI Context Modules - Loading Guide

## Overview
This directory contains modular AI context files that help AI coding assistants understand and work with this realm effectively. Load only the modules relevant to your current task to optimize context usage.

## Core Module (Always Load)
- **00-CORE.md** - Essential architecture, patterns, and conventions
  - Project structure and philosophy
  - Technology stack overview
  - Development workflow
  - Key conventions and patterns

## Feature Modules (Load As Needed)

### 🔐 01-AUTH.md - Authentication System
Load when working on:
- User login/logout functionality
- Registration flows
- Password reset
- OAuth provider integration
- Protected routes and middleware
- Session management
- Role-based access control

### 💳 02-PAYMENTS.md - Payment Processing
Load when working on:
- Stripe integration
- Subscription management
- Payment webhooks
- Billing pages
- Invoice generation
- Payment method management
- Pricing plans

### 📧 03-EMAIL.md - Email System
Load when working on:
- Transactional emails
- Email templates
- Email service configuration
- Welcome emails
- Password reset emails
- Notification emails

### 👨‍💼 04-ADMIN.md - Admin Dashboard
Load when working on:
- Admin panel features
- User management
- System settings
- Analytics dashboards
- Content management
- Audit logs

### 🎨 05-LANDING.md - Marketing Pages
Load when working on:
- Homepage
- Pricing page
- About page
- SEO optimization
- Marketing components
- Conversion optimization

## Loading Strategy

### For General Development
```
Load: 00-CORE.md
```

### For Authentication Work
```
Load: 00-CORE.md + 01-AUTH.md
```

### For Full-Stack Feature Development
```
Load: 00-CORE.md + relevant feature modules
```

### For Complex Features Spanning Multiple Areas
```
Example: Adding team billing feature
Load: 00-CORE.md + 01-AUTH.md + 02-PAYMENTS.md + 04-ADMIN.md
```

## Module Structure
Each module follows this structure:
1. **Overview** - High-level description of the feature
2. **Key Files** - Important files and their locations
3. **Common Tasks** - Step-by-step guides for frequent operations
4. **Patterns** - Feature-specific patterns and conventions
5. **Troubleshooting** - Common issues and solutions

## Best Practices for AI Assistants
1. Always load 00-CORE.md first
2. Load only necessary feature modules
3. Check module overview before diving into details
4. Follow established patterns from the modules
5. Refer to examples in the modules
6. Update modules when adding new patterns

## Creating New Modules
When adding new features that warrant their own module:
1. Create `XX-FEATURE.md` with next number
2. Follow the existing module structure
3. Document key files and patterns
4. Add common task examples
5. Update this README

## Module Dependencies
Some modules depend on others:
- PAYMENTS depends on AUTH (user must be authenticated)
- ADMIN depends on AUTH (role-based access)
- EMAIL is standalone but often used with AUTH

Load dependent modules together for complete context.