# Admin Panel Module - AI Context

## 👑 **Admin Dashboard & Management Patterns**

Load this context when working with admin/management features.

---

## 🏗️ **Admin Architecture**

### **Admin Flow**
```
Admin Login → Role Check → Admin Dashboard → Management Actions → Audit Logs
```

### **Key Components**
- **Admin Routes**: Protected admin-only pages
- **User Management**: CRUD operations for users
- **System Analytics**: Usage metrics and insights
- **Audit Logging**: Track admin actions

---

## 🔐 **Admin Route Protection**

### **Admin Middleware**
```typescript
// middleware/admin.ts
import { auth } from '@/lib/auth';
import { UserRole } from '@/types/auth.types';

export async function requireAdmin(request: NextRequest) {
  const session = await auth();
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  if (!['ADMIN', 'SUPER_ADMIN'].includes(session.user.role)) {
    return NextResponse.json({ error: 'Forbidden - Admin required' }, { status: 403 });
  }

  return null; // Access granted
}

export async function requireSuperAdmin(request: NextRequest) {
  const session = await auth();
  
  if (!session?.user || session.user.role !== 'SUPER_ADMIN') {
    return NextResponse.json({ error: 'Forbidden - Super Admin required' }, { status: 403 });
  }

  return null;
}
```

### **Admin Layout**
```typescript
// app/admin/layout.tsx
export default async function AdminLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth();
  
  // Redirect non-admins
  if (!session?.user || !['ADMIN', 'SUPER_ADMIN'].includes(session.user.role)) {
    redirect('/login');
  }

  return (
    <div className="min-h-screen bg-gray-100">
      <AdminSidebar />
      <div className="lg:pl-64">
        <AdminHeader user={session.user} />
        <main className="p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

---

## 👥 **User Management Service**

### **Admin User Service**
```typescript
// services/admin.service.ts
export class AdminService {
  /**
   * Gets paginated users with filters
   */
  static async getUsers({
    page = 1,
    limit = 20,
    search,
    role,
    status,
  }: {
    page?: number;
    limit?: number;
    search?: string;
    role?: UserRole;
    status?: 'ACTIVE' | 'SUSPENDED';
  }): Promise<{ users: User[]; total: number; stats: UserStats }> {
    const where: any = {};

    if (search) {
      where.OR = [
        { email: { contains: search, mode: 'insensitive' } },
        { name: { contains: search, mode: 'insensitive' } },
      ];
    }

    if (role) where.role = role;
    if (status) where.status = status;

    const [users, total, stats] = await Promise.all([
      db.user.findMany({
        where,
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          status: true,
          createdAt: true,
          lastLoginAt: true,
          _count: {
            select: {
              subscriptions: { where: { status: 'ACTIVE' } },
              uploads: true,
            },
          },
        },
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
      }),
      db.user.count({ where }),
      this.getUserStats(),
    ]);

