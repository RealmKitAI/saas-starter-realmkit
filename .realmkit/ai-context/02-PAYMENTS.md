# Payments Module - AI Context

## Payment System Overview
This realm uses Stripe for payment processing with a subscription-based billing model. The system handles recurring subscriptions, usage-based billing, and one-time payments through a service-layer architecture.

## Key Files and Locations

### Configuration
- `src/lib/stripe/config.ts` - Stripe client configuration
- `src/lib/stripe/plans.ts` - Subscription plan definitions
- `.env.local` - Stripe API keys and webhook secrets

### API Routes
- `src/app/api/webhooks/stripe/route.ts` - Stripe webhook handler
- `src/app/api/billing/subscription/route.ts` - Subscription management
- `src/app/api/billing/portal/route.ts` - Customer portal access
- `src/app/api/billing/checkout/route.ts` - Checkout session creation

### Services
- `src/services/billing/billing.service.ts` - Main billing operations
- `src/services/billing/stripe.service.ts` - Stripe-specific operations
- `src/services/billing/subscription.service.ts` - Subscription management

### Pages
- `src/app/(dashboard)/dashboard/billing/page.tsx` - Billing dashboard
- `src/app/(marketing)/pricing/page.tsx` - Public pricing page

### Components
- `src/components/billing/SubscriptionCard.tsx` - Current subscription display
- `src/components/billing/PricingPlans.tsx` - Pricing plan selection
- `src/components/billing/BillingHistory.tsx` - Payment history
- `src/components/billing/PaymentMethod.tsx` - Payment method management

## Subscription Model

### Plan Structure
```typescript
// src/lib/stripe/plans.ts
export const plans = {
  free: {
    name: "Free",
    price: 0,
    interval: null,
    features: ["Basic features", "Limited usage"],
    stripePriceId: null
  },
  pro: {
    name: "Pro",
    price: 29,
    interval: "month",
    features: ["All features", "Priority support"],
    stripePriceId: "price_xxx"
  },
  enterprise: {
    name: "Enterprise", 
    price: 99,
    interval: "month",
    features: ["Custom features", "White-label"],
    stripePriceId: "price_yyy"
  }
}
```

### Database Schema
```prisma
model User {
  id                    String    @id @default(cuid())
  stripeCustomerId      String?   @unique
  subscriptionId        String?   @unique
  subscriptionStatus    String?   // active, past_due, canceled, etc.
  currentPeriodEnd      DateTime?
  plan                  String    @default("free")
}

model Invoice {
  id                String   @id @default(cuid())
  userId            String
  stripeInvoiceId   String   @unique
  amount            Int      // in cents
  status            String   // paid, open, void, etc.
  createdAt         DateTime @default(now())
  user              User     @relation(fields: [userId], references: [id])
}
```

## Common Payment Tasks

### 1. Creating a Checkout Session
```typescript
// services/billing/billing.service.ts
export const billingService = {
  async createCheckoutSession(userId: string, planId: string) {
    const user = await userService.findById(userId)
    const plan = plans[planId]
    
    // Create or get Stripe customer
    let customerId = user.stripeCustomerId
    if (!customerId) {
      const customer = await stripe.customers.create({
        email: user.email,
        metadata: { userId }
      })
      customerId = customer.id
      await userService.update(userId, { stripeCustomerId: customerId })
    }
    
    // Create checkout session
    const session = await stripe.checkout.sessions.create({
      customer: customerId,
      mode: 'subscription',
      payment_method_types: ['card'],
      line_items: [{
        price: plan.stripePriceId,
        quantity: 1
      }],
      success_url: `${process.env.NEXTAUTH_URL}/dashboard/billing?success=true`,
      cancel_url: `${process.env.NEXTAUTH_URL}/pricing?canceled=true`
    })
    
    return session.url
  }
}
```

### 2. Handling Stripe Webhooks
```typescript
// app/api/webhooks/stripe/route.ts
export async function POST(request: Request) {
  const body = await request.text()
  const signature = request.headers.get('stripe-signature')!
  
  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    return new Response('Webhook signature verification failed', { status: 400 })
  }
  
  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated':
      await handleSubscriptionChange(event.data.object)
      break
    case 'customer.subscription.deleted':
      await handleSubscriptionCanceled(event.data.object)
      break
    case 'invoice.payment_succeeded':
      await handlePaymentSucceeded(event.data.object)
      break
    case 'invoice.payment_failed':
      await handlePaymentFailed(event.data.object)
      break
  }
  
  return new Response('OK')
}
```

