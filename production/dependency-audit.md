# Dependency Security & Supply Chain

## Ghost Packages / Typosquatting

### What It Is
Attackers publish packages with names similar to popular packages:
- `lodash` → `1odash`, `lodahs`, `lodashs`
- `express` → `expresss`, `expres`
- `react` → `reactjs`, `react-native-community` (fake scopes)

### Prevention
```bash
# ✅ Verify package before installing
# Check: downloads, publish date, author, GitHub link
npm info <package-name>

# ✅ Use lockfiles (commit them!)
# package-lock.json, yarn.lock, pnpm-lock.yaml

# ✅ Audit dependencies
npm audit
yarn audit
pnpm audit
```

## Known Vulnerabilities (CVEs)

```bash
# ✅ Regular vulnerability scanning
npm audit                          # Built-in
npx audit-ci --critical            # CI/CD integration
snyk test                          # Snyk
trivy fs .                         # Trivy (also does containers)

# ✅ Automated dependency updates
# GitHub Dependabot, Renovate Bot, Snyk
```

### Dependabot Configuration
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "security-team"
```

## Supply Chain Attacks

### npm postinstall Scripts
```json
// ❌ RISK — packages can run arbitrary code on install
{
  "scripts": {
    "postinstall": "node malicious-script.js" // runs automatically!
  }
}
```

```bash
# ✅ Prevent auto-running scripts
npm install --ignore-scripts
# Then run scripts manually for packages you trust
npm rebuild
```

### Dependency Confusion
Attackers publish public packages matching internal/private package names.

```json
// ✅ Prevention — scope your packages
{
  "name": "@your-org/internal-package",
  "publishConfig": {
    "registry": "https://your-private-registry.com"
  }
}

// .npmrc — pin scoped packages to private registry
@your-org:registry=https://your-private-registry.com
```

## Lockfile Integrity

```bash
# ✅ Install from lockfile only (CI/CD)
npm ci                    # instead of npm install
yarn install --frozen-lockfile
pnpm install --frozen-lockfile

# ✅ Detect lockfile tampering
# Git hooks or CI checks to verify lockfile matches package.json
```

## License Compliance

```bash
# ✅ Check licenses of all dependencies
npx license-checker --summary
npx license-checker --failOn "GPL-3.0;AGPL-3.0"  # block copyleft in commercial
```

## Checklist
- [ ] Lockfiles committed to repository
- [ ] `npm ci` used in CI/CD (not `npm install`)
- [ ] `npm audit` runs in CI pipeline
- [ ] Dependabot or Renovate configured for auto-updates
- [ ] New dependencies reviewed before adding (downloads, author, age)
- [ ] No unnecessary dependencies (evaluate alternatives)
- [ ] `--ignore-scripts` used for untrusted packages
- [ ] Scoped packages used for internal packages
- [ ] License compliance checked
- [ ] Dependencies periodically pruned (unused packages removed)
