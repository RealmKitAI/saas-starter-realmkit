# Landing Page Module - AI Context

## 🚀 **Landing Page & Marketing Site Patterns**

Load this context when working with landing pages, marketing sites, and public-facing content.

---

## 🏗️ **Landing Page Architecture**

### **Landing Flow**
```
Visitor → Landing Page → Value Props → CTA → Signup/Login → Product
```

### **Key Components**
- **Hero Section**: Primary value proposition and CTA
- **Features Section**: Key product benefits
- **Pricing Section**: Plans and pricing tiers
- **Social Proof**: Testimonials, logos, stats
- **FAQ Section**: Common questions addressed

---

## 📄 **Landing Page Structure**

### **Main Landing Page**
```typescript
// app/page.tsx (Root landing page)
export default function LandingPage() {
  return (
    <div className="min-h-screen">
      <LandingHeader />
      <HeroSection />
      <FeaturesSection />
      <SocialProofSection />
      <PricingSection />
      <FAQSection />
      <CTASection />
      <LandingFooter />
    </div>
  );
}
```

### **Landing Layout**
```typescript
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen bg-white">
      <LandingHeader />
      <main>
        {children}
      </main>
      <LandingFooter />
    </div>
  );
}
```

---

## 🎨 **Hero Section Patterns**

### **Hero with Video/Image**
```typescript
// components/landing/hero-section.tsx
export function HeroSection() {
  return (
    <section className="relative bg-gradient-to-r from-blue-600 to-purple-700 text-white">
      <div className="absolute inset-0 bg-black opacity-20"></div>
      <div className="relative max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-24">
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">
          {/* Content */}
          <div className="space-y-8">
            <div className="space-y-4">
              <h1 className="text-4xl md:text-6xl font-bold leading-tight">
                Build Better
                <span className="text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 to-orange-500">
                  {" "}SaaS Apps
                </span>
              </h1>
              <p className="text-xl md:text-2xl text-gray-200 max-w-2xl">
                Get your SaaS up and running in minutes, not months. 
                Complete with auth, payments, and everything you need.
              </p>
            </div>

            {/* CTAs */}
            <div className="flex flex-col sm:flex-row gap-4">
              <Button size="lg" className="bg-white text-gray-900 hover:bg-gray-100">
                Start Free Trial
                <ArrowRightIcon className="ml-2 h-5 w-5" />
              </Button>
              <Button size="lg" variant="outline" className="border-white text-white hover:bg-white hover:text-gray-900">
                Watch Demo
                <PlayIcon className="ml-2 h-5 w-5" />
              </Button>
            </div>

            {/* Social Proof */}
            <div className="flex items-center space-x-6 text-sm">
              <div className="flex items-center space-x-1">
                <div className="flex -space-x-2">
                  {Array.from({ length: 3 }, (_, i) => (
                    <img
                      key={i}
                      className="inline-block h-8 w-8 rounded-full ring-2 ring-white"
                      src={`/avatars/user-${i + 1}.jpg`}
                      alt="User"
                    />
                  ))}
                </div>
                <span className="text-gray-200">Join 1,000+ developers</span>
              </div>
              <div className="flex items-center space-x-1">
                <StarIcon className="h-5 w-5 text-yellow-400 fill-current" />
                <span>4.9/5 rating</span>
              </div>
            </div>
          </div>

          {/* Visual */}
          <div className="relative">
            <div className="bg-white rounded-lg shadow-2xl p-8">
              <DashboardPreview />
            </div>
            {/* Floating elements */}
            <div className="absolute -top-4 -right-4 bg-green-500 text-white px-3 py-1 rounded-full text-sm font-semibold">
              Live Demo
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

---

## ⭐ **Features Section Patterns**

### **Feature Grid with Icons**
```typescript
// components/landing/features-section.tsx
const features = [
  {
    icon: <AuthIcon className="h-8 w-8" />,
    title: "Authentication Ready",
    description: "Complete auth system with NextAuth.js, social logins, and role-based access control.",
  },
  {
    icon: <PaymentIcon className="h-8 w-8" />,
    title: "Stripe Integration",
    description: "Built-in payment processing, subscriptions, and billing management with Stripe.",
  },
  {
    icon: <DatabaseIcon className="h-8 w-8" />,
    title: "Database Included",
    description: "PostgreSQL with Prisma ORM, migrations, and type-safe queries out of the box.",
  },
  {
    icon: <CloudIcon className="h-8 w-8" />,
    title: "File Uploads",
    description: "S3 integration with image processing, CDN delivery, and secure file management.",
  },
  {
    icon: <EmailIcon className="h-8 w-8" />,
    title: "Email System",
    description: "Transactional emails, notifications, and beautiful email templates included.",
  },
  {
    icon: <AdminIcon className="h-8 w-8" />,
    title: "Admin Dashboard",
    description: "Complete admin panel with user management, analytics, and system controls.",
  },
];

