# Access Control Vulnerabilities

## IDOR (Insecure Direct Object Reference)

### What It Is
User A can access User B's resources by changing an ID in the request.

### Vulnerable Pattern
```javascript
// VULNERABLE — no ownership check
app.get('/api/orders/:id', async (req, res) => {
  const order = await db.orders.findUnique({ where: { id: req.params.id } });
  res.json(order); // Anyone can read any order!
});

// SECURE — verify ownership
app.get('/api/orders/:id', authMiddleware, async (req, res) => {
  const order = await db.orders.findUnique({
    where: { id: req.params.id, userId: req.user.id } // ownership check
  });
  if (!order) return res.status(404).json({ error: 'Not found' }); // 404 not 403
  res.json(order);
});
```

### Key Rules
1. **Always verify ownership** on every data access — never trust client-supplied IDs
2. **Return 404 (not 403)** for unauthorized access — prevents resource enumeration
3. **Use UUIDs** instead of sequential IDs — makes guessing harder (defense in depth)
4. **Check parent resource ownership** — if accessing a comment, verify user owns the parent post
5. **Re-validate permissions** after any privilege change

### Authorization Checklist
- [ ] Verify user owns the resource on every request
- [ ] Check organization membership for multi-tenant apps
- [ ] Validate role permissions for role-based actions
- [ ] Re-validate permissions after privilege changes
- [ ] Check parent resource ownership for nested resources

---

## Privilege Escalation

### Horizontal Escalation
User A accessing User B's resources at the same privilege level.

### Vertical Escalation
Regular user accessing admin functionality.

### Vulnerable Patterns
```javascript
// VULNERABLE — role from client
app.post('/api/users', async (req, res) => {
  const user = await db.users.create({
    data: { ...req.body } // attacker sends { role: "admin" }
  });
});

// SECURE — server-side role assignment
app.post('/api/users', async (req, res) => {
  const { name, email } = req.body; // whitelist fields
  const user = await db.users.create({
    data: { name, email, role: 'user' } // server controls role
  });
});
```

### Prevention
1. **Never trust role info from client** — validate server-side
2. **Whitelist allowed fields** for creation/update operations
3. **Admin routes must verify admin role** in middleware, not just at the UI level
4. **Audit all endpoints** for missing auth middleware

---

## Mass Assignment

### What It Is
Accepting unfiltered request bodies allows users to modify fields they shouldn't.

### Vulnerable Pattern
```javascript
// VULNERABLE — user can set { role: "admin", verified: true }
app.put('/api/profile', async (req, res) => {
  await db.users.update({ where: { id: req.user.id }, data: req.body });
});

// SECURE — whitelist allowed fields
app.put('/api/profile', async (req, res) => {
  const allowed = ['name', 'email', 'avatar', 'bio'];
  const updates = {};
  for (const key of allowed) {
    if (req.body[key] !== undefined) updates[key] = req.body[key];
  }
  await db.users.update({ where: { id: req.user.id }, data: updates });
});
```

### This Applies To
- Any ORM/framework — always explicitly define which fields a request can modify
- REST APIs, GraphQL mutations, Server Actions
- Nested objects (user can modify nested relations)

### Fields to Never Accept From Client
- `role`, `isAdmin`, `permissions`
- `verified`, `emailVerified`
- `subscriptionTier`, `plan`
- `createdAt`, `updatedAt`
- `id`, `userId` (for ownership)
- `password` (only through dedicated change-password flow)
- `balance`, `credits`

---

## Implementation Pattern — Secure Resource Access

```
function getResource(resourceId, currentUser):
    resource = database.find(resourceId)

    if resource is null:
        return 404  // Don't reveal if resource exists

    if resource.ownerId != currentUser.id:
        if not currentUser.hasOrgAccess(resource.orgId):
            return 404  // Return 404, not 403, to prevent enumeration

    return resource
```

### Multi-Tenant Isolation
```javascript
// SECURE — tenant isolation at query level
async function getOrgData(orgId, userId) {
  // First verify user belongs to org
  const membership = await db.orgMembers.findUnique({
    where: { orgId_userId: { orgId, userId } }
  });
  if (!membership) throw new Error('Not found');

  // Then query scoped to org
  return db.projects.findMany({ where: { orgId } });
}
```
