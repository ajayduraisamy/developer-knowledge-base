# Authorization - Role-based access, permissions, protected routes, middleware

## Introduction

Authorization determines what an authenticated user is allowed to do within an application. While authentication verifies identity, authorization controls access to resources and actions. Role-Based Access Control (RBAC) is the most common authorization model, where permissions are assigned to roles, and users are assigned to roles. Authorization is implemented at multiple layers: route protection in the frontend, middleware in the backend, and data-level access control in the database layer.

## Role-based access control (RBAC)

### What It Is

RBAC is an authorization model where access rights are assigned to roles rather than individual users. Users are then assigned to one or more roles. This approach simplifies permission management by grouping permissions into logical roles.

```javascript
// Basic RBAC model
const ROLES = {
  ADMIN: 'admin',
  MODERATOR: 'moderator',
  USER: 'user',
  GUEST: 'guest'
}

const PERMISSIONS = {
  READ_POST: 'read:post',
  CREATE_POST: 'create:post',
  UPDATE_POST: 'update:post',
  DELETE_POST: 'delete:post',
  MANAGE_USERS: 'manage:users',
  MANAGE_SETTINGS: 'manage:settings'
}

const rolePermissions = {
  [ROLES.ADMIN]: Object.values(PERMISSIONS),
  [ROLES.MODERATOR]: [
    PERMISSIONS.READ_POST,
    PERMISSIONS.UPDATE_POST,
    PERMISSIONS.DELETE_POST
  ],
  [ROLES.USER]: [
    PERMISSIONS.READ_POST,
    PERMISSIONS.CREATE_POST,
    PERMISSIONS.UPDATE_POST
  ],
  [ROLES.GUEST]: [
    PERMISSIONS.READ_POST
  ]
}
```

### Why It Is Important

RBAC provides a scalable, maintainable authorization model. Adding or removing permissions for a group of users becomes a matter of modifying a role definition rather than updating individual user records. It follows the principle of least privilege, ensuring users only have the permissions they need.

### How It Works Internally

When a user makes a request, the system identifies the user and their role(s), looks up the permissions associated with those roles, and checks whether the required permission for the requested action is in the set of granted permissions. This check can happen at multiple levels: route middleware, service layer, and data access layer.

```javascript
// Authorization flow:
// 1. Request arrives with authentication token
// 2. Extract user identity and roles from token
// 3. Determine required permission for the request
// 4. Look up permissions for user's roles
// 5. Check if required permission is in user's permission set
// 6. Grant or deny access
```

### Syntax

```javascript
// Role definition
const roleHierarchy = {
  admin: ['user', 'moderator'],
  moderator: ['user'],
  user: [],
  guest: []
}

// Permission check function
function hasPermission(user, requiredPermission) {
  return user.permissions.includes(requiredPermission)
}

// Role check function
function hasRole(user, role) {
  return user.roles.includes(role)
}

// Inheritance-aware check
function hasPermissionWithInheritance(user, requiredPermission) {
  const inheritedRoles = user.roles.flatMap(role => {
    return [...(roleHierarchy[role] || []), role]
  })

  const allPermissions = [...new Set(inheritedRoles)].flatMap(role => {
    return rolePermissions[role] || []
  })

  return allPermissions.includes(requiredPermission)
}
```

### Beginner Examples

```javascript
// Simple role check in frontend
function canDeletePost(user, post) {
  if (user.role === 'admin') return true
  if (user.role === 'moderator') return true
  if (user.id === post.authorId && user.role === 'user') return true
  return false
}

// Conditional rendering based on role
function AdminPanel({ user }) {
  if (user.role !== 'admin') {
    return <p>Access denied. Admin only.</p>
  }

  return (
    <div>
      <h2>Admin Panel</h2>
      <UserManagement />
      <SystemSettings />
    </div>
  )
}

// Simple permission check
function checkPermission(user, permission) {
  const perms = {
    admin: ['read', 'write', 'delete', 'manage'],
    editor: ['read', 'write'],
    viewer: ['read']
  }

  return perms[user.role]?.includes(permission) ?? false
}

console.log(checkPermission({ role: 'editor' }, 'delete')) // false
```

### Intermediate Examples

