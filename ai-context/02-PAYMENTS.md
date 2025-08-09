# Payments Module - AI Context

## 💳 **Stripe Payment Integration**

Load this context **ONLY** when working with payment features. Not all SaaS apps need payments.

---

## 🏗️ **Stripe Architecture Overview**

### **Payment Flow**
```
User → Checkout Session → Stripe → Webhook → Database → Feature Access
```

### **Key Components**
- **Stripe Client**: For creating checkout sessions
- **Webhook Handler**: For processing payment events
- **Billing Service**: Business logic for subscriptions
- **Subscription Model**: Database tracking

---

## 💰 **Stripe Configuration**

### **Stripe Client Setup**
```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
  typescript: true,
});

// Plan configuration
export const PLANS = {
  FREE: {
    name: 'Free',
    price: 0,
    features: ['5 projects', 'Basic support'],
    limits: {
      projects: 5,
      storage: '1GB',
    },
  },
  PRO: {
    name: 'Pro',
    price: 2900, // $29.00 in cents
    priceId: process.env.STRIPE_PRO_PRICE_ID!,
    features: ['Unlimited projects', 'Priority support', 'Advanced analytics'],
    limits: {
      projects: -1, // unlimited
      storage: '100GB',
    },
  },
  ENTERPRISE: {
    name: 'Enterprise',
    price: 9900, // $99.00 in cents
    priceId: process.env.STRIPE_ENTERPRISE_PRICE_ID!,
    features: ['Everything in Pro', 'Custom integrations', 'Dedicated support'],
    limits: {
      projects: -1,
      storage: '1TB',
    },
  },
} as const;

export type PlanKey = keyof typeof PLANS;
```

---

## 🔄 **Billing Service Patterns**

### **Subscription Management**
```typescript
// services/billing.service.ts
export class BillingService {
  /**
   * Creates Stripe checkout session for subscription
   */
  static async createCheckoutSession({
    userId,
    planKey,
    successUrl,
    cancelUrl,
  }: {
    userId: string;
    planKey: PlanKey;
    successUrl: string;
    cancelUrl: string;
  }): Promise<{ url: string }> {
    // Get user details
    const user = await UserService.getById(userId);
    if (!user) {
      throw new Error('User not found');
    }

    const plan = PLANS[planKey];
    if (!plan.priceId) {
      throw new Error('Invalid plan for checkout');
    }

    // Create or get Stripe customer
    let customerId = user.stripeCustomerId;
    if (!customerId) {
      const customer = await stripe.customers.create({
        email: user.email,
        name: user.name || undefined,
        metadata: {
          userId,
        },
      });
      
      customerId = customer.id;
      
      // Update user with customer ID
      await UserService.updateStripeCustomerId(userId, customerId);
    }

    // Create checkout session
    const session = await stripe.checkout.sessions.create({
      customer: customerId,
      payment_method_types: ['card'],
      billing_address_collection: 'required',
      line_items: [
        {
          price: plan.priceId,
          quantity: 1,
        },
      ],
      mode: 'subscription',
      success_url: successUrl,
      cancel_url: cancelUrl,
      metadata: {
        userId,
        planKey,
      },
    });

    if (!session.url) {
      throw new Error('Failed to create checkout session');
    }

    return { url: session.url };
  }

  /**
   * Gets current user subscription
   */
  static async getUserSubscription(userId: string): Promise<UserSubscription | null> {
    const subscription = await db.subscription.findFirst({
      where: {
        userId,
        status: 'ACTIVE',
      },
      orderBy: {
        createdAt: 'desc',
      },
    });

    if (!subscription) return null;

    return {
      id: subscription.id,
      plan: subscription.plan,
      status: subscription.status,
      currentPeriodEnd: subscription.currentPeriodEnd,
      cancelAtPeriodEnd: subscription.cancelAtPeriodEnd,
      features: PLANS[subscription.plan as PlanKey]?.features || [],
      limits: PLANS[subscription.plan as PlanKey]?.limits || {},
    };
  }

  /**
   * Creates customer portal session for subscription management
   */
  static async createPortalSession(
    userId: string,
    returnUrl: string
  ): Promise<{ url: string }> {
    const user = await UserService.getById(userId);
    if (!user?.stripeCustomerId) {
      throw new Error('No Stripe customer found');
    }

    const session = await stripe.billingPortal.sessions.create({
      customer: user.stripeCustomerId,
      return_url: returnUrl,
    });

    return { url: session.url };
  }
}
```