### 3. Subscription Status Checking
```typescript
// Middleware for checking subscription access
export async function requiresSubscription(userId: string) {
  const user = await userService.findById(userId)
  
  if (!user.subscriptionId || user.subscriptionStatus !== 'active') {
    throw new Error('Active subscription required')
  }
  
  // Check if subscription has expired
  if (user.currentPeriodEnd && new Date() > user.currentPeriodEnd) {
    throw new Error('Subscription has expired')
  }
  
  return true
}

// In API routes
export async function GET(request: Request) {
  const session = await auth()
  if (!session) return new Response('Unauthorized', { status: 401 })
  
  try {
    await requiresSubscription(session.user.id)
    // Handle premium feature request
  } catch (error) {
    return new Response('Subscription required', { status: 402 })
  }
}
```

### 4. Customer Portal Integration
```typescript
// services/billing/billing.service.ts
export const billingService = {
  async createPortalSession(userId: string) {
    const user = await userService.findById(userId)
    
    if (!user.stripeCustomerId) {
      throw new Error('No Stripe customer found')
    }
    
    const session = await stripe.billingPortal.sessions.create({
      customer: user.stripeCustomerId,
      return_url: `${process.env.NEXTAUTH_URL}/dashboard/billing`
    })
    
    return session.url
  }
}
```

### 5. Usage-Based Billing (Optional)
```typescript
// For metered billing
export const billingService = {
  async recordUsage(userId: string, usage: number) {
    const user = await userService.findById(userId)
    
    if (user.subscriptionId) {
      // Get the subscription item for usage tracking
      const subscription = await stripe.subscriptions.retrieve(user.subscriptionId)
      const usageItem = subscription.items.data.find(item => 
        item.price.recurring?.usage_type === 'metered'
      )
      
      if (usageItem) {
        await stripe.subscriptionItems.createUsageRecord(usageItem.id, {
          quantity: usage,
          timestamp: Math.floor(Date.now() / 1000)
        })
      }
    }
  }
}
```

## Pricing Page Implementation
```typescript
// components/billing/PricingPlans.tsx
'use client'
import { useState } from 'react'

export function PricingPlans() {
  const [isLoading, setIsLoading] = useState<string | null>(null)
  
  const handleUpgrade = async (planId: string) => {
    setIsLoading(planId)
    
    try {
      const response = await fetch('/api/billing/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ planId })
      })
      
      const { url } = await response.json()
      window.location.href = url
    } catch (error) {
      console.error('Checkout error:', error)
    } finally {
      setIsLoading(null)
    }
  }
  
  return (
    <div className="grid md:grid-cols-3 gap-8">
      {Object.entries(plans).map(([key, plan]) => (
        <div key={key} className="border rounded-lg p-6">
          <h3 className="text-xl font-bold">{plan.name}</h3>
          <p className="text-3xl font-bold">${plan.price}</p>
          <button 
            onClick={() => handleUpgrade(key)}
            disabled={isLoading === key}
            className="w-full mt-4 bg-blue-600 text-white px-4 py-2 rounded"
          >
            {isLoading === key ? 'Loading...' : 'Choose Plan'}
          </button>
        </div>
      ))}
    </div>
  )
}
```

## Security Best Practices
1. **Webhook Verification**: Always verify webhook signatures
2. **Idempotency**: Handle duplicate webhook events gracefully
3. **Server-Side Validation**: Never trust client-side payment data
4. **PCI Compliance**: Use Stripe Elements for card collection
5. **Customer Data**: Store minimal payment data locally

## Testing Payments
```typescript
// Use Stripe test cards
const testCards = {
  success: '4242424242424242',
  declined: '4000000000000002',
  requires3DS: '4000002500003155'
}

// Mock Stripe in tests
jest.mock('stripe', () => ({
  customers: {
    create: jest.fn().mockResolvedValue({ id: 'cus_test' })
  },
  checkout: {
    sessions: {
      create: jest.fn().mockResolvedValue({ url: 'https://checkout.stripe.com/test' })
    }
  }
}))
```

## Environment Variables
```env
# .env.local
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

## Common Issues & Solutions

### Webhook Endpoint Not Working
- Ensure webhook endpoint is accessible from internet
- Verify webhook secret matches Stripe dashboard
- Check that raw body is passed to verification

### Subscription Not Updating After Payment
- Verify webhook handlers are processing events correctly
- Check database updates in webhook handlers
- Ensure user lookup is working properly

### Test Mode vs Live Mode
- Use different API keys for test/live
- Test webhooks with Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`