```javascript
// Complete RBAC system
class RBAC {
  constructor() {
    this.roles = {}
    this.users = {}
  }

  defineRole(name, permissions = [], inherits = []) {
    this.roles[name] = {
      name,
      permissions: new Set(permissions),
      inherits
    }
  }

  assignRole(userId, role) {
    if (!this.users[userId]) {
      this.users[userId] = { roles: new Set() }
    }
    this.users[userId].roles.add(role)
  }

  getEffectivePermissions(userId) {
    const user = this.users[userId]
    if (!user) return new Set()

    const permissions = new Set()
    const visited = new Set()

    const resolveRole = (roleName) => {
      if (visited.has(roleName)) return
      visited.add(roleName)

      const role = this.roles[roleName]
      if (!role) return

      role.permissions.forEach(p => permissions.add(p))
      role.inherits.forEach(parentRole => resolveRole(parentRole))
    }

    user.roles.forEach(r => resolveRole(r))
    return permissions
  }

  can(userId, ...requiredPermissions) {
    const userPermissions = this.getEffectivePermissions(userId)
    return requiredPermissions.every(p => userPermissions.has(p))
  }

  canAny(userId, ...requiredPermissions) {
    const userPermissions = this.getEffectivePermissions(userId)
    return requiredPermissions.some(p => userPermissions.has(p))
  }
}

// Usage
const rbac = new RBAC()
rbac.defineRole('guest', ['read:public'])
rbac.defineRole('user', ['read:profile', 'update:profile'], ['guest'])
rbac.defineRole('moderator', ['read:all', 'delete:comments'], ['user'])
rbac.defineRole('admin', ['manage:all'], ['moderator'])

rbac.assignRole('user1', 'user')
rbac.assignRole('user2', 'moderator')

console.log(rbac.can('user1', 'read:public'))      // true
console.log(rbac.can('user2', 'delete:comments'))  // true
console.log(rbac.can('user1', 'delete:comments'))  // false

// Permission-based UI rendering
class PermissionGate {
  constructor(rbac, userId) {
    this.rbac = rbac
    this.userId = userId
  }

  can(...permissions) {
    return this.rbac.can(this.userId, ...permissions)
  }

  canAny(...permissions) {
    return this.rbac.canAny(this.userId, ...permissions)
  }

  ifCan(permissions, component, fallback = null) {
    return this.can(...(Array.isArray(permissions) ? permissions : [permissions]))
      ? component
      : fallback
  }
}

// React example
function App() {
  const gate = new PermissionGate(rbac, currentUser.id)

  return (
    <div>
      {gate.ifCan('read:profile', <UserProfile />)}
      {gate.ifCan('manage:all', <AdminPanel />, <p>Contact admin for access</p>)}
      {gate.canAny('update:profile', 'manage:all') && <EditProfileButton />}
    </div>
  )
}
```

### Advanced Examples