---

## 🎣 **Webhook Handling**

### **Webhook Route Pattern**
```typescript
// app/api/billing/webhook/route.ts
import { stripe } from '@/lib/stripe';
import { BillingService } from '@/services/billing.service';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature');

  if (!signature) {
    return new Response('Missing signature', { status: 400 });
  }

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

      case 'invoice.payment_failed':
        await BillingService.handlePaymentFailed(event.data.object);
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

### **Webhook Event Handlers**
```typescript
// services/billing.service.ts (continued)
export class BillingService {
  /**
   * Handles successful checkout completion
   */
  static async handleCheckoutCompleted(
    session: Stripe.Checkout.Session
  ): Promise<void> {
    const userId = session.metadata?.userId;
    const planKey = session.metadata?.planKey as PlanKey;

    if (!userId || !planKey) {
      throw new Error('Missing metadata in checkout session');
    }

    // Get the subscription from Stripe
    const stripeSubscription = await stripe.subscriptions.retrieve(
      session.subscription as string
    );

    // Create subscription record
    await db.subscription.create({
      data: {
        userId,
        plan: planKey,
        status: 'ACTIVE',
        stripeSubscriptionId: stripeSubscription.id,
        stripePriceId: stripeSubscription.items.data[0].price.id,
        currentPeriodStart: new Date(stripeSubscription.current_period_start * 1000),
        currentPeriodEnd: new Date(stripeSubscription.current_period_end * 1000),
      },
    });

    // Send welcome email
    await EmailService.sendSubscriptionWelcome(userId, planKey);
  }

  /**
   * Handles subscription updates
   */
  static async handleSubscriptionUpdated(
    subscription: Stripe.Subscription
  ): Promise<void> {
    await db.subscription.updateMany({
      where: {
        stripeSubscriptionId: subscription.id,
      },
      data: {
        status: subscription.status.toUpperCase() as SubscriptionStatus,
        currentPeriodStart: new Date(subscription.current_period_start * 1000),
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
        cancelAtPeriodEnd: subscription.cancel_at_period_end,
      },
    });
  }

  /**
   * Handles subscription cancellation
   */
  static async handleSubscriptionCanceled(
    subscription: Stripe.Subscription
  ): Promise<void> {
    await db.subscription.updateMany({
      where: {
        stripeSubscriptionId: subscription.id,
      },
      data: {
        status: 'CANCELED',
        canceledAt: new Date(),
      },
    });

    // Send cancellation email
    const dbSubscription = await db.subscription.findFirst({
      where: { stripeSubscriptionId: subscription.id },
      include: { user: true },
    });

    if (dbSubscription) {
      await EmailService.sendSubscriptionCanceled(dbSubscription.userId);
    }
  }
}
```

---

## 🎨 **Payment UI Components**

### **Pricing Plans Component**
```typescript
// components/pricing/pricing-plans.tsx
'use client';

interface PricingPlansProps {
  currentPlan?: PlanKey;
  onSelectPlan: (planKey: PlanKey) => void;
}

export function PricingPlans({ currentPlan, onSelectPlan }: PricingPlansProps) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
      {Object.entries(PLANS).map(([key, plan]) => (
        <div
          key={key}
          className={cn(
            "relative p-6 rounded-lg border-2",
            currentPlan === key
              ? "border-blue-500 bg-blue-50"
              : "border-gray-200"
          )}
        >
          {currentPlan === key && (
            <div className="absolute -top-3 left-1/2 transform -translate-x-1/2">
              <span className="bg-blue-500 text-white px-3 py-1 rounded-full text-sm">
                Current Plan
              </span>
            </div>
          )}

          <div className="text-center">
            <h3 className="text-xl font-bold">{plan.name}</h3>
            <div className="mt-2">
              <span className="text-4xl font-bold">
                ${plan.price / 100}
              </span>
              {plan.price > 0 && (
                <span className="text-gray-500">/month</span>
              )}
            </div>
          </div>

          <ul className="mt-6 space-y-3">
            {plan.features.map((feature, index) => (
              <li key={index} className="flex items-center">
                <CheckIcon className="h-5 w-5 text-green-500 mr-3" />
                {feature}
              </li>
            ))}
          </ul>

          <Button
            className="w-full mt-8"
            onClick={() => onSelectPlan(key as PlanKey)}
            disabled={currentPlan === key}
            variant={currentPlan === key ? "outline" : "default"}
          >
            {currentPlan === key ? 'Current Plan' : `Choose ${plan.name}`}
          </Button>
        </div>
      ))}
    </div>
  );
}
```

### **Billing Dashboard Component**
```typescript
// components/billing/billing-dashboard.tsx
'use client';

