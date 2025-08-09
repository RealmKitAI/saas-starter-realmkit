# Email Module - AI Context

## Email System Overview
This realm uses Resend for transactional email delivery with a template-based system. The email service handles user notifications, authentication emails, billing updates, and marketing communications through a clean service layer.

## Key Files and Locations

### Configuration
- `src/lib/email/config.ts` - Resend client configuration
- `src/lib/email/templates/` - Email template definitions
- `.env.local` - Resend API key and email settings

### Services
- `src/services/email/email.service.ts` - Main email operations
- `src/services/email/templates.service.ts` - Template rendering
- `src/services/email/newsletter.service.ts` - Newsletter/marketing emails

### Templates
- `src/lib/email/templates/welcome.tsx` - Welcome email template
- `src/lib/email/templates/password-reset.tsx` - Password reset email
- `src/lib/email/templates/billing-update.tsx` - Billing notifications
- `src/lib/email/templates/base.tsx` - Base template layout

### API Routes
- `src/app/api/email/send/route.ts` - Manual email sending
- `src/app/api/email/newsletter/route.ts` - Newsletter subscription

## Email Template System

### Base Template Structure
```tsx
// src/lib/email/templates/base.tsx
import { Html, Head, Body, Container, Section, Text, Button } from '@react-email/components'

interface BaseTemplateProps {
  title: string
  children: React.ReactNode
}

export function BaseTemplate({ title, children }: BaseTemplateProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'sans-serif', backgroundColor: '#f6f9fc' }}>
        <Container style={{ margin: '0 auto', padding: '20px' }}>
          <Section style={{ backgroundColor: 'white', padding: '40px', borderRadius: '8px' }}>
            <img src="https://your-domain.com/logo.png" alt="Your SaaS" width="120" />
            <Text style={{ fontSize: '24px', fontWeight: 'bold', marginBottom: '20px' }}>
              {title}
            </Text>
            {children}
            <Text style={{ fontSize: '14px', color: '#666', marginTop: '40px' }}>
              © 2025 Your SaaS. All rights reserved.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}
```

### Welcome Email Template
```tsx
// src/lib/email/templates/welcome.tsx
import { BaseTemplate } from './base'
import { Text, Button } from '@react-email/components'

interface WelcomeEmailProps {
  userEmail: string
  userName?: string
}

export function WelcomeEmail({ userEmail, userName }: WelcomeEmailProps) {
  return (
    <BaseTemplate title="Welcome to Your SaaS!">
      <Text style={{ fontSize: '16px', lineHeight: '24px' }}>
        Hi {userName || userEmail},
      </Text>
      <Text style={{ fontSize: '16px', lineHeight: '24px' }}>
        Welcome to Your SaaS! We're excited to have you on board. Your account has been 
        successfully created and you can now access all our features.
      </Text>
      <Button
        href={`${process.env.NEXTAUTH_URL}/dashboard`}
        style={{
          backgroundColor: '#007ee6',
          color: 'white',
          padding: '12px 20px',
          borderRadius: '4px',
          textDecoration: 'none'
        }}
      >
        Get Started
      </Button>
      <Text style={{ fontSize: '14px', color: '#666', marginTop: '20px' }}>
        If you have any questions, feel free to reach out to our support team.
      </Text>
    </BaseTemplate>
  )
}
```

## Email Service Implementation