```javascript
// Attribute-Based Access Control (ABAC) - more granular than RBAC
class ABACEngine {
  constructor() {
    this.policies = []
  }

  addPolicy(name, condition, effect = 'allow') {
    this.policies.push({ name, condition, effect })
  }

  evaluate(user, resource, action, context = {}) {
    for (const policy of this.policies) {
      const result = policy.condition({ user, resource, action, context })
      if (result !== undefined) {
        return policy.effect === 'allow'
      }
    }
    return false // Deny by default
  }
}

// Usage
const abac = new ABACEngine()

// Policy: Users can edit their own posts
abac.addPolicy('own-post-edit', ({ user, resource, action }) => {
  if (action === 'edit' && resource.type === 'post') {
    return resource.authorId === user.id
  }
})

// Policy: Admins can edit any post
abac.addPolicy('admin-edit-all', ({ user, action }) => {
  if (action === 'edit' && user.role === 'admin') {
    return true
  }
})

// Policy: Premium users can access premium content
abac.addPolicy('premium-access', ({ user, resource }) => {
  if (resource.tier === 'premium') {
    return user.subscription === 'premium'
  }
})

// Express authorization middleware with RBAC + ABAC
function createAuthorizationMiddleware(rbac, abac) {
  return function authorize(action, resourceType) {
    return async (req, res, next) => {
      const user = req.user
      const resource = req.resource || {}
      const context = {
        ip: req.ip,
        userAgent: req.get('User-Agent'),
        timestamp: Date.now()
      }

      // Check ABAC first
      if (abac) {
        const abacResult = abac.evaluate(user, { ...resource, type: resourceType }, action, context)
        if (abacResult) return next()
        if (abacResult === false) {
          return res.status(403).json({ message: 'Access denied by policy' })
        }
      }

      // Fall back to RBAC
      const requiredPermission = `${action}:${resourceType}`
      if (rbac.can(user.id, requiredPermission)) {
        return next()
      }

      return res.status(403).json({ message: 'Insufficient permissions' })
    }
  }
}

// Complete authorization middleware suite
class AuthorizationMiddleware {
  constructor(rbac, options = {}) {
    this.rbac = rbac
    this.options = {
      userProperty: 'user',
      ...options
    }
  }

  requireRole(...roles) {
    return (req, res, next) => {
      const user = req[this.options.userProperty]
      if (!user) {
        return res.status(401).json({ message: 'Authentication required' })
      }

      const hasRole = roles.some(role => user.roles.includes(role))
      if (!hasRole) {
        return res.status(403).json({
          message: `Required role: ${roles.join(' or ')}`,
          required: roles,
          userRoles: user.roles
        })
      }

      next()
    }
  }

  requirePermission(...permissions) {
    return (req, res, next) => {
      const user = req[this.options.userProperty]
      if (!user) {
        return res.status(401).json({ message: 'Authentication required' })
      }

      const hasPermission = permissions.every(p => user.permissions.includes(p))
      if (!hasPermission) {
        return res.status(403).json({
          message: `Missing required permissions`,
          required: permissions
        })
      }

      next()
    }
  }

  requireOwnership(resourceFetcher) {
    return async (req, res, next) => {
      const user = req[this.options.userProperty]
      if (!user) {
        return res.status(401).json({ message: 'Authentication required' })
      }

      // Admins bypass ownership checks
      if (user.roles.includes('admin')) {
        return next()
      }

      try {
        const resource = await resourceFetcher(req)
        if (!resource) {
          return res.status(404).json({ message: 'Resource not found' })
        }

        if (resource.userId !== user.id && resource.authorId !== user.id) {
          return res.status(403).json({ message: 'Access denied: not the owner' })
        }

        req.resource = resource
        next()
      } catch (error) {
        next(error)
      }
    }
  }

  scopeQuery(userProperty) {
    return (req, res, next) => {
      const user = req[this.options.userProperty]
      if (!user) return next()

      // Admins see all; regular users only see their own data
      if (!user.roles.includes('admin')) {
        req.scopedQuery = {
          ...req.query,
          userId: user.id
        }
      }

      next()
    }
  }
}

// React Router protected route component
function ProtectedRoute({ user, requiredPermissions, requiredRoles, children }) {
  // Check roles
  if (requiredRoles) {
    const hasRole = requiredRoles.some(role => user?.roles?.includes(role))
    if (!hasRole) {
      return <Navigate to="/unauthorized" replace />
    }
  }

  // Check permissions
  if (requiredPermissions) {
    const hasPermission = requiredPermissions.every(p => user?.permissions?.includes(p))
    if (!hasPermission) {
      return <Navigate to="/unauthorized" replace />
    }
  }

  return children
}

// Usage in routes
function AppRoutes() {
  return (
    <Routes>
      <Route path="/posts" element={<PostList />} />
      <Route path="/posts/new" element={
        <ProtectedRoute requiredPermissions="create:post">
          <CreatePost />
        </ProtectedRoute>
      } />
      <Route path="/admin" element={
        <ProtectedRoute requiredRoles={['admin']}>
          <AdminDashboard />
        </ProtectedRoute>
      } />
      <Route path="/unathorized" element={<UnauthorizedPage />} />
    </Routes>
  )
}
```

### Real-World Use Cases

- **SaaS applications**: Different subscription tiers get different feature access
- **Enterprise dashboards**: Role-based views for executives, managers, and staff
- **Content management systems**: Editor, author, and admin roles
- **Healthcare systems**: HIPAA-compliant access control (doctors, nurses, admins)
- **Banking applications**: Teller, manager, and auditor roles
- **E-commerce platforms**: Customer, seller, and platform admin roles
- **Developer tools**: Read-only vs write access to API resources

### Common Mistakes

```javascript
// Mistake: Only checking authorization on the frontend
function deletePost(postId) {
  // Frontend hides delete button for non-admins
  // But an attacker can still call the API!
  return fetch(`/api/posts/${postId}`, { method: 'DELETE' })
}
// Always authorize on the backend too!

// Mistake: Not using the principle of least privilege
// Giving admin to users who only need editor access

// Mistake: Hard-coding user IDs in authorization logic
// Creates maintenance nightmare

// Mistake: Not considering role hierarchy
// Checking for 'admin' but missing that 'superadmin' might exist

// Mistake: Client-side role manipulation
// Never trust client-provided role information

// Mistake: Missing authorization in service layer
// Only checking at route level, not when services call each other
```

### Best Practices

