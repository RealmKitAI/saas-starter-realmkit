# Admin Module - AI Context

## Admin System Overview
This realm includes a comprehensive admin dashboard for managing users, monitoring system health, and controlling application settings. The admin system uses role-based access control (RBAC) with middleware protection and audit logging.

## Key Files and Locations

### Admin Pages
- `src/app/(dashboard)/dashboard/admin/page.tsx` - Admin dashboard home
- `src/app/(dashboard)/dashboard/admin/users/page.tsx` - User management
- `src/app/(dashboard)/dashboard/admin/analytics/page.tsx` - System analytics
- `src/app/(dashboard)/dashboard/admin/settings/page.tsx` - System settings
- `src/app/(dashboard)/dashboard/admin/audit/page.tsx` - Audit logs

### API Routes
- `src/app/api/admin/users/route.ts` - User management API
- `src/app/api/admin/analytics/route.ts` - Analytics data API
- `src/app/api/admin/settings/route.ts` - System settings API
- `src/app/api/admin/audit/route.ts` - Audit log API

### Services
- `src/services/admin/admin.service.ts` - Main admin operations
- `src/services/admin/user-management.service.ts` - User management logic
- `src/services/admin/analytics.service.ts` - Analytics calculations
- `src/services/admin/audit.service.ts` - Audit logging

### Components
- `src/components/admin/UserTable.tsx` - User management table
- `src/components/admin/AnalyticsCards.tsx` - Metrics display cards
- `src/components/admin/SettingsForm.tsx` - System settings form
- `src/components/admin/AuditLogTable.tsx` - Audit log display

### Middleware
- `src/middleware/admin-auth.ts` - Admin role verification
- `src/middleware/audit-logger.ts` - Automatic audit logging

## Role-Based Access Control

### User Roles
```typescript
// types/user.ts
export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin'
}

export interface User {
  id: string
  email: string
  role: UserRole
  // ... other fields
}
```

### Admin Middleware
```typescript
// middleware/admin-auth.ts
import { auth } from '@/lib/auth'
import { NextResponse } from 'next/server'

export async function adminAuthMiddleware() {
  const session = await auth()
  
  if (!session) {
    return NextResponse.redirect('/login')
  }
  
  if (!['admin', 'super_admin'].includes(session.user.role)) {
    return NextResponse.redirect('/dashboard?error=insufficient_permissions')
  }
  
  return NextResponse.next()
}

// In page components
export default async function AdminPage() {
  const session = await auth()
  
  if (!session || !['admin', 'super_admin'].includes(session.user.role)) {
    redirect('/dashboard')
  }
  
  return <AdminDashboard />
}
```

## User Management System

### User Management Service
```typescript
// services/admin/user-management.service.ts
export const userManagementService = {
  async getAllUsers(page = 1, limit = 50, search?: string) {
    const offset = (page - 1) * limit
    
    const where = search ? {
      OR: [
        { email: { contains: search, mode: 'insensitive' } },
        { name: { contains: search, mode: 'insensitive' } }
      ]
    } : {}
    
    const [users, total] = await Promise.all([
      db.user.findMany({
        where,
        skip: offset,
        take: limit,
        orderBy: { createdAt: 'desc' },
        include: {
          _count: {
            select: { sessions: true }
          }
        }
      }),
      db.user.count({ where })
    ])
    
    return { users, total, pages: Math.ceil(total / limit) }
  },

  async updateUserRole(adminId: string, userId: string, newRole: UserRole) {
    // Audit log
    await auditService.log({
      adminId,
      action: 'user_role_change',
      targetUserId: userId,
      oldValue: await this.getUserRole(userId),
      newValue: newRole
    })
    
    return db.user.update({
      where: { id: userId },
      data: { role: newRole },
      select: { id: true, email: true, role: true }
    })
  },

  async suspendUser(adminId: string, userId: string, reason: string) {
    await auditService.log({
      adminId,
      action: 'user_suspended',
      targetUserId: userId,
      details: { reason }
    })
    
    return db.user.update({
      where: { id: userId },
      data: { 
        suspended: true,
        suspendedAt: new Date(),
        suspensionReason: reason
      }
    })
  },

  async deleteUser(adminId: string, userId: string) {
    // Soft delete with audit trail
    await auditService.log({
      adminId,
      action: 'user_deleted',
      targetUserId: userId
    })
    
    return db.user.update({
      where: { id: userId },
      data: { 
        deleted: true,
        deletedAt: new Date(),
        email: `deleted_${userId}@deleted.local` // Prevent email conflicts
      }
    })
  }
}
```

