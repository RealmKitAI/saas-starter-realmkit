# Landing Page Module - AI Context

## Marketing System Overview
This realm includes a complete marketing website with landing pages optimized for conversion, SEO, and user acquisition. The marketing system uses Server Components for performance and includes analytics tracking, A/B testing capabilities, and conversion optimization.

## Key Files and Locations

### Marketing Pages
- `src/app/(marketing)/page.tsx` - Homepage/landing page
- `src/app/(marketing)/pricing/page.tsx` - Pricing page
- `src/app/(marketing)/about/page.tsx` - About page
- `src/app/(marketing)/features/page.tsx` - Features overview
- `src/app/(marketing)/blog/page.tsx` - Blog listing
- `src/app/(marketing)/contact/page.tsx` - Contact page

### Components
- `src/components/marketing/Hero.tsx` - Hero section
- `src/components/marketing/Features.tsx` - Feature showcase
- `src/components/marketing/Pricing.tsx` - Pricing cards
- `src/components/marketing/Testimonials.tsx` - Customer testimonials
- `src/components/marketing/FAQ.tsx` - Frequently asked questions
- `src/components/marketing/CTA.tsx` - Call-to-action sections
- `src/components/marketing/Newsletter.tsx` - Newsletter signup

### Layout & Navigation
- `src/app/(marketing)/layout.tsx` - Marketing layout
- `src/components/marketing/Header.tsx` - Marketing header
- `src/components/marketing/Footer.tsx` - Marketing footer
- `src/components/marketing/Navigation.tsx` - Main navigation

### SEO & Analytics
- `src/lib/seo/metadata.ts` - SEO metadata configuration
- `src/lib/analytics/tracking.ts` - Analytics tracking setup
- `src/components/marketing/Analytics.tsx` - Analytics components

## Landing Page Structure

### Hero Section Component
```tsx
// src/components/marketing/Hero.tsx
import { Button } from '@/components/ui/button'
import { ArrowRight, Play } from 'lucide-react'

export function Hero() {
  return (
    <section className="relative overflow-hidden bg-gradient-to-br from-blue-50 to-indigo-100 py-20 sm:py-32">
      <div className="container mx-auto px-4">
        <div className="mx-auto max-w-4xl text-center">
          {/* Badge */}
          <div className="mb-8 inline-flex items-center rounded-full bg-blue-100 px-4 py-2 text-sm text-blue-700">
            <span className="mr-2">🚀</span>
            New: Advanced analytics dashboard now available
          </div>
          
          {/* Main Headline */}
          <h1 className="mb-6 text-4xl font-bold tracking-tight text-gray-900 sm:text-6xl">
            Build your SaaS
            <span className="text-blue-600"> faster than ever</span>
          </h1>
          
          {/* Subheadline */}
          <p className="mb-8 text-xl text-gray-600 sm:text-2xl">
            The complete toolkit for launching your SaaS product. 
            Authentication, payments, analytics, and more - all pre-built and ready to customize.
          </p>
          
          {/* CTA Buttons */}
          <div className="flex flex-col gap-4 sm:flex-row sm:justify-center">
            <Button size="lg" className="group">
              Start Building Now
              <ArrowRight className="ml-2 h-4 w-4 transition-transform group-hover:translate-x-1" />
            </Button>
            <Button size="lg" variant="outline" className="group">
              <Play className="mr-2 h-4 w-4" />
              Watch Demo
            </Button>
          </div>
          
          {/* Social Proof */}
          <div className="mt-12 text-sm text-gray-500">
            Trusted by 1,000+ developers worldwide
          </div>
        </div>
        
        {/* Hero Image/Demo */}
        <div className="mt-16 relative">
          <div className="mx-auto max-w-4xl">
            <img
              src="/dashboard-preview.png"
              alt="Dashboard Preview"
              className="rounded-xl shadow-2xl"
            />
          </div>
        </div>
      </div>
    </section>
  )
}
```

