# GraphQL Security

## Introspection in Production

```javascript
// ❌ VULNERABLE — introspection reveals entire API schema
const server = new ApolloServer({ typeDefs, resolvers });

// ✅ SECURE — disable in production
const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: process.env.NODE_ENV !== 'production',
});
```

## Query Depth Limiting

```javascript
// ❌ VULNERABLE — unlimited depth allows resource exhaustion
// Attack query: { user { friends { friends { friends { ... } } } } }

// ✅ SECURE — limit query depth
import depthLimit from 'graphql-depth-limit';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(10)], // max 10 levels
});
```

## Query Complexity / Cost Analysis

```javascript
// ✅ Limit total query cost
import { createComplexityRule, simpleEstimator } from 'graphql-query-complexity';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityRule({
      maximumComplexity: 1000,
      estimators: [simpleEstimator({ defaultComplexity: 1 })],
      onComplete: (complexity) => {
        if (complexity > 500) console.warn(`High complexity query: ${complexity}`);
      },
    }),
  ],
});
```

## Batching Attack Prevention

```javascript
// ❌ VULNERABLE — unlimited batching
// Attacker sends 1000 queries in single request for brute force

// ✅ SECURE — limit batch size
const server = new ApolloServer({
  typeDefs,
  resolvers,
  allowBatchedHttpRequests: false, // disable entirely
  // Or limit: plugins that cap batch size
});
```

## Authorization in Resolvers

```javascript
// ❌ VULNERABLE — no auth check in resolver
const resolvers = {
  Query: {
    user: (_, { id }) => db.users.findUnique({ where: { id } }),
  },
};

// ✅ SECURE — check auth and ownership
const resolvers = {
  Query: {
    user: (_, { id }, context) => {
      if (!context.user) throw new AuthenticationError('Must be logged in');
      if (context.user.id !== id && context.user.role !== 'admin') {
        throw new ForbiddenError('Access denied');
      }
      return db.users.findUnique({ where: { id } });
    },
  },
  Mutation: {
    updateUser: (_, { id, input }, context) => {
      if (!context.user || context.user.id !== id) {
        throw new ForbiddenError('Access denied');
      }
      const allowed = ['name', 'email', 'bio']; // whitelist fields
      const data = Object.fromEntries(
        Object.entries(input).filter(([key]) => allowed.includes(key))
      );
      return db.users.update({ where: { id }, data });
    },
  },
};
```

## Checklist
- [ ] Introspection disabled in production
- [ ] Query depth limiting configured (max 10 levels)
- [ ] Query complexity/cost analysis with maximum limit
- [ ] Batching disabled or limited
- [ ] Authentication checked in all resolvers
- [ ] Authorization (ownership) checked for data access
- [ ] Field-level whitelisting on mutations
- [ ] Rate limiting on GraphQL endpoint
- [ ] Persisted queries considered for production