### Core Email Service
```typescript
// src/services/email/email.service.ts
import { Resend } from 'resend'
import { render } from '@react-email/render'

const resend = new Resend(process.env.RESEND_API_KEY)

export const emailService = {
  async sendWelcomeEmail(userEmail: string, userName?: string) {
    try {
      const emailHtml = render(WelcomeEmail({ userEmail, userName }))
      
      const result = await resend.emails.send({
        from: process.env.EMAIL_FROM || 'noreply@yoursaas.com',
        to: userEmail,
        subject: 'Welcome to Your SaaS!',
        html: emailHtml,
        headers: {
          'X-Entity-Ref-ID': `welcome-${Date.now()}`
        }
      })
      
      console.log('Welcome email sent:', result.data?.id)
      return result
    } catch (error) {
      console.error('Failed to send welcome email:', error)
      throw error
    }
  },

  async sendPasswordResetEmail(userEmail: string, resetToken: string) {
    try {
      const resetUrl = `${process.env.NEXTAUTH_URL}/reset-password?token=${resetToken}`
      const emailHtml = render(PasswordResetEmail({ resetUrl, userEmail }))
      
      const result = await resend.emails.send({
        from: process.env.EMAIL_FROM || 'noreply@yoursaas.com',
        to: userEmail,
        subject: 'Reset Your Password',
        html: emailHtml,
        headers: {
          'X-Entity-Ref-ID': `reset-${resetToken}`
        }
      })
      
      return result
    } catch (error) {
      console.error('Failed to send password reset email:', error)
      throw error
    }
  },

  async sendBillingUpdateEmail(
    userEmail: string, 
    updateType: 'subscription_created' | 'payment_succeeded' | 'payment_failed',
    details: any
  ) {
    try {
      const emailHtml = render(BillingUpdateEmail({ updateType, details, userEmail }))
      
      const subjectMap = {
        subscription_created: 'Subscription Activated',
        payment_succeeded: 'Payment Successful',
        payment_failed: 'Payment Failed'
      }
      
      const result = await resend.emails.send({
        from: process.env.EMAIL_FROM || 'noreply@yoursaas.com',
        to: userEmail,
        subject: subjectMap[updateType],
        html: emailHtml,
        headers: {
          'X-Entity-Ref-ID': `billing-${updateType}-${Date.now()}`
        }
      })
      
      return result
    } catch (error) {
      console.error('Failed to send billing update email:', error)
      throw error
    }
  },

  async sendNotificationEmail(
    userEmail: string,
    subject: string,
    content: string,
    actionButton?: { text: string; url: string }
  ) {
    try {
      const emailHtml = render(
        NotificationEmail({ content, actionButton, userEmail })
      )
      
      const result = await resend.emails.send({
        from: process.env.EMAIL_FROM || 'noreply@yoursaas.com',
        to: userEmail,
        subject,
        html: emailHtml
      })
      
      return result
    } catch (error) {
      console.error('Failed to send notification email:', error)
      throw error
    }
  }
}
```

## Common Email Tasks

### 1. Sending Email on User Registration
```typescript
// In auth service after user creation
import { emailService } from '@/services/email/email.service'

export const authService = {
  async register(data: RegisterInput) {
    // Create user
    const user = await db.user.create({
      data: {
        email: data.email,
        password: hashedPassword,
        name: data.name
      }
    })
    
    // Send welcome email (async, don't await)
    emailService.sendWelcomeEmail(user.email, user.name)
      .catch(error => console.error('Welcome email failed:', error))
    
    return user
  }
}
```

### 2. Password Reset Email Flow
```typescript
// services/auth/auth.service.ts
export const authService = {
  async requestPasswordReset(email: string) {
    const user = await userService.findByEmail(email)
    if (!user) {
      // Don't reveal if user exists
      return { success: true }
    }
    
    // Generate reset token
    const resetToken = crypto.randomUUID()
    const expiresAt = new Date(Date.now() + 1000 * 60 * 60) // 1 hour
    
    // Store token
    await db.passwordResetToken.create({
      data: {
        token: resetToken,
        userId: user.id,
        expiresAt
      }
    })
    
    // Send reset email
    await emailService.sendPasswordResetEmail(user.email, resetToken)
    
    return { success: true }
  }
}
```

