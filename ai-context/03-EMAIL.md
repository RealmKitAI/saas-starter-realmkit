# Email & Notifications Module - AI Context

## 📧 **Email Integration Patterns**

Load this context when working with email/notification features.

---

## 🏗️ **Email Service Architecture**

### **Email Flow**
```
Trigger → Email Service → Provider (Resend/SendGrid) → User Inbox
```

### **Key Components**
- **Email Service**: Business logic for sending emails
- **Template System**: Reusable email templates
- **Queue System**: For bulk/delayed emails
- **Provider Integration**: Resend, SendGrid, etc.

---

## 📨 **Email Configuration**

### **Resend Client Setup**
```typescript
// lib/email.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);

// Email configuration
export const EMAIL_CONFIG = {
  from: process.env.EMAIL_FROM || 'noreply@yourapp.com',
  replyTo: process.env.EMAIL_REPLY_TO || 'support@yourapp.com',
} as const;
```

### **Email Templates**
```typescript
// templates/email-templates.tsx
import { Html, Head, Body, Container, Text, Button } from '@react-email/components';

interface WelcomeEmailProps {
  userName: string;
  loginUrl: string;
}

export function WelcomeEmail({ userName, loginUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'Arial, sans-serif' }}>
        <Container style={{ padding: '20px' }}>
          <Text style={{ fontSize: '24px', fontWeight: 'bold' }}>
            Welcome, {userName}!
          </Text>
          <Text>
            Thanks for signing up. Get started by logging into your account.
          </Text>
          <Button
            href={loginUrl}
            style={{
              backgroundColor: '#007ee6',
              color: 'white',
              padding: '12px 20px',
              borderRadius: '5px',
              textDecoration: 'none',
            }}
          >
            Go to Dashboard
          </Button>
        </Container>
      </Body>
    </Html>
  );
}

export function PasswordResetEmail({ resetUrl }: { resetUrl: string }) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'Arial, sans-serif' }}>
        <Container style={{ padding: '20px' }}>
          <Text style={{ fontSize: '24px', fontWeight: 'bold' }}>
            Reset Your Password
          </Text>
          <Text>
            Click the link below to reset your password. This link expires in 24 hours.
          </Text>
          <Button
            href={resetUrl}
            style={{
              backgroundColor: '#dc2626',
              color: 'white',
              padding: '12px 20px',
              borderRadius: '5px',
              textDecoration: 'none',
            }}
          >
            Reset Password
          </Button>
        </Container>
      </Body>
    </Html>
  );
}
```

---

## 📧 **Email Service Patterns**

### **Core Email Service**
```typescript
// services/email.service.ts
import { render } from '@react-email/render';
import { WelcomeEmail, PasswordResetEmail } from '@/templates/email-templates';

export class EmailService {
  /**
   * Sends welcome email to new user
   */
  static async sendWelcome(userEmail: string, userName: string): Promise<void> {
    const loginUrl = `${process.env.NEXT_PUBLIC_APP_URL}/login`;
    
    const emailHtml = render(
      WelcomeEmail({ userName, loginUrl })
    );

    await resend.emails.send({
      from: EMAIL_CONFIG.from,
      to: userEmail,
      subject: `Welcome to ${process.env.NEXT_PUBLIC_APP_NAME}!`,
      html: emailHtml,
    });
  }

  /**
   * Sends password reset email
   */
  static async sendPasswordReset(
    userEmail: string, 
    resetToken: string
  ): Promise<void> {
    const resetUrl = `${process.env.NEXT_PUBLIC_APP_URL}/reset-password?token=${resetToken}`;
    
    const emailHtml = render(
      PasswordResetEmail({ resetUrl })
    );

    await resend.emails.send({
      from: EMAIL_CONFIG.from,
      to: userEmail,
      subject: 'Reset Your Password',
      html: emailHtml,
    });
  }

  /**
   * Sends subscription welcome email
   */
  static async sendSubscriptionWelcome(
    userId: string, 
    planName: string
  ): Promise<void> {
    const user = await UserService.getById(userId);
    if (!user) return;

    const emailHtml = render(`
      <h1>Welcome to ${planName}!</h1>
      <p>Hi ${user.name},</p>
      <p>Your ${planName} subscription is now active. Enjoy your new features!</p>
      <a href="${process.env.NEXT_PUBLIC_APP_URL}/dashboard">Go to Dashboard</a>
    `);

    await resend.emails.send({
      from: EMAIL_CONFIG.from,
      to: user.email,
      subject: `Welcome to ${planName}!`,
      html: emailHtml,
    });
  }

  /**
   * Sends bulk email to multiple users
   */
  static async sendBulkEmail(
    userIds: string[],
    subject: string,
    template: React.ReactElement
  ): Promise<void> {
    const users = await UserService.getByIds(userIds);
    const emailHtml = render(template);

    // Send in batches to avoid rate limits
    const batchSize = 50;
    for (let i = 0; i < users.length; i += batchSize) {
      const batch = users.slice(i, i + batchSize);
      
      await Promise.all(
        batch.map(user => 
          resend.emails.send({
            from: EMAIL_CONFIG.from,
            to: user.email,
            subject,
            html: emailHtml,
          })
        )
      );

      // Small delay between batches
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
}
```