### Features Section
```tsx
// src/components/marketing/Features.tsx
import { 
  Shield, 
  CreditCard, 
  Mail, 
  BarChart3, 
  Users, 
  Zap 
} from 'lucide-react'

const features = [
  {
    icon: Shield,
    title: 'Authentication Ready',
    description: 'Complete auth system with social logins, email verification, and role-based access control.'
  },
  {
    icon: CreditCard,
    title: 'Stripe Integration',
    description: 'Full payment processing with subscriptions, billing portal, and webhook handling.'
  },
  {
    icon: Mail,
    title: 'Email System',
    description: 'Transactional emails, templates, and newsletter functionality built-in.'
  },
  {
    icon: BarChart3,
    title: 'Analytics Dashboard',
    description: 'Track user behavior, revenue metrics, and business KPIs out of the box.'
  },
  {
    icon: Users,
    title: 'Admin Panel',
    description: 'Comprehensive admin interface for user management and system settings.'
  },
  {
    icon: Zap,
    title: 'Performance Optimized',
    description: 'Built with Next.js 14, TypeScript, and modern best practices for speed.'
  }
]

export function Features() {
  return (
    <section className="py-20 bg-white">
      <div className="container mx-auto px-4">
        <div className="mx-auto max-w-3xl text-center mb-16">
          <h2 className="text-3xl font-bold text-gray-900 sm:text-4xl">
            Everything you need to launch
          </h2>
          <p className="mt-4 text-lg text-gray-600">
            Skip months of development with our production-ready components and services.
          </p>
        </div>
        
        <div className="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
          {features.map((feature, index) => (
            <div key={index} className="text-center">
              <div className="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-lg bg-blue-100">
                <feature.icon className="h-6 w-6 text-blue-600" />
              </div>
              <h3 className="mb-2 text-xl font-semibold text-gray-900">
                {feature.title}
              </h3>
              <p className="text-gray-600">
                {feature.description}
              </p>
            </div>
          ))}
        </div>
      </div>
    </section>
  )
}
```

### Pricing Section
```tsx
// src/components/marketing/Pricing.tsx
'use client'
import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { Check } from 'lucide-react'

const plans = [
  {
    name: 'Starter',
    price: { monthly: 29, yearly: 290 },
    description: 'Perfect for small projects and MVPs',
    features: [
      'Up to 1,000 users',
      'Basic analytics',
      'Email support',
      'Standard templates',
      'Basic integrations'
    ],
    popular: false
  },
  {
    name: 'Professional',
    price: { monthly: 99, yearly: 990 },
    description: 'Best for growing businesses',
    features: [
      'Up to 10,000 users',
      'Advanced analytics',
      'Priority support',
      'Custom branding',
      'Advanced integrations',
      'A/B testing',
      'Custom domains'
    ],
    popular: true
  },
  {
    name: 'Enterprise',
    price: { monthly: 299, yearly: 2990 },
    description: 'For large-scale applications',
    features: [
      'Unlimited users',
      'Custom analytics',
      'Dedicated support',
      'White-label solution',
      'Custom integrations',
      'SLA guarantee',
      'On-premise option'
    ],
    popular: false
  }
]

export function Pricing() {
  const [isYearly, setIsYearly] = useState(false)
  
  return (
    <section className="py-20 bg-gray-50">
      <div className="container mx-auto px-4">
        <div className="mx-auto max-w-3xl text-center mb-16">
          <h2 className="text-3xl font-bold text-gray-900 sm:text-4xl">
            Simple, transparent pricing
          </h2>
          <p className="mt-4 text-lg text-gray-600">
            Choose the plan that's right for your project. No hidden fees.
          </p>
          
          {/* Billing Toggle */}
          <div className="mt-8 flex items-center justify-center">
            <button
              onClick={() => setIsYearly(false)}
              className={`px-4 py-2 text-sm font-medium ${
                !isYearly ? 'text-blue-600' : 'text-gray-500'
              }`}
            >
              Monthly
            </button>
            <div className="mx-3 flex items-center">
              <button
                onClick={() => setIsYearly(!isYearly)}
                className={`relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${
                  isYearly ? 'bg-blue-600' : 'bg-gray-200'
                }`}
              >
                <span
                  className={`inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${
                    isYearly ? 'translate-x-6' : 'translate-x-1'
                  }`}
                />
              </button>
            </div>
            <button
              onClick={() => setIsYearly(true)}
              className={`px-4 py-2 text-sm font-medium ${
                isYearly ? 'text-blue-600' : 'text-gray-500'
              }`}
            >
              Yearly
              <span className="ml-1 text-xs text-green-600 font-semibold">
                Save 20%
              </span>
            </button>
          </div>
        </div>
        
        <div className="grid gap-8 lg:grid-cols-3">
          {plans.map((plan, index) => (
            <div
              key={index}
              className={`relative rounded-2xl bg-white p-8 shadow-lg ${
                plan.popular ? 'ring-2 ring-blue-600' : ''
              }`}
            >
              {plan.popular && (
                <div className="absolute -top-3 left-1/2 -translate-x-1/2">
                  <span className="bg-blue-600 text-white px-4 py-1 text-sm font-medium rounded-full">
                    Most Popular
                  </span>
                </div>
              )}
              
              <div className="text-center">
                <h3 className="text-xl font-semibold text-gray-900">
                  {plan.name}
                </h3>
                <p className="mt-2 text-gray-600">
                  {plan.description}
                </p>
                <div className="mt-6">
                  <span className="text-4xl font-bold text-gray-900">
                    ${isYearly ? plan.price.yearly : plan.price.monthly}
                  </span>
                  <span className="text-gray-600">
                    /{isYearly ? 'year' : 'month'}
                  </span>
                </div>
              </div>
              
              <ul className="mt-8 space-y-3">
                {plan.features.map((feature, featureIndex) => (
                  <li key={featureIndex} className="flex items-center">
                    <Check className="h-5 w-5 text-green-500 mr-3" />
                    <span className="text-gray-700">{feature}</span>
                  </li>
                ))}
              </ul>
              
              <Button
                className={`w-full mt-8 ${
                  plan.popular
                    ? 'bg-blue-600 hover:bg-blue-700'
                    : 'bg-gray-900 hover:bg-gray-800'
                }`}
              >
                Get Started
              </Button>
            </div>
          ))}
        </div>
      </div>
    </section>
  )
}
```