### 3. Billing Email Integration
```typescript
// In Stripe webhook handler
async function handlePaymentSucceeded(invoice: Stripe.Invoice) {
  const user = await userService.findByStripeCustomerId(invoice.customer as string)
  
  if (user) {
    await emailService.sendBillingUpdateEmail(
      user.email,
      'payment_succeeded',
      {
        amount: invoice.amount_paid / 100,
        currency: invoice.currency,
        period: {
          start: new Date(invoice.period_start * 1000),
          end: new Date(invoice.period_end * 1000)
        }
      }
    )
  }
}
```

### 4. Bulk Email/Newsletter System
```typescript
// services/email/newsletter.service.ts
export const newsletterService = {
  async sendNewsletter(subject: string, template: string, data: any) {
    // Get all subscribed users
    const subscribers = await db.user.findMany({
      where: { newsletterSubscribed: true },
      select: { email: true, name: true }
    })
    
    // Send in batches to avoid rate limits
    const batchSize = 100
    for (let i = 0; i < subscribers.length; i += batchSize) {
      const batch = subscribers.slice(i, i + batchSize)
      
      await Promise.all(
        batch.map(subscriber =>
          emailService.sendNewsletterEmail(
            subscriber.email,
            subject,
            template,
            { ...data, userName: subscriber.name }
          )
        )
      )
      
      // Wait between batches
      if (i + batchSize < subscribers.length) {
        await new Promise(resolve => setTimeout(resolve, 1000))
      }
    }
  }
}
```

### 5. Email Template Testing
```typescript
// For development - preview emails
// app/api/email/preview/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const template = searchParams.get('template')
  
  let emailHtml: string
  
  switch (template) {
    case 'welcome':
      emailHtml = render(WelcomeEmail({ 
        userEmail: 'test@example.com', 
        userName: 'Test User' 
      }))
      break
    case 'password-reset':
      emailHtml = render(PasswordResetEmail({ 
        resetUrl: 'https://example.com/reset?token=123',
        userEmail: 'test@example.com'
      }))
      break
    default:
      return new Response('Template not found', { status: 404 })
  }
  
  return new Response(emailHtml, {
    headers: { 'Content-Type': 'text/html' }
  })
}
```

## Email Configuration

### Environment Variables
```env
# .env.local
RESEND_API_KEY=re_xxxxxxxxxxxxx
EMAIL_FROM=noreply@yourdomain.com
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

### Domain Setup
1. Add your domain to Resend dashboard
2. Configure DNS records for domain verification
3. Set up SPF, DKIM, and DMARC records for deliverability

## Email Best Practices

### Deliverability
- Use verified domains
- Include proper unsubscribe links
- Maintain good sender reputation
- Avoid spammy content

### User Experience
- Clear subject lines
- Mobile-responsive templates
- Consistent branding
- Quick loading images

### Performance
- Send emails asynchronously
- Use background jobs for bulk emails
- Implement retry logic for failures
- Monitor email delivery rates

## Database Schema for Email Tracking
```prisma
model EmailLog {
  id          String   @id @default(cuid())
  userId      String?
  email       String
  subject     String
  template    String
  status      String   // sent, failed, delivered, opened
  resendId    String?  // Resend message ID
  error       String?
  createdAt   DateTime @default(now())
  user        User?    @relation(fields: [userId], references: [id])
}

model NewsletterSubscription {
  id          String   @id @default(cuid())
  email       String   @unique
  subscribed  Boolean  @default(true)
  userId      String?
  createdAt   DateTime @default(now())
  user        User?    @relation(fields: [userId], references: [id])
}
```

## Troubleshooting Common Issues

### Emails Not Sending
- Check RESEND_API_KEY is set correctly
- Verify domain is verified in Resend dashboard
- Check rate limits (Resend has daily limits)

### Emails Going to Spam
- Set up proper DNS records (SPF, DKIM, DMARC)
- Use verified sender domain
- Avoid spam trigger words
- Include proper unsubscribe links

### Template Rendering Issues
- Ensure all props are passed to templates
- Check for undefined variables in templates
- Test templates in development preview endpoint