    return { users, total, stats };
  }

  /**
   * Gets user statistics for admin dashboard
   */
  static async getUserStats(): Promise<UserStats> {
    const [totalUsers, activeUsers, newUsersThisMonth, paidUsers] = await Promise.all([
      db.user.count(),
      db.user.count({ where: { status: 'ACTIVE' } }),
      db.user.count({
        where: {
          createdAt: {
            gte: new Date(new Date().getFullYear(), new Date().getMonth(), 1),
          },
        },
      }),
      db.subscription.count({ where: { status: 'ACTIVE' } }),
    ]);

    return {
      totalUsers,
      activeUsers,
      newUsersThisMonth,
      paidUsers,
      conversionRate: totalUsers > 0 ? (paidUsers / totalUsers) * 100 : 0,
    };
  }

  /**
   * Updates user role (Super Admin only)
   */
  static async updateUserRole(
    userId: string,
    newRole: UserRole,
    adminId: string
  ): Promise<void> {
    // Log the action
    await this.logAdminAction(adminId, 'UPDATE_USER_ROLE', {
      userId,
      newRole,
    });

    // Update user
    await db.user.update({
      where: { id: userId },
      data: { role: newRole },
    });

    // Send notification email if promoting to admin
    if (['ADMIN', 'SUPER_ADMIN'].includes(newRole)) {
      const user = await UserService.getById(userId);
      if (user) {
        await EmailService.sendRoleUpdate(user.email, newRole);
      }
    }
  }

  /**
   * Suspends or activates user account
   */
  static async updateUserStatus(
    userId: string,
    status: 'ACTIVE' | 'SUSPENDED',
    reason: string,
    adminId: string
  ): Promise<void> {
    await this.logAdminAction(adminId, 'UPDATE_USER_STATUS', {
      userId,
      status,
      reason,
    });

    await db.user.update({
      where: { id: userId },
      data: { status },
    });

    // Send notification email
    const user = await UserService.getById(userId);
    if (user) {
      if (status === 'SUSPENDED') {
        await EmailService.sendAccountSuspended(user.email, reason);
      } else {
        await EmailService.sendAccountReactivated(user.email);
      }
    }
  }

  /**
   * Gets system analytics for admin dashboard
   */
  static async getSystemAnalytics(): Promise<SystemAnalytics> {
    const [
      userGrowth,
      revenueStats,
      topFeatures,
      errorStats,
    ] = await Promise.all([
      this.getUserGrowthData(),
      this.getRevenueStats(),
      this.getFeatureUsageStats(),
      this.getErrorStats(),
    ]);

    return {
      userGrowth,
      revenueStats,
      topFeatures,
      errorStats,
    };
  }

  /**
   * Logs admin actions for audit trail
   */
  static async logAdminAction(
    adminId: string,
    action: string,
    details: Record<string, any>
  ): Promise<void> {
    await db.adminLog.create({
      data: {
        adminId,
        action,
        details,
        timestamp: new Date(),
      },
    });
  }

  /**
   * Gets admin action logs with pagination
   */
  static async getAdminLogs({
    page = 1,
    limit = 50,
    adminId,
    action,
  }: {
    page?: number;
    limit?: number;
    adminId?: string;
    action?: string;
  }): Promise<{ logs: AdminLog[]; total: number }> {
    const where: any = {};
    if (adminId) where.adminId = adminId;
    if (action) where.action = action;

    const [logs, total] = await Promise.all([
      db.adminLog.findMany({
        where,
        include: {
          admin: {
            select: { email: true, name: true },
          },
        },
        orderBy: { timestamp: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
      }),
      db.adminLog.count({ where }),
    ]);

    return { logs, total };
  }

  private static async getUserGrowthData(): Promise<UserGrowthData[]> {
    // Get user signups for last 12 months
    const months = Array.from({ length: 12 }, (_, i) => {
      const date = new Date();
      date.setMonth(date.getMonth() - i);
      return date;
    }).reverse();

    return await Promise.all(
      months.map(async (month) => {
        const nextMonth = new Date(month);
        nextMonth.setMonth(nextMonth.getMonth() + 1);

        const count = await db.user.count({
          where: {
            createdAt: {
              gte: month,
              lt: nextMonth,
            },
          },
        });

        return {
          month: month.toISOString().slice(0, 7), // YYYY-MM format
          users: count,
        };
      })
    );
  }

  private static async getRevenueStats(): Promise<RevenueStats> {
    const activeSubscriptions = await db.subscription.findMany({
      where: { status: 'ACTIVE' },
      select: { plan: true },
    });

    const monthlyRevenue = activeSubscriptions.reduce((total, sub) => {
      const planRevenue = {
        'PRO': 29,
        'ENTERPRISE': 99,
        'FREE': 0,
      }[sub.plan] || 0;
      
      return total + planRevenue;
    }, 0);

    return {
      monthlyRecurringRevenue: monthlyRevenue,
      totalActiveSubscriptions: activeSubscriptions.length,
      averageRevenuePerUser: activeSubscriptions.length > 0 
        ? monthlyRevenue / activeSubscriptions.length 
        : 0,
    };
  }
}
```

---

## 🎨 **Admin UI Components**

### **Admin Dashboard Overview**
```typescript
// components/admin/admin-dashboard.tsx
'use client';

interface AdminDashboardProps {
  stats: UserStats;
  analytics: SystemAnalytics;
}