## SEO Optimization

### Metadata Configuration
```typescript
// src/lib/seo/metadata.ts
import { Metadata } from 'next'

export const siteConfig = {
  name: 'Your SaaS',
  description: 'The complete toolkit for launching your SaaS product',
  url: 'https://yoursaas.com',
  ogImage: 'https://yoursaas.com/og-image.jpg',
  links: {
    twitter: 'https://twitter.com/yoursaas',
    github: 'https://github.com/yoursaas'
  }
}

export function constructMetadata({
  title = siteConfig.name,
  description = siteConfig.description,
  image = siteConfig.ogImage,
  noIndex = false
}: {
  title?: string
  description?: string
  image?: string
  noIndex?: boolean
} = {}): Metadata {
  return {
    title,
    description,
    openGraph: {
      title,
      description,
      type: 'website',
      images: [{ url: image }]
    },
    twitter: {
      card: 'summary_large_image',
      title,
      description,
      images: [image],
      creator: '@yoursaas'
    },
    icons: {
      icon: '/favicon.ico',
      shortcut: '/favicon-16x16.png',
      apple: '/apple-touch-icon.png'
    },
    metadataBase: new URL(siteConfig.url),
    ...(noIndex && {
      robots: {
        index: false,
        follow: false
      }
    })
  }
}

// Usage in pages
export const metadata = constructMetadata({
  title: 'Pricing - Your SaaS',
  description: 'Simple, transparent pricing for your SaaS project'
})
```

### Structured Data
```tsx
// src/components/marketing/StructuredData.tsx
export function StructuredData() {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'SoftwareApplication',
    name: 'Your SaaS',
    applicationCategory: 'BusinessApplication',
    description: 'The complete toolkit for launching your SaaS product',
    url: 'https://yoursaas.com',
    screenshot: 'https://yoursaas.com/screenshot.jpg',
    author: {
      '@type': 'Organization',
      name: 'Your Company'
    },
    offers: {
      '@type': 'Offer',
      price: '29',
      priceCurrency: 'USD',
      priceValidUntil: '2025-12-31'
    }
  }
  
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
    />
  )
}
```

## Analytics Integration

### Analytics Setup
```typescript
// src/lib/analytics/tracking.ts
declare global {
  interface Window {
    gtag: Function
  }
}

export const GA_TRACKING_ID = process.env.NEXT_PUBLIC_GA_ID

export const pageview = (url: string) => {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('config', GA_TRACKING_ID, {
      page_location: url
    })
  }
}

export const event = ({
  action,
  category,
  label,
  value
}: {
  action: string
  category: string
  label?: string
  value?: number
}) => {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('event', action, {
      event_category: category,
      event_label: label,
      value: value
    })
  }
}

// Track conversions
export const trackConversion = (conversionType: string, value?: number) => {
  event({
    action: 'conversion',
    category: 'engagement',
    label: conversionType,
    value
  })
}
```