interface BillingDashboardProps {
  subscription: UserSubscription | null;
  onUpgrade: (planKey: PlanKey) => void;
  onManageSubscription: () => void;
}

export function BillingDashboard({
  subscription,
  onUpgrade,
  onManageSubscription,
}: BillingDashboardProps) {
  if (!subscription || subscription.plan === 'FREE') {
    return (
      <div className="p-6 bg-gray-50 rounded-lg">
        <h3 className="text-lg font-semibold mb-4">Free Plan</h3>
        <p className="text-gray-600 mb-4">
          You're currently on the free plan. Upgrade to unlock more features.
        </p>
        <Button onClick={() => onUpgrade('PRO')}>
          Upgrade to Pro
        </Button>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="p-6 bg-blue-50 rounded-lg">
        <div className="flex justify-between items-start">
          <div>
            <h3 className="text-lg font-semibold">
              {PLANS[subscription.plan as PlanKey]?.name} Plan
            </h3>
            <p className="text-gray-600">
              ${PLANS[subscription.plan as PlanKey]?.price / 100}/month
            </p>
          </div>
          <Badge
            variant={subscription.status === 'ACTIVE' ? 'default' : 'secondary'}
          >
            {subscription.status}
          </Badge>
        </div>

        <div className="mt-4">
          <p className="text-sm text-gray-600">
            {subscription.cancelAtPeriodEnd ? (
              <>Cancels on {format(subscription.currentPeriodEnd, 'PP')}</>
            ) : (
              <>Renews on {format(subscription.currentPeriodEnd, 'PP')}</>
            )}
          </p>
        </div>
      </div>

      <div className="p-6 border rounded-lg">
        <h4 className="font-semibold mb-3">Current Features</h4>
        <ul className="space-y-2">
          {subscription.features.map((feature, index) => (
            <li key={index} className="flex items-center">
              <CheckIcon className="h-4 w-4 text-green-500 mr-2" />
              {feature}
            </li>
          ))}
        </ul>
      </div>

      <div className="flex gap-4">
        <Button variant="outline" onClick={onManageSubscription}>
          Manage Subscription
        </Button>
        {subscription.plan === 'PRO' && (
          <Button onClick={() => onUpgrade('ENTERPRISE')}>
            Upgrade to Enterprise
          </Button>
        )}
      </div>
    </div>
  );
}
```

---

## 🔒 **Feature Access Control**

### **Subscription Middleware**
```typescript
// lib/subscription-guard.ts
export async function checkFeatureAccess(
  userId: string,
  requiredPlan: PlanKey
): Promise<boolean> {
  const subscription = await BillingService.getUserSubscription(userId);
  
  if (!subscription) return false;
  
  const planHierarchy = ['FREE', 'PRO', 'ENTERPRISE'];
  const userPlanIndex = planHierarchy.indexOf(subscription.plan);
  const requiredPlanIndex = planHierarchy.indexOf(requiredPlan);
  
  return userPlanIndex >= requiredPlanIndex;
}

export function requirePlan(planKey: PlanKey) {
  return async function(request: NextRequest) {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const hasAccess = await checkFeatureAccess(session.user.id, planKey);
    if (!hasAccess) {
      return NextResponse.json(
        { error: 'Subscription required', requiredPlan: planKey },
        { status: 402 }
      );
    }

    return null; // Access granted
  };
}
```

### **Feature Gate Hook**
```typescript
// hooks/use-feature-access.ts
export function useFeatureAccess() {
  const { data: session } = useSession();
  const [subscription, setSubscription] = useState<UserSubscription | null>(null);

  useEffect(() => {
    if (session?.user) {
      BillingService.getUserSubscription(session.user.id)
        .then(setSubscription);
    }
  }, [session]);

  const hasFeature = useCallback((requiredPlan: PlanKey): boolean => {
    if (!subscription) return false;
    
    const planHierarchy = ['FREE', 'PRO', 'ENTERPRISE'];
    const userPlanIndex = planHierarchy.indexOf(subscription.plan);
    const requiredPlanIndex = planHierarchy.indexOf(requiredPlan);
    
    return userPlanIndex >= requiredPlanIndex;
  }, [subscription]);

  return {
    subscription,
    hasFeature,
    isPro: hasFeature('PRO'),
    isEnterprise: hasFeature('ENTERPRISE'),
  };
}
```

---

## 🧪 **Payment Testing**

### **Billing Service Testing**
```typescript
// services/__tests__/billing.service.test.ts
describe('BillingService', () => {
  describe('createCheckoutSession', () => {
    it('should create checkout session for existing customer', async () => {
      const mockUser = {
        id: 'user_123',
        email: 'test@example.com',
        stripeCustomerId: 'cus_123',
      };

      mockUserService.getById.mockResolvedValue(mockUser);
      mockStripe.checkout.sessions.create.mockResolvedValue({
        id: 'cs_123',
        url: 'https://checkout.stripe.com/pay/cs_123',
      });

      const result = await BillingService.createCheckoutSession({
        userId: 'user_123',
        planKey: 'PRO',
        successUrl: 'https://app.com/success',
        cancelUrl: 'https://app.com/cancel',
      });

      expect(result.url).toBe('https://checkout.stripe.com/pay/cs_123');
      expect(mockStripe.checkout.sessions.create).toHaveBeenCalledWith({
        customer: 'cus_123',
        payment_method_types: ['card'],
        line_items: [{ price: PLANS.PRO.priceId, quantity: 1 }],
        mode: 'subscription',
        success_url: 'https://app.com/success',
        cancel_url: 'https://app.com/cancel',
        metadata: { userId: 'user_123', planKey: 'PRO' },
      });
    });
  });
});
```

### **Webhook Testing**
```typescript
// tests/integration/webhooks.test.ts
describe('/api/billing/webhook', () => {
  it('should handle checkout.session.completed', async () => {
    const mockEvent = {
      type: 'checkout.session.completed',
      data: {
        object: {
          id: 'cs_123',
          subscription: 'sub_123',
          metadata: {
            userId: 'user_123',
            planKey: 'PRO',
          },
        },
      },
    };

    mockStripe.webhooks.constructEvent.mockReturnValue(mockEvent);
    mockStripe.subscriptions.retrieve.mockResolvedValue({
      id: 'sub_123',
      current_period_start: 1640995200,
      current_period_end: 1643673600,
      items: { data: [{ price: { id: 'price_123' } }] },
    });

    const response = await request(app)
      .post('/api/billing/webhook')
      .set('stripe-signature', 'valid_signature')
      .send(JSON.stringify(mockEvent));

    expect(response.status).toBe(200);
    expect(mockDb.subscription.create).toHaveBeenCalledWith({
      data: {
        userId: 'user_123',
        plan: 'PRO',
        status: 'ACTIVE',
        stripeSubscriptionId: 'sub_123',
        stripePriceId: 'price_123',
        currentPeriodStart: expect.any(Date),
        currentPeriodEnd: expect.any(Date),
      },
    });
  });
});
```

---

## 📋 **Stripe Integration Checklist**

When implementing Stripe payments:
- [ ] Set up Stripe client with proper API version
- [ ] Define plans with prices and features
- [ ] Implement checkout session creation
- [ ] Set up webhook endpoint with signature verification
- [ ] Handle all relevant webhook events
- [ ] Create subscription management UI
- [ ] Implement feature access control
- [ ] Add customer portal integration
- [ ] Test with Stripe test cards
- [ ] Write comprehensive tests
- [ ] Set up proper error handling
- [ ] Configure webhook endpoints in Stripe dashboard

---

**This payment context provides complete Stripe integration patterns. Only load when payment features are required.**