```javascript
// 1. Defense in depth - authorize at every layer
// Frontend: conditional rendering
// Route: middleware check
// Service: permission verification
// Database: row-level security

// 2. Use principle of least privilege
const userPermissions = ['read:own_profile', 'update:own_profile']

// 3. Centralize authorization logic
class AuthorizationService {
  constructor(rbac) {
    this.rbac = rbac
  }

  can(userId, action, resource) {
    return this.rbac.can(userId, `${action}:${resource.type}`)
  }
}

// 4. Fail closed (deny by default)
function authorize(user, required) {
  if (!user) return false
  if (!required) return true
  return required.every(p => user.permissions.includes(p))
}

// 5. Log authorization failures
function auditLog(userId, action, resource, granted) {
  console.log({
    timestamp: new Date().toISOString(),
    userId,
    action,
    resource,
    granted,
    ip: getClientIP()
  })
}

// 6. Cache permission lookups
const permissionCache = new Map()
function getCachedPermissions(userId) {
  if (!permissionCache.has(userId)) {
    permissionCache.set(userId, computePermissions(userId))
    setTimeout(() => permissionCache.delete(userId), 300000) // 5 min TTL
  }
  return permissionCache.get(userId)
}
```

### Performance Considerations

- Cache resolved permissions to avoid recomputing on every request
- Use in-memory caches (Redis) for distributed systems
- Minimize database queries in authorization middleware
- Consider pre-computing flattened permission sets for each user
- Use efficient data structures (Sets for permission lookups, not arrays)
- Batch authorization checks when processing multiple resources
- Permission inheritance resolution adds overhead; flatten at role creation time
- Use bitfield-based permissions for high-performance scenarios

### Interview Questions

**Q: What is the difference between RBAC and ABAC?**

A: RBAC assigns permissions to roles and users to roles. It's simpler but less flexible. ABAC evaluates policies based on attributes of the user, resource, action, and environment (context). ABAC is more granular and dynamic but more complex to implement. RBAC is often sufficient for most applications, while ABAC is useful for complex compliance requirements.

**Q: How do you handle authorization in a microservices architecture?**

A: Options include a centralized authorization service that all services query, embedding permission claims in JWT tokens, using API gateways for pre-authorization, or distributed policy evaluation (each service checks permissions against a shared policy store). JWTs with embedded claims are common but require careful management of token size and revocation.

**Q: How do you implement row-level security in a database?**

A: Row-level security restricts which rows a user can access. Implementations include WHERE clauses that filter by user ID/tenant ID (most common), PostgreSQL Row-Level Security policies, or view-based access (creating views with built-in filters). The key is to never trust the client to specify which rows they can access.

### Coding Challenges

```javascript
// Challenge 1: Implement a permission resolver with role inheritance
function resolvePermissions(role, roleDefs) {
  const permissions = new Set()
  const visited = new Set()

  function traverse(roleName) {
    if (visited.has(roleName)) return
    visited.add(roleName)

    const def = roleDefs[roleName]
    if (!def) return

    def.permissions.forEach(p => permissions.add(p))
    def.inherits.forEach(parent => traverse(parent))
  }

  traverse(role)
  return [...permissions]
}

// Challenge 2: Protected route HOC
function withAuthorization(WrappedComponent, requiredPermissions) {
  return function AuthenticatedComponent(props) {
    const user = useUser()
    const hasAccess = requiredPermissions.every(p => user?.permissions?.includes(p))

    if (!user) return <LoginPrompt />
    if (!hasAccess) return <ForbiddenPage />
    return <WrappedComponent {...props} />
  }
}

// Challenge 3: Attribute-based policy evaluator
function evaluatePolicy(policy, user, resource, action) {
  // Policy format: { effect, condition }
  const { effect, condition } = policy

  const matches = Object.entries(condition).every(([key, value]) => {
    switch (key) {
      case 'user.role': return user.role === value
      case 'user.department': return user.department === value
      case 'resource.owner': return resource[value] === user.id
      case 'resource.type': return resource.type === value
      case 'resource.tier': return value.includes(resource.tier)
      case 'time.range': {
        const hour = new Date().getHours()
        return hour >= value[0] && hour < value[1]
      }
      default: return false
    }
  })

  return matches ? effect === 'allow' : null
}
```

### Related Topics

- `Authentication` - Prerequisite for authorization decisions
- `JWT` - Embedding roles/permissions in tokens
- `Middleware` - Express.js middleware for authorization
- `Protected Routes` - Frontend route guards
- `OWASP` - Authorization security best practices
- `OAuth2` - Scoped authorization for third-party apps
- `GraphQL` - Field-level authorization in resolvers
- `Testing` - Authorization unit and integration tests
