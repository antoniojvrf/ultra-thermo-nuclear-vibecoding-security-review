# Docker & CI/CD Security

## Container Security

### Running as Non-Root
```dockerfile
# ❌ VULNERABLE — running as root
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# ✅ SECURE — non-root user
FROM node:20-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:20-slim
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /app .
USER appuser
EXPOSE 3000
CMD ["node", "server.js"]
```

### Secrets in Docker
```dockerfile
# ❌ CRITICAL — secrets in Dockerfile or image layers
ENV DATABASE_URL=postgresql://user:password@host/db
COPY .env /app/.env
RUN echo "API_KEY=sk_live_xxx" >> /app/config

# ✅ SECURE — inject at runtime
# Use environment variables, Docker secrets, or mounted volumes
# docker run -e DATABASE_URL=... myapp
# Or Docker secrets: docker secret create db_url ./db_url.txt
```

### Image Security
```dockerfile
# ✅ Use specific version tags (not :latest)
FROM node:20.11.0-slim

# ✅ Use distroless or slim images (smaller attack surface)
FROM gcr.io/distroless/nodejs20-debian12

# ✅ Multi-stage build (don't ship dev dependencies)
FROM node:20-slim AS builder
RUN npm ci
RUN npm run build

FROM node:20-slim
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
```

### .dockerignore
```
.git
.env
.env.*
node_modules
*.md
.github
tests
coverage
```

## CI/CD Pipeline Security

### GitHub Actions — Permissions
```yaml
# ❌ VULNERABLE — default permissions too broad
name: Deploy
on: push

# ✅ SECURE — minimal permissions
name: Deploy
on: push
permissions:
  contents: read  # minimum needed
  
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write  # only if using OIDC
```

### Secrets in CI/CD
```yaml
# ❌ VULNERABLE — secrets in logs
- run: echo "Deploying with key ${{ secrets.DEPLOY_KEY }}"
- run: curl -H "Authorization: Bearer ${{ secrets.API_KEY }}" https://api.com

# ✅ SECURE — mask secrets, use environment
- run: deploy.sh
  env:
    DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
    
# ❌ VULNERABLE — using pull_request_target with checkout of PR code
on: pull_request_target
steps:
  - uses: actions/checkout@v4
    with:
      ref: ${{ github.event.pull_request.head.sha }}  # attacker controls this code!
  - run: npm install  # runs attacker's postinstall script with secrets access
```

### Dependency Pinning
```yaml
# ❌ VULNERABLE — unpinned actions
- uses: actions/checkout@main  # could be compromised

# ✅ SECURE — pin to commit SHA
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
```

## Checklist
- [ ] Containers run as non-root user
- [ ] No secrets in Dockerfile or image layers
- [ ] Base images use specific version tags (not :latest)
- [ ] Multi-stage builds exclude dev dependencies
- [ ] .dockerignore excludes .env, .git, node_modules
- [ ] GitHub Actions use minimal `permissions`
- [ ] Secrets not echoed/logged in CI output
- [ ] Actions pinned to commit SHA (not tag/branch)
- [ ] `pull_request_target` not used with PR code checkout
- [ ] Container images scanned for vulnerabilities (Trivy, Snyk)