---

## 🔔 **Notification System**

### **In-App Notifications**
```typescript
// services/notification.service.ts
export enum NotificationType {
  INFO = 'info',
  SUCCESS = 'success', 
  WARNING = 'warning',
  ERROR = 'error',
}

export class NotificationService {
  /**
   * Creates in-app notification for user
   */
  static async createNotification({
    userId,
    title,
    message,
    type = NotificationType.INFO,
    actionUrl,
  }: {
    userId: string;
    title: string;
    message: string;
    type?: NotificationType;
    actionUrl?: string;
  }): Promise<void> {
    await db.notification.create({
      data: {
        userId,
        title,
        message,
        type,
        actionUrl,
        read: false,
      },
    });
  }

  /**
   * Gets user notifications
   */
  static async getUserNotifications(
    userId: string,
    limit = 20
  ): Promise<Notification[]> {
    return await db.notification.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' },
      take: limit,
    });
  }

  /**
   * Marks notification as read
   */
  static async markAsRead(notificationId: string, userId: string): Promise<void> {
    await db.notification.updateMany({
      where: { 
        id: notificationId,
        userId, // Security: only mark own notifications
      },
      data: { read: true },
    });
  }

  /**
   * Marks all user notifications as read
   */
  static async markAllAsRead(userId: string): Promise<void> {
    await db.notification.updateMany({
      where: { userId, read: false },
      data: { read: true },
    });
  }
}
```

### **Real-time Notifications**
```typescript
// hooks/use-notifications.ts
'use client';

export function useNotifications() {
  const { data: session } = useSession();
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    if (!session?.user) return;

    // Fetch notifications
    const fetchNotifications = async () => {
      const response = await fetch('/api/notifications');
      const data = await response.json();
      setNotifications(data.notifications);
      setUnreadCount(data.unreadCount);
    };

    fetchNotifications();

    // Set up polling for real-time updates
    const interval = setInterval(fetchNotifications, 30000); // 30 seconds

    return () => clearInterval(interval);
  }, [session]);

  const markAsRead = async (notificationId: string) => {
    await fetch(`/api/notifications/${notificationId}/read`, {
      method: 'PUT',
    });

    // Update local state
    setNotifications(prev => 
      prev.map(n => 
        n.id === notificationId ? { ...n, read: true } : n
      )
    );
    setUnreadCount(prev => Math.max(0, prev - 1));
  };

  const markAllAsRead = async () => {
    await fetch('/api/notifications/mark-all-read', {
      method: 'PUT',
    });

    setNotifications(prev => prev.map(n => ({ ...n, read: true })));
    setUnreadCount(0);
  };

  return {
    notifications,
    unreadCount,
    markAsRead,
    markAllAsRead,
  };
}
```

---

## 🎨 **Email UI Components**