### Conversion Tracking Component
```tsx
// src/components/marketing/ConversionTracker.tsx
'use client'
import { useEffect } from 'react'
import { trackConversion } from '@/lib/analytics/tracking'

interface ConversionTrackerProps {
  conversionType: 'signup' | 'trial' | 'purchase' | 'demo_request'
  value?: number
}

export function ConversionTracker({ conversionType, value }: ConversionTrackerProps) {
  useEffect(() => {
    trackConversion(conversionType, value)
  }, [conversionType, value])
  
  return null // This component doesn't render anything
}
```

## Performance Optimization

### Image Optimization
```tsx
// Always use Next.js Image component
import Image from 'next/image'

export function OptimizedHero() {
  return (
    <div className="relative">
      <Image
        src="/hero-background.jpg"
        alt="Hero Background"
        fill
        className="object-cover"
        priority // Load immediately for above-fold images
        sizes="100vw"
      />
    </div>
  )
}
```

### Font Optimization
```typescript
// src/app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Improve performance
  variable: '--font-inter'
})

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans">{children}</body>
    </html>
  )
}
```

## A/B Testing Setup

### Feature Flag System
```typescript
// src/lib/experiments/feature-flags.ts
export const featureFlags = {
  heroVariantB: false,
  pricingTestV2: false,
  newCheckoutFlow: false
}

export function getFeatureFlag(flag: keyof typeof featureFlags): boolean {
  // In production, this might come from a service like LaunchDarkly
  return featureFlags[flag]
}

// Usage in components
export function Hero() {
  const showVariantB = getFeatureFlag('heroVariantB')
  
  if (showVariantB) {
    return <HeroVariantB />
  }
  
  return <HeroVariantA />
}
```

## Newsletter Integration

### Newsletter Component
```tsx
// src/components/marketing/Newsletter.tsx
'use client'
import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'

export function Newsletter() {
  const [email, setEmail] = useState('')
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle')
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setStatus('loading')
    
    try {
      const response = await fetch('/api/newsletter/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email })
      })
      
      if (response.ok) {
        setStatus('success')
        setEmail('')
      } else {
        setStatus('error')
      }
    } catch (error) {
      setStatus('error')
    }
  }
  
  return (
    <section className="bg-blue-600 py-16">
      <div className="container mx-auto px-4 text-center">
        <h2 className="text-3xl font-bold text-white mb-4">
          Stay Updated
        </h2>
        <p className="text-blue-100 mb-8 max-w-2xl mx-auto">
          Get the latest updates, tips, and insights delivered to your inbox.
        </p>
        
        <form onSubmit={handleSubmit} className="max-w-md mx-auto flex gap-2">
          <Input
            type="email"
            placeholder="Enter your email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
            className="flex-1"
          />
          <Button 
            type="submit" 
            disabled={status === 'loading'}
            variant="secondary"
          >
            {status === 'loading' ? 'Subscribing...' : 'Subscribe'}
          </Button>
        </form>
        
        {status === 'success' && (
          <p className="text-green-200 mt-4">Thanks for subscribing!</p>
        )}
        {status === 'error' && (
          <p className="text-red-200 mt-4">Something went wrong. Please try again.</p>
        )}
      </div>
    </section>
  )
}
```

## Landing Page Best Practices

### Conversion Optimization
1. **Clear Value Proposition**: Immediately communicate what you do and why it matters
2. **Social Proof**: Include testimonials, user counts, and logos
3. **Compelling CTAs**: Use action-oriented language and contrasting colors
4. **Remove Friction**: Minimize form fields and steps to conversion
5. **Mobile Optimization**: Ensure excellent mobile experience

### Performance Guidelines
1. **Core Web Vitals**: Optimize LCP, FID, and CLS scores
2. **Image Optimization**: Use Next.js Image component with proper sizing
3. **Code Splitting**: Lazy load components not in viewport
4. **Caching**: Implement proper caching strategies
5. **CDN**: Use CDN for static assets

### SEO Checklist
- [ ] Proper title tags and meta descriptions
- [ ] Structured data markup
- [ ] XML sitemap
- [ ] Robots.txt
- [ ] Internal linking structure
- [ ] Page speed optimization
- [ ] Mobile-friendly design
- [ ] SSL certificate

This landing page system provides a complete foundation for marketing your SaaS product with built-in conversion optimization, SEO, and performance best practices.