export function AdminDashboard({ stats, analytics }: AdminDashboardProps) {
  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold">Admin Dashboard</h1>
        <div className="text-sm text-gray-500">
          Last updated: {format(new Date(), 'PPp')}
        </div>
      </div>

      {/* Stats Cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <StatCard
          title="Total Users"
          value={stats.totalUsers.toLocaleString()}
          change={`+${stats.newUsersThisMonth}`}
          changeLabel="this month"
          icon={<UsersIcon className="h-8 w-8 text-blue-600" />}
        />
        <StatCard
          title="Active Users"
          value={stats.activeUsers.toLocaleString()}
          change={`${((stats.activeUsers / stats.totalUsers) * 100).toFixed(1)}%`}
          changeLabel="of total"
          icon={<UserCheckIcon className="h-8 w-8 text-green-600" />}
        />
        <StatCard
          title="Paid Users"
          value={stats.paidUsers.toLocaleString()}
          change={`${stats.conversionRate.toFixed(1)}%`}
          changeLabel="conversion"
          icon={<CreditCardIcon className="h-8 w-8 text-purple-600" />}
        />
        <StatCard
          title="MRR"
          value={`$${analytics.revenueStats.monthlyRecurringRevenue.toLocaleString()}`}
          change={`$${analytics.revenueStats.averageRevenuePerUser.toFixed(2)}`}
          changeLabel="ARPU"
          icon={<TrendingUpIcon className="h-8 w-8 text-orange-600" />}
        />
      </div>

      {/* Charts */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div className="bg-white p-6 rounded-lg shadow">
          <h3 className="text-lg font-semibold mb-4">User Growth</h3>
          <UserGrowthChart data={analytics.userGrowth} />
        </div>
        
        <div className="bg-white p-6 rounded-lg shadow">
          <h3 className="text-lg font-semibold mb-4">Feature Usage</h3>
          <FeatureUsageChart data={analytics.topFeatures} />
        </div>
      </div>

      {/* Quick Actions */}
      <div className="bg-white p-6 rounded-lg shadow">
        <h3 className="text-lg font-semibold mb-4">Quick Actions</h3>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <Button className="justify-start" variant="outline">
            <PlusIcon className="h-4 w-4 mr-2" />
            Add New User
          </Button>
          <Button className="justify-start" variant="outline">
            <MailIcon className="h-4 w-4 mr-2" />
            Send Newsletter
          </Button>
          <Button className="justify-start" variant="outline">
            <DownloadIcon className="h-4 w-4 mr-2" />
            Export Data
          </Button>
        </div>
      </div>
    </div>
  );
}

function StatCard({
  title,
  value,
  change,
  changeLabel,
  icon,
}: {
  title: string;
  value: string;
  change: string;
  changeLabel: string;
  icon: React.ReactNode;
}) {
  return (
    <div className="bg-white p-6 rounded-lg shadow">
      <div className="flex items-center justify-between">
        <div>
          <p className="text-sm font-medium text-gray-600">{title}</p>
          <p className="text-3xl font-bold text-gray-900">{value}</p>
          <p className="text-sm text-gray-500">
            <span className="font-medium text-green-600">{change}</span> {changeLabel}
          </p>
        </div>
        {icon}
      </div>
    </div>
  );
}
```

### **User Management Table**
```typescript
// components/admin/user-management-table.tsx
'use client';

interface UserManagementTableProps {
  initialUsers: User[];
  initialTotal: number;
}