export function FeaturesSection() {
  return (
    <section className="py-24 bg-gray-50">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center space-y-4 mb-16">
          <h2 className="text-3xl md:text-4xl font-bold text-gray-900">
            Everything You Need to Build SaaS
          </h2>
          <p className="text-xl text-gray-600 max-w-3xl mx-auto">
            Stop reinventing the wheel. Our starter includes all the essential features 
            you need to launch your SaaS product quickly.
          </p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {features.map((feature, index) => (
            <div
              key={index}
              className="bg-white rounded-xl p-8 shadow-sm hover:shadow-md transition-shadow"
            >
              <div className="text-blue-600 mb-4">
                {feature.icon}
              </div>
              <h3 className="text-xl font-semibold text-gray-900 mb-3">
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
  );
}
```

---

## 💰 **Pricing Section Patterns**

### **Pricing Cards with Toggle**
```typescript
// components/landing/pricing-section.tsx
'use client';

export function PricingSection() {
  const [annually, setAnnually] = useState(true);
  
  const plans = [
    {
      name: 'Starter',
      description: 'Perfect for side projects',
      price: { monthly: 0, annual: 0 },
      features: [
        '1 project included',
        'Basic authentication',
        'PostgreSQL database',
        'Email support',
        'Basic analytics',
      ],
      cta: 'Start Free',
      popular: false,
    },
    {
      name: 'Pro',
      description: 'For growing businesses',
      price: { monthly: 29, annual: 290 },
      features: [
        'Unlimited projects',
        'Advanced authentication',
        'Stripe payments included',
        'File uploads & storage',
        'Email templates',
        'Admin dashboard',
        'Priority support',
        'Advanced analytics',
      ],
      cta: 'Start Pro Trial',
      popular: true,
    },
    {
      name: 'Enterprise',
      description: 'For large organizations',
      price: { monthly: 99, annual: 990 },
      features: [
        'Everything in Pro',
        'Custom integrations',
        'SSO & SAML',
        'Advanced security',
        'Custom domains',
        'Dedicated support',
        'SLA guarantee',
        'White-label options',
      ],
      cta: 'Contact Sales',
      popular: false,
    },
  ];

  return (
    <section className="py-24 bg-white">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center space-y-4 mb-16">
          <h2 className="text-3xl md:text-4xl font-bold text-gray-900">
            Simple, Transparent Pricing
          </h2>
          <p className="text-xl text-gray-600 max-w-3xl mx-auto">
            Choose the perfect plan for your needs. Start free, scale as you grow.
          </p>

          {/* Billing Toggle */}
          <div className="flex items-center justify-center space-x-4 mt-8">
            <span className={`text-sm ${!annually ? 'text-gray-900' : 'text-gray-500'}`}>
              Monthly
            </span>
            <button
              onClick={() => setAnnually(!annually)}
              className={`relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${
                annually ? 'bg-blue-600' : 'bg-gray-200'
              }`}
            >
              <span
                className={`inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${
                  annually ? 'translate-x-6' : 'translate-x-1'
                }`}
              />
            </button>
            <span className={`text-sm ${annually ? 'text-gray-900' : 'text-gray-500'}`}>
              Annual
            </span>
            <span className="bg-green-100 text-green-800 px-2 py-1 rounded-full text-xs font-semibold">
              Save 20%
            </span>
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
          {plans.map((plan, index) => (
            <div
              key={index}
              className={`relative rounded-2xl p-8 ${
                plan.popular
                  ? 'bg-blue-50 border-2 border-blue-200'
                  : 'bg-white border border-gray-200'
              }`}
            >
              {plan.popular && (
                <div className="absolute -top-3 left-1/2 transform -translate-x-1/2">
                  <span className="bg-blue-600 text-white px-4 py-1 rounded-full text-sm font-semibold">
                    Most Popular
                  </span>
                </div>
              )}

              <div className="space-y-6">
                <div>
                  <h3 className="text-2xl font-bold text-gray-900">{plan.name}</h3>
                  <p className="text-gray-600 mt-2">{plan.description}</p>
                </div>

                <div className="flex items-baseline space-x-2">
                  <span className="text-5xl font-bold text-gray-900">
                    ${annually ? plan.price.annual : plan.price.monthly}
                  </span>
                  <span className="text-gray-500">
                    {plan.price.monthly > 0 ? (annually ? '/year' : '/month') : ''}
                  </span>
                </div>

                <Button
                  className={`w-full py-3 ${
                    plan.popular
                      ? 'bg-blue-600 hover:bg-blue-700 text-white'
                      : 'bg-gray-900 hover:bg-gray-800 text-white'
                  }`}
                >
                  {plan.cta}
                </Button>

                <div className="space-y-3">
                  <p className="text-sm font-semibold text-gray-900">
                    Everything included:
                  </p>
                  <ul className="space-y-2">
                    {plan.features.map((feature, featureIndex) => (
                      <li key={featureIndex} className="flex items-center space-x-3">
                        <CheckIcon className="h-5 w-5 text-green-500" />
                        <span className="text-gray-600">{feature}</span>
                      </li>
                    ))}
                  </ul>
                </div>
              </div>
            </div>
          ))}
        </div>

        {/* FAQ Link */}
        <div className="text-center mt-12">
          <p className="text-gray-600">
            Have questions about our plans?{' '}
            <a href="#faq" className="text-blue-600 hover:underline">
              Check our FAQ
            </a>{' '}
            or{' '}
            <a href="/contact" className="text-blue-600 hover:underline">
              contact us
            </a>
          </p>
        </div>
      </div>
    </section>
  );
}
```

---

## 🗣️ **Social Proof Patterns**

### **Testimonials with Photos**
```typescript
// components/landing/social-proof-section.tsx
const testimonials = [
  {
    content: "This starter saved us months of development time. We went from idea to paying customers in just 2 weeks!",
    author: "Sarah Chen",
    role: "Founder, TechFlow",
    avatar: "/testimonials/sarah.jpg",
    company: "TechFlow",
    rating: 5,
  },
  {
    content: "The code quality is exceptional. Everything follows best practices and the documentation is crystal clear.",
    author: "Marcus Rodriguez",
    role: "Lead Developer, BuildCorp",
    avatar: "/testimonials/marcus.jpg",
    company: "BuildCorp",
    rating: 5,
  },
  {
    content: "Amazing support and constant updates. The team really cares about making this the best SaaS starter.",
    author: "Emily Watson",
    role: "CTO, DataViz",
    avatar: "/testimonials/emily.jpg",
    company: "DataViz",
    rating: 5,
  },
];

const companies = [
  { name: "TechFlow", logo: "/logos/techflow.svg" },
  { name: "BuildCorp", logo: "/logos/buildcorp.svg" },
  { name: "DataViz", logo: "/logos/dataviz.svg" },
  { name: "InnovateLab", logo: "/logos/innovatelab.svg" },
  { name: "CloudSync", logo: "/logos/cloudsync.svg" },
];

export function SocialProofSection() {
  return (
    <section className="py-24 bg-gray-50">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        {/* Stats */}
        <div className="text-center mb-16">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
            <div className="space-y-2">
              <div className="text-4xl font-bold text-blue-600">1,000+</div>
              <div className="text-gray-600">Happy Developers</div>
            </div>
            <div className="space-y-2">
              <div className="text-4xl font-bold text-blue-600">50+</div>
              <div className="text-gray-600">SaaS Apps Launched</div>
            </div>
            <div className="space-y-2">
              <div className="text-4xl font-bold text-blue-600">99.9%</div>
              <div className="text-gray-600">Uptime Guarantee</div>
            </div>
          </div>
        </div>

        {/* Company Logos */}
        <div className="mb-16">
          <p className="text-center text-gray-600 mb-8">
            Trusted by innovative companies worldwide
          </p>
          <div className="flex flex-wrap justify-center items-center gap-8 opacity-60">
            {companies.map((company, index) => (
              <img
                key={index}
                src={company.logo}
                alt={company.name}
                className="h-8 grayscale hover:grayscale-0 transition-all"
              />
            ))}
          </div>
        </div>

        {/* Testimonials */}
        <div className="space-y-4 mb-8">
          <h2 className="text-3xl md:text-4xl font-bold text-gray-900 text-center">
            What Developers Say
          </h2>
          <p className="text-xl text-gray-600 text-center">
            Don't just take our word for it
          </p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {testimonials.map((testimonial, index) => (
            <div key={index} className="bg-white rounded-xl p-8 shadow-sm">
              <div className="flex items-center space-x-1 mb-4">
                {Array.from({ length: testimonial.rating }, (_, i) => (
                  <StarIcon key={i} className="h-5 w-5 text-yellow-400 fill-current" />
                ))}
              </div>
              
              <blockquote className="text-gray-700 mb-6">
                "{testimonial.content}"
              </blockquote>
              
              <div className="flex items-center space-x-4">
                <img
                  className="h-12 w-12 rounded-full"
                  src={testimonial.avatar}
                  alt={testimonial.author}
                />
                <div>
                  <div className="font-semibold text-gray-900">
                    {testimonial.author}
                  </div>
                  <div className="text-sm text-gray-600">
                    {testimonial.role}, {testimonial.company}
                  </div>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

---

## ❓ **FAQ Section Patterns**

### **Expandable FAQ**
```typescript
// components/landing/faq-section.tsx
'use client';

const faqs = [
  {
    question: "What's included in the SaaS starter?",
    answer: "Our starter includes authentication with NextAuth.js, Stripe payment integration, PostgreSQL database with Prisma, file uploads with S3, email system, admin dashboard, Docker setup, and comprehensive documentation."
  },
  {
    question: "Can I customize the code?",
    answer: "Absolutely! You get full access to the source code. Modify, extend, and customize anything to fit your specific needs. The code follows best practices and is well-documented for easy customization."
  },
  {
    question: "What tech stack is used?",
    answer: "We use Next.js 14 with App Router, TypeScript, Tailwind CSS, PostgreSQL, Prisma ORM, NextAuth.js, Stripe, AWS S3, and Docker. Everything is containerized for easy deployment."
  },
  {
    question: "Is there ongoing support?",
    answer: "Yes! We provide documentation, example code, and community support. Pro and Enterprise plans include priority support and regular updates."
  },
  {
    question: "Can I use this for client projects?",
    answer: "Yes, our license allows you to use the starter for unlimited personal and commercial projects, including client work."
  },
];

export function FAQSection() {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  return (
    <section className="py-24 bg-white">
      <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center space-y-4 mb-16">
          <h2 className="text-3xl md:text-4xl font-bold text-gray-900">
            Frequently Asked Questions
          </h2>
          <p className="text-xl text-gray-600">
            Everything you need to know about our SaaS starter
          </p>
        </div>

        <div className="space-y-4">
          {faqs.map((faq, index) => (
            <div key={index} className="border border-gray-200 rounded-lg">
              <button
                className="w-full px-6 py-4 text-left flex justify-between items-center hover:bg-gray-50 transition-colors"
                onClick={() => setOpenIndex(openIndex === index ? null : index)}
              >
                <span className="font-semibold text-gray-900">
                  {faq.question}
                </span>
                <ChevronDownIcon
                  className={`h-5 w-5 text-gray-500 transition-transform ${
                    openIndex === index ? 'transform rotate-180' : ''
                  }`}
                />
              </button>
              {openIndex === index && (
                <div className="px-6 pb-4">
                  <p className="text-gray-600">{faq.answer}</p>
                </div>
              )}
            </div>
          ))}
        </div>

        <div className="text-center mt-12">
          <p className="text-gray-600 mb-4">
            Still have questions?
          </p>
          <Button variant="outline">
            Contact Support
          </Button>
        </div>
      </div>
    </section>
  );
}
```

---

## 🎯 **CTA Section Patterns**

### **Final CTA with Urgency**
```typescript
// components/landing/cta-section.tsx
export function CTASection() {
  return (
    <section className="py-24 bg-gradient-to-r from-blue-600 to-purple-700 text-white">
      <div className="max-w-4xl mx-auto text-center px-4 sm:px-6 lg:px-8">
        <div className="space-y-8">
          <div className="space-y-4">
            <h2 className="text-3xl md:text-4xl font-bold">
              Ready to Launch Your SaaS?
            </h2>
            <p className="text-xl text-blue-100 max-w-2xl mx-auto">
              Join thousands of developers who've accelerated their SaaS development. 
              Start building today with our production-ready starter.
            </p>
          </div>

          <div className="flex flex-col sm:flex-row gap-4 justify-center">
            <Button size="lg" className="bg-white text-gray-900 hover:bg-gray-100">
              Start Free Trial
              <ArrowRightIcon className="ml-2 h-5 w-5" />
            </Button>
            <Button size="lg" variant="outline" className="border-white text-white hover:bg-white hover:text-gray-900">
              View Documentation
            </Button>
          </div>

          <div className="flex items-center justify-center space-x-6 text-sm text-blue-200">
            <div className="flex items-center space-x-2">
              <CheckIcon className="h-5 w-5" />
              <span>14-day free trial</span>
            </div>
            <div className="flex items-center space-x-2">
              <CheckIcon className="h-5 w-5" />
              <span>Cancel anytime</span>
            </div>
            <div className="flex items-center space-x-2">
              <CheckIcon className="h-5 w-5" />
              <span>No setup fees</span>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

---

## 🧭 **Landing Header & Footer**

### **Landing Navigation**
```typescript
// components/landing/landing-header.tsx
'use client';

export function LandingHeader() {
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  return (
    <header className="bg-white shadow-sm sticky top-0 z-50">
      <nav className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between h-16">
          <div className="flex items-center">
            <Link href="/" className="flex items-center space-x-2">
              <LogoIcon className="h-8 w-8" />
              <span className="text-2xl font-bold text-gray-900">
                SaaS Starter
              </span>
            </Link>
          </div>

          {/* Desktop Navigation */}
          <div className="hidden md:flex items-center space-x-8">
            <Link href="#features" className="text-gray-700 hover:text-gray-900">
              Features
            </Link>
            <Link href="#pricing" className="text-gray-700 hover:text-gray-900">
              Pricing
            </Link>
            <Link href="/docs" className="text-gray-700 hover:text-gray-900">
              Docs
            </Link>
            <Link href="/blog" className="text-gray-700 hover:text-gray-900">
              Blog
            </Link>
          </div>

          <div className="hidden md:flex items-center space-x-4">
            <Button variant="ghost" asChild>
              <Link href="/login">Sign In</Link>
            </Button>
            <Button asChild>
              <Link href="/register">Get Started</Link>
            </Button>
          </div>

          {/* Mobile menu button */}
          <div className="md:hidden flex items-center">
            <button
              onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
              className="text-gray-700"
            >
              <MenuIcon className="h-6 w-6" />
            </button>
          </div>
        </div>

        {/* Mobile Navigation */}
        {mobileMenuOpen && (
          <div className="md:hidden py-4 border-t">
            <div className="flex flex-col space-y-4">
              <Link href="#features" className="text-gray-700">Features</Link>
              <Link href="#pricing" className="text-gray-700">Pricing</Link>
              <Link href="/docs" className="text-gray-700">Docs</Link>
              <Link href="/blog" className="text-gray-700">Blog</Link>
              <div className="flex flex-col space-y-2 pt-4 border-t">
                <Button variant="ghost" asChild>
                  <Link href="/login">Sign In</Link>
                </Button>
                <Button asChild>
                  <Link href="/register">Get Started</Link>
                </Button>
              </div>
            </div>
          </div>
        )}
      </nav>
    </header>
  );
}
```

### **Landing Footer**
```typescript
// components/landing/landing-footer.tsx
export function LandingFooter() {
  return (
    <footer className="bg-gray-900 text-white">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-8">
          <div className="space-y-4">
            <div className="flex items-center space-x-2">
              <LogoIcon className="h-8 w-8" />
              <span className="text-2xl font-bold">SaaS Starter</span>
            </div>
            <p className="text-gray-400">
              The fastest way to build and launch your SaaS application.
            </p>
          </div>

          <div>
            <h3 className="font-semibold mb-4">Product</h3>
            <ul className="space-y-2 text-gray-400">
              <li><Link href="#features" className="hover:text-white">Features</Link></li>
              <li><Link href="#pricing" className="hover:text-white">Pricing</Link></li>
              <li><Link href="/docs" className="hover:text-white">Documentation</Link></li>
              <li><Link href="/changelog" className="hover:text-white">Changelog</Link></li>
            </ul>
          </div>

          <div>
            <h3 className="font-semibold mb-4">Company</h3>
            <ul className="space-y-2 text-gray-400">
              <li><Link href="/about" className="hover:text-white">About</Link></li>
              <li><Link href="/blog" className="hover:text-white">Blog</Link></li>
              <li><Link href="/contact" className="hover:text-white">Contact</Link></li>
              <li><Link href="/careers" className="hover:text-white">Careers</Link></li>
            </ul>
          </div>

          <div>
            <h3 className="font-semibold mb-4">Legal</h3>
            <ul className="space-y-2 text-gray-400">
              <li><Link href="/privacy" className="hover:text-white">Privacy Policy</Link></li>
              <li><Link href="/terms" className="hover:text-white">Terms of Service</Link></li>
              <li><Link href="/security" className="hover:text-white">Security</Link></li>
            </ul>
          </div>
        </div>

        <div className="border-t border-gray-800 mt-12 pt-8 flex flex-col md:flex-row justify-between items-center">
          <p className="text-gray-400 text-sm">
            © 2024 SaaS Starter. All rights reserved.
          </p>
          <div className="flex space-x-6 mt-4 md:mt-0">
            <a href="#" className="text-gray-400 hover:text-white">
              <TwitterIcon className="h-5 w-5" />
            </a>
            <a href="#" className="text-gray-400 hover:text-white">
              <GitHubIcon className="h-5 w-5" />
            </a>
            <a href="#" className="text-gray-400 hover:text-white">
              <LinkedInIcon className="h-5 w-5" />
            </a>
          </div>
        </div>
      </div>
    </footer>
  );
}
```

---

## 🧪 **Landing Page Testing**

### **Landing Component Tests**
```typescript
// components/__tests__/hero-section.test.tsx
describe('HeroSection', () => {
  it('should render hero content correctly', () => {
    render(<HeroSection />);
    
    expect(screen.getByText(/Build Better SaaS Apps/)).toBeInTheDocument();
    expect(screen.getByText('Start Free Trial')).toBeInTheDocument();
    expect(screen.getByText('Watch Demo')).toBeInTheDocument();
  });

  it('should have proper CTA buttons', () => {
    render(<HeroSection />);
    
    const startButton = screen.getByText('Start Free Trial');
    const demoButton = screen.getByText('Watch Demo');
    
    expect(startButton).toHaveAttribute('href', '/register');
    expect(demoButton).toHaveAttribute('href', '/demo');
  });
});
```

### **SEO Testing**
```typescript
// tests/seo/landing-seo.test.ts
describe('Landing Page SEO', () => {
  it('should have proper meta tags', async () => {
    const page = await browser.newPage();
    await page.goto('http://localhost:3000');
    
    const title = await page.title();
    expect(title).toContain('SaaS Starter');
    
    const description = await page.$eval('meta[name="description"]', el => el.content);
    expect(description).toContain('fastest way to build SaaS');
  });
});
```

---

## 📋 **Landing Page Checklist**

When implementing landing pages:
- [ ] Create compelling hero section with clear value prop
- [ ] Add feature highlights with benefits
- [ ] Include social proof (testimonials, logos, stats)
- [ ] Design pricing section with clear CTAs
- [ ] Build responsive FAQ section
- [ ] Add final CTA section with urgency
- [ ] Implement proper navigation and footer
- [ ] Optimize for SEO (meta tags, schema markup)
- [ ] Add analytics tracking
- [ ] Test on mobile devices
- [ ] Implement A/B testing capabilities
- [ ] Add loading states and animations

---

**This landing page context provides complete marketing site patterns. Only load when building public-facing marketing pages.**