### User Management Component
```tsx
// components/admin/UserTable.tsx
'use client'
import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'

interface User {
  id: string
  email: string
  name?: string
  role: string
  createdAt: string
  suspended: boolean
}

export function UserTable({ initialUsers }: { initialUsers: User[] }) {
  const [users, setUsers] = useState(initialUsers)
  const [loading, setLoading] = useState<string | null>(null)
  
  const handleRoleChange = async (userId: string, newRole: string) => {
    setLoading(userId)
    try {
      const response = await fetch('/api/admin/users', {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId, role: newRole })
      })
      
      if (response.ok) {
        setUsers(users.map(user => 
          user.id === userId ? { ...user, role: newRole } : user
        ))
      }
    } catch (error) {
      console.error('Failed to update user role:', error)
    } finally {
      setLoading(null)
    }
  }
  
  const handleSuspendUser = async (userId: string) => {
    const reason = prompt('Reason for suspension:')
    if (!reason) return
    
    setLoading(userId)
    try {
      const response = await fetch('/api/admin/users/suspend', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId, reason })
      })
      
      if (response.ok) {
        setUsers(users.map(user => 
          user.id === userId ? { ...user, suspended: true } : user
        ))
      }
    } catch (error) {
      console.error('Failed to suspend user:', error)
    } finally {
      setLoading(null)
    }
  }
  
  return (
    <div className="border rounded-lg">
      <table className="w-full">
        <thead>
          <tr className="border-b">
            <th className="text-left p-4">User</th>
            <th className="text-left p-4">Role</th>
            <th className="text-left p-4">Status</th>
            <th className="text-left p-4">Created</th>
            <th className="text-left p-4">Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id} className="border-b">
              <td className="p-4">
                <div>
                  <div className="font-medium">{user.name || 'No name'}</div>
                  <div className="text-sm text-gray-500">{user.email}</div>
                </div>
              </td>
              <td className="p-4">
                <select
                  value={user.role}
                  onChange={(e) => handleRoleChange(user.id, e.target.value)}
                  disabled={loading === user.id}
                  className="border rounded px-2 py-1"
                >
                  <option value="user">User</option>
                  <option value="admin">Admin</option>
                  <option value="super_admin">Super Admin</option>
                </select>
              </td>
              <td className="p-4">
                <Badge variant={user.suspended ? 'destructive' : 'default'}>
                  {user.suspended ? 'Suspended' : 'Active'}
                </Badge>
              </td>
              <td className="p-4">
                {new Date(user.createdAt).toLocaleDateString()}
              </td>
              <td className="p-4">
                <div className="flex gap-2">
                  {!user.suspended && (
                    <Button
                      size="sm"
                      variant="outline"
                      onClick={() => handleSuspendUser(user.id)}
                      disabled={loading === user.id}
                    >
                      Suspend
                    </Button>
                  )}
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

## System Analytics

### Analytics Service
```typescript
// services/admin/analytics.service.ts
export const analyticsService = {
  async getDashboardMetrics() {
    const [
      totalUsers,
      activeUsers,
      totalRevenue,
      subscriptions
    ] = await Promise.all([
      db.user.count(),
      db.user.count({
        where: {
          sessions: {
            some: {
              updatedAt: {
                gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // 30 days
              }
            }
          }
        }
      }),
      this.getTotalRevenue(),
      this.getSubscriptionStats()
    ])
    
    return {
      totalUsers,
      activeUsers,
      totalRevenue,
      subscriptions,
      growth: await this.getGrowthMetrics()
    }
  },

  async getUserGrowth(days = 30) {
    const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000)
    
    const users = await db.user.findMany({
      where: {
        createdAt: { gte: startDate }
      },
      select: {
        createdAt: true
      },
      orderBy: { createdAt: 'asc' }
    })
    
    // Group by day
    const dailyGrowth = users.reduce((acc, user) => {
      const date = user.createdAt.toISOString().split('T')[0]
      acc[date] = (acc[date] || 0) + 1
      return acc
    }, {} as Record<string, number>)
    
    return dailyGrowth
  },

  async getRevenueAnalytics(days = 30) {
    const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000)
    
    const invoices = await db.invoice.findMany({
      where: {
        createdAt: { gte: startDate },
        status: 'paid'
      },
      select: {
        amount: true,
        createdAt: true
      }
    })
    
    const dailyRevenue = invoices.reduce((acc, invoice) => {
      const date = invoice.createdAt.toISOString().split('T')[0]
      acc[date] = (acc[date] || 0) + invoice.amount
      return acc
    }, {} as Record<string, number>)
    
    return dailyRevenue
  }
}
```

## Audit Logging System

### Audit Service
```typescript
// services/admin/audit.service.ts
export const auditService = {
  async log(entry: {
    adminId: string
    action: string
    targetUserId?: string
    details?: any
    oldValue?: any
    newValue?: any
  }) {
    return db.auditLog.create({
      data: {
        adminId: entry.adminId,
        action: entry.action,
        targetUserId: entry.targetUserId,
        details: entry.details ? JSON.stringify(entry.details) : null,
        oldValue: entry.oldValue ? JSON.stringify(entry.oldValue) : null,
        newValue: entry.newValue ? JSON.stringify(entry.newValue) : null,
        timestamp: new Date(),
        ipAddress: this.getClientIP(),
        userAgent: this.getUserAgent()
      }
    })
  },

  async getAuditLogs(page = 1, limit = 50, filters?: {
    adminId?: string
    action?: string
    dateFrom?: Date
    dateTo?: Date
  }) {
    const offset = (page - 1) * limit
    
    const where = {
      ...(filters?.adminId && { adminId: filters.adminId }),
      ...(filters?.action && { action: filters.action }),
      ...(filters?.dateFrom || filters?.dateTo) && {
        timestamp: {
          ...(filters.dateFrom && { gte: filters.dateFrom }),
          ...(filters.dateTo && { lte: filters.dateTo })
        }
      }
    }
    
    const [logs, total] = await Promise.all([
      db.auditLog.findMany({
        where,
        skip: offset,
        take: limit,
        orderBy: { timestamp: 'desc' },
        include: {
          admin: { select: { email: true, name: true } },
          targetUser: { select: { email: true, name: true } }
        }
      }),
      db.auditLog.count({ where })
    ])
    
    return { logs, total, pages: Math.ceil(total / limit) }
  }
}
```

### Database Schema
```prisma
model AuditLog {
  id           String   @id @default(cuid())
  adminId      String
  action       String   // user_created, user_deleted, role_changed, etc.
  targetUserId String?
  details      String?  // JSON string
  oldValue     String?  // JSON string
  newValue     String?  // JSON string
  timestamp    DateTime @default(now())
  ipAddress    String?
  userAgent    String?
  
  admin        User  @relation("AdminAuditLogs", fields: [adminId], references: [id])
  targetUser   User? @relation("TargetUserAuditLogs", fields: [targetUserId], references: [id])
}