### **Notification Bell Component**
```typescript
// components/notifications/notification-bell.tsx
'use client';

export function NotificationBell() {
  const { notifications, unreadCount, markAsRead } = useNotifications();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="relative">
      <Button
        variant="ghost"
        size="sm"
        onClick={() => setIsOpen(!isOpen)}
        className="relative"
      >
        <BellIcon className="h-5 w-5" />
        {unreadCount > 0 && (
          <span className="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center">
            {unreadCount > 99 ? '99+' : unreadCount}
          </span>
        )}
      </Button>

      {isOpen && (
        <div className="absolute right-0 mt-2 w-80 bg-white border rounded-lg shadow-lg z-50">
          <div className="p-4 border-b">
            <div className="flex justify-between items-center">
              <h3 className="font-semibold">Notifications</h3>
              {unreadCount > 0 && (
                <Button 
                  variant="ghost" 
                  size="sm"
                  onClick={() => markAllAsRead()}
                >
                  Mark all as read
                </Button>
              )}
            </div>
          </div>

          <div className="max-h-96 overflow-y-auto">
            {notifications.length === 0 ? (
              <div className="p-4 text-center text-gray-500">
                No notifications
              </div>
            ) : (
              notifications.map(notification => (
                <div
                  key={notification.id}
                  className={`p-3 border-b hover:bg-gray-50 cursor-pointer ${
                    !notification.read ? 'bg-blue-50' : ''
                  }`}
                  onClick={() => {
                    if (!notification.read) {
                      markAsRead(notification.id);
                    }
                    if (notification.actionUrl) {
                      window.location.href = notification.actionUrl;
                    }
                  }}
                >
                  <div className="flex justify-between items-start">
                    <div className="flex-1">
                      <h4 className="font-medium text-sm">
                        {notification.title}
                      </h4>
                      <p className="text-gray-600 text-xs mt-1">
                        {notification.message}
                      </p>
                      <p className="text-gray-400 text-xs mt-1">
                        {format(new Date(notification.createdAt), 'MMM d, h:mm a')}
                      </p>
                    </div>
                    {!notification.read && (
                      <div className="w-2 h-2 bg-blue-500 rounded-full ml-2" />
                    )}
                  </div>
                </div>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## 🧪 **Email Testing**

### **Email Service Testing**
```typescript
// services/__tests__/email.service.test.ts
describe('EmailService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('sendWelcome', () => {
    it('should send welcome email with correct parameters', async () => {
      const mockSend = jest.fn().mockResolvedValue({ success: true });
      (resend.emails.send as jest.Mock) = mockSend;

      await EmailService.sendWelcome('test@example.com', 'John Doe');

      expect(mockSend).toHaveBeenCalledWith({
        from: EMAIL_CONFIG.from,
        to: 'test@example.com',
        subject: expect.stringContaining('Welcome'),
        html: expect.stringContaining('John Doe'),
      });
    });
  });

  describe('sendBulkEmail', () => {
    it('should send emails in batches', async () => {
      const mockUsers = Array.from({ length: 75 }, (_, i) => ({
        id: `user_${i}`,
        email: `user${i}@example.com`,
        name: `User ${i}`,
      }));

      mockUserService.getByIds.mockResolvedValue(mockUsers);
      const mockSend = jest.fn().mockResolvedValue({ success: true });
      (resend.emails.send as jest.Mock) = mockSend;

      await EmailService.sendBulkEmail(
        mockUsers.map(u => u.id),
        'Test Subject',
        <div>Test Template</div>
      );

      // Should send in 2 batches (50 + 25)
      expect(mockSend).toHaveBeenCalledTimes(75);
    });
  });
});
```

---

## 📋 **Email Integration Checklist**

When implementing email features:
- [ ] Set up email provider (Resend/SendGrid)
- [ ] Create reusable email templates
- [ ] Implement core email service methods
- [ ] Add notification system for in-app alerts
- [ ] Create notification UI components
- [ ] Handle email failures gracefully
- [ ] Add email rate limiting
- [ ] Test email templates in development
- [ ] Set up email analytics/tracking
- [ ] Write comprehensive tests

---

**This email context provides complete email and notification patterns. Only load when email features are required.**