export function UserManagementTable({ 
  initialUsers, 
  initialTotal 
}: UserManagementTableProps) {
  const [users, setUsers] = useState(initialUsers);
  const [total, setTotal] = useState(initialTotal);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [filters, setFilters] = useState({
    search: '',
    role: '',
    status: '',
  });

  const fetchUsers = async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({
        page: page.toString(),
        ...filters,
      });

      const response = await fetch(`/api/admin/users?${params}`);
      const data = await response.json();
      
      setUsers(data.users);
      setTotal(data.total);
    } catch (error) {
      toast.error('Failed to fetch users');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, [page, filters]);

  const handleRoleUpdate = async (userId: string, newRole: UserRole) => {
    try {
      await fetch(`/api/admin/users/${userId}/role`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ role: newRole }),
      });

      // Update local state
      setUsers(prev => 
        prev.map(user => 
          user.id === userId ? { ...user, role: newRole } : user
        )
      );

      toast.success('User role updated');
    } catch (error) {
      toast.error('Failed to update user role');
    }
  };

  const handleStatusUpdate = async (
    userId: string, 
    status: 'ACTIVE' | 'SUSPENDED',
    reason?: string
  ) => {
    try {
      await fetch(`/api/admin/users/${userId}/status`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ status, reason }),
      });

      setUsers(prev => 
        prev.map(user => 
          user.id === userId ? { ...user, status } : user
        )
      );

      toast.success(`User ${status.toLowerCase()}`);
    } catch (error) {
      toast.error('Failed to update user status');
    }
  };

  return (
    <div className="space-y-4">
      {/* Filters */}
      <div className="flex space-x-4">
        <Input
          placeholder="Search users..."
          value={filters.search}
          onChange={(e) => setFilters(prev => ({ ...prev, search: e.target.value }))}
          className="max-w-sm"
        />
        <Select
          value={filters.role}
          onValueChange={(value) => setFilters(prev => ({ ...prev, role: value }))}
        >
          <SelectTrigger className="w-32">
            <SelectValue placeholder="Role" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="">All Roles</SelectItem>
            <SelectItem value="USER">User</SelectItem>
            <SelectItem value="ADMIN">Admin</SelectItem>
            <SelectItem value="SUPER_ADMIN">Super Admin</SelectItem>
          </SelectContent>
        </Select>
      </div>

      {/* Table */}
      <div className="bg-white rounded-lg shadow overflow-hidden">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                User
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Role
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Status
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Joined
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Subscription
              </th>
              <th className="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {loading ? (
              <tr>
                <td colSpan={6} className="px-6 py-4 text-center">
                  <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600 mx-auto" />
                </td>
              </tr>
            ) : (
              users.map((user) => (
                <UserRow
                  key={user.id}
                  user={user}
                  onRoleUpdate={handleRoleUpdate}
                  onStatusUpdate={handleStatusUpdate}
                />
              ))
            )}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="flex items-center justify-between">
        <div className="text-sm text-gray-700">
          Showing {(page - 1) * 20 + 1} to {Math.min(page * 20, total)} of {total} users
        </div>
        <div className="flex space-x-2">
          <Button
            variant="outline"
            onClick={() => setPage(prev => Math.max(1, prev - 1))}
            disabled={page === 1}
          >
            Previous
          </Button>
          <Button
            variant="outline"
            onClick={() => setPage(prev => prev + 1)}
            disabled={page * 20 >= total}
          >
            Next
          </Button>
        </div>
      </div>
    </div>
  );
}
```

---

## 📊 **Admin API Routes**

### **User Management API**
```typescript
// app/api/admin/users/route.ts
export async function GET(request: NextRequest) {
  try {
    // Check admin permission
    const authResult = await requireAdmin(request);
    if (authResult) return authResult;

    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') ?? '1');
    const search = searchParams.get('search') ?? undefined;
    const role = searchParams.get('role') as UserRole ?? undefined;
    const status = searchParams.get('status') as 'ACTIVE' | 'SUSPENDED' ?? undefined;

    const result = await AdminService.getUsers({
      page,
      search,
      role,
      status,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Admin users API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### **Analytics API**
```typescript
// app/api/admin/analytics/route.ts
export async function GET(request: NextRequest) {
  try {
    const authResult = await requireAdmin(request);
    if (authResult) return authResult;

    const analytics = await AdminService.getSystemAnalytics();

    return NextResponse.json({ analytics });
  } catch (error) {
    console.error('Admin analytics API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## 🧪 **Admin Testing**

### **Admin Service Testing**
```typescript
// services/__tests__/admin.service.test.ts
describe('AdminService', () => {
  describe('getUsers', () => {
    it('should return paginated users with stats', async () => {
      const mockUsers = [
        { id: '1', email: 'user1@example.com', role: 'USER' },
        { id: '2', email: 'user2@example.com', role: 'ADMIN' },
      ];

      mockDb.user.findMany.mockResolvedValue(mockUsers);
      mockDb.user.count.mockResolvedValue(2);

      const result = await AdminService.getUsers({ page: 1, limit: 20 });

      expect(result.users).toEqual(mockUsers);
      expect(result.total).toBe(2);
      expect(result.stats).toBeDefined();
    });
  });

  describe('updateUserRole', () => {
    it('should update user role and log action', async () => {
      const userId = 'user_123';
      const newRole = UserRole.ADMIN;
      const adminId = 'admin_123';

      await AdminService.updateUserRole(userId, newRole, adminId);

      expect(mockDb.user.update).toHaveBeenCalledWith({
        where: { id: userId },
        data: { role: newRole },
      });

      expect(mockDb.adminLog.create).toHaveBeenCalledWith({
        data: {
          adminId,
          action: 'UPDATE_USER_ROLE',
          details: { userId, newRole },
          timestamp: expect.any(Date),
        },
      });
    });
  });
});
```

---

## 📋 **Admin Panel Checklist**

When implementing admin features:
- [ ] Set up admin role-based access control
- [ ] Create admin dashboard with key metrics
- [ ] Implement user management (CRUD operations)
- [ ] Add system analytics and reporting
- [ ] Create audit logging for admin actions
- [ ] Build admin UI components
- [ ] Add bulk operations for efficiency
- [ ] Implement data export functionality
- [ ] Set up admin notification system
- [ ] Write comprehensive tests
- [ ] Add proper error handling
- [ ] Document admin procedures

---

**This admin context provides complete admin panel patterns. Only load when admin/management features are required.**