// Add to User model
model User {
  // ... existing fields
  role         String   @default("user")
  suspended    Boolean  @default(false)
  suspendedAt  DateTime?
  suspensionReason String?
  deleted      Boolean  @default(false)
  deletedAt    DateTime?
  
  // Audit relationships
  adminAuditLogs    AuditLog[] @relation("AdminAuditLogs")
  targetAuditLogs   AuditLog[] @relation("TargetUserAuditLogs")
}
```

## System Settings Management

### Settings Service
```typescript
// services/admin/settings.service.ts
export const settingsService = {
  async getSettings() {
    const settings = await db.systemSetting.findMany()
    return settings.reduce((acc, setting) => {
      acc[setting.key] = JSON.parse(setting.value)
      return acc
    }, {} as Record<string, any>)
  },

  async updateSetting(key: string, value: any, adminId: string) {
    await auditService.log({
      adminId,
      action: 'setting_updated',
      details: { key },
      oldValue: await this.getSetting(key),
      newValue: value
    })
    
    return db.systemSetting.upsert({
      where: { key },
      update: { value: JSON.stringify(value) },
      create: { key, value: JSON.stringify(value) }
    })
  },

  async getSetting(key: string, defaultValue?: any) {
    const setting = await db.systemSetting.findUnique({
      where: { key }
    })
    
    return setting ? JSON.parse(setting.value) : defaultValue
  }
}
```

## Admin Dashboard Features

### Feature Flags
```typescript
// Admin can control feature flags
const features = {
  userRegistration: true,
  maintenanceMode: false,
  newUserEmailVerification: true,
  stripePayments: true
}

// Use in middleware
export function middleware(request: NextRequest) {
  const isMaintenanceMode = await settingsService.getSetting('maintenanceMode', false)
  
  if (isMaintenanceMode && !request.nextUrl.pathname.startsWith('/admin')) {
    return NextResponse.redirect('/maintenance')
  }
}
```

### Bulk Operations
```typescript
// Bulk user operations
export const userManagementService = {
  async bulkUpdateUsers(adminId: string, userIds: string[], updates: Partial<User>) {
    await auditService.log({
      adminId,
      action: 'bulk_user_update',
      details: { userIds, updates }
    })
    
    return db.user.updateMany({
      where: { id: { in: userIds } },
      data: updates
    })
  }
}
```

## Security Considerations

1. **Role Verification**: Always verify admin role on both client and server
2. **Audit Everything**: Log all admin actions for compliance
3. **Rate Limiting**: Implement rate limits on admin endpoints
4. **IP Restrictions**: Consider IP whitelist for admin access
5. **Session Management**: Admin sessions should have shorter timeouts

## Testing Admin Features
```typescript
// Mock admin user for tests
const mockAdminSession = {
  user: {
    id: 'admin-123',
    email: 'admin@test.com',
    role: 'admin'
  }
}

// Test admin middleware
describe('Admin Authentication', () => {
  it('should allow admin access', async () => {
    // Test implementation
  })
  
  it('should deny non-admin access', async () => {
    // Test implementation
  })
})
```