# Compliance & Data Protection

## GDPR Requirements

### Account Deletion
```typescript
// ✅ Complete account deletion flow
async function deleteUserAccount(userId: string) {
  await db.$transaction(async (tx) => {
    // 1. Delete user-generated content (or anonymize)
    await tx.comments.deleteMany({ where: { userId } });
    await tx.posts.deleteMany({ where: { userId } });

    // 2. Delete files from storage
    const files = await tx.files.findMany({ where: { userId } });
    for (const file of files) {
      await storage.delete(file.path);
    }
    await tx.files.deleteMany({ where: { userId } });

    // 3. Cancel subscriptions
    const user = await tx.users.findUnique({ where: { id: userId } });
    if (user?.stripeCustomerId) {
      await stripe.subscriptions.cancel(user.stripeSubscriptionId);
    }

    // 4. Revoke all sessions and tokens
    await tx.sessions.deleteMany({ where: { userId } });
    await tx.refreshTokens.deleteMany({ where: { userId } });
    await tx.apiKeys.deleteMany({ where: { userId } });

    // 5. Delete or anonymize the user record
    await tx.users.delete({ where: { id: userId } });
    // Or anonymize: await tx.users.update({ where: { id: userId }, data: { email: `deleted-${userId}@deleted.local`, name: 'Deleted User', ... } });

    // 6. Log deletion for audit trail
    await tx.auditLog.create({
      data: { action: 'ACCOUNT_DELETED', subjectId: userId, timestamp: new Date() }
    });
  });
}
```

### Data Minimization
- Collect only data you actually need
- Set retention periods for stored data
- Auto-delete expired data

### Consent Management
- Record explicit consent for data processing
- Allow users to withdraw consent
- Don't pre-check consent checkboxes

## Data Export (Right to Portability)
```typescript
async function exportUserData(userId: string) {
  const user = await db.users.findUnique({
    where: { id: userId },
    include: {
      posts: true,
      comments: true,
      profile: true,
    }
  });

  // Return machine-readable format (JSON)
  return {
    account: { name: user.name, email: user.email, createdAt: user.createdAt },
    posts: user.posts.map(p => ({ title: p.title, content: p.content, date: p.createdAt })),
    comments: user.comments,
  };
}
```

## Backup Strategy

### Requirements
- **Automated backups** — daily minimum, hourly for critical data
- **Off-site storage** — backups in different region/provider
- **Encryption** — backups encrypted at rest
- **Testing** — regularly test restore procedures
- **Retention** — define and enforce retention periods

### Database Backup Checklist
- [ ] Automated daily backups configured
- [ ] Point-in-time recovery enabled (if using managed DB)
- [ ] Backups stored in separate region
- [ ] Backups encrypted at rest
- [ ] Restore procedure documented and tested
- [ ] Retention period defined (e.g., 30 days daily, 12 months monthly)
- [ ] Backup monitoring and alerting configured

## Checklist
- [ ] Account deletion fully implemented (all user data removed)
- [ ] Sessions/tokens invalidated on account deletion
- [ ] Subscriptions cancelled on account deletion
- [ ] Data export available for users (GDPR portability)
- [ ] Consent recorded with timestamp
- [ ] Data retention periods defined and enforced
- [ ] Privacy policy up to date
- [ ] Backup strategy documented and tested
- [ ] Audit logs for security-relevant actions
