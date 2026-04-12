# Contributing to VibeCoding Security Skill

Thank you for helping make AI-assisted development more secure! 🛡️

## How to Contribute

### 1. Adding a New Vulnerability Pattern

If you've found a vulnerability pattern that AI assistants commonly introduce and isn't covered, open a PR with:

- **Which file to update**: Add to the relevant `vulnerabilities/*.md` or `stack/*.md` file
- **Real-world example**: Show the vulnerable code pattern
- **Exploit scenario**: Explain what an attacker could do (be concrete, not abstract)
- **Fix**: Show the secure version as a before/after diff
- **Add to checklist**: If it warrants a new check, add it to the appropriate `checklists/*.md` and `core/audit-process.md`

### 2. Adding a New Stack Module

If there's a popular framework or service not yet covered (e.g., PlanetScale, Prisma-specific patterns, Clerk, etc.):

1. Create `stack/<name>.md` following the existing structure
2. Update `SKILL.md` to reference the new file in the References Index
3. Add stack detection in the tech stack detection table in `SKILL.md`
4. Add relevant checks to `core/audit-process.md`

### 3. Improving an Existing Module

- Keep code examples minimal — show only what illustrates the vulnerability
- Always show ❌ vulnerable pattern before ✅ secure pattern
- Add language/framework labels to code fences (e.g., ` ```typescript `)
- Keep bypass technique tables accurate and sourced from real-world exploits

### 4. Fixing an Error

Found outdated information, an incorrect fix, or a better secure pattern? Open a PR with what changed and why.

---

## File Structure Conventions

```
vulnerabilities/     # Generic vulnerability classes (not framework-specific)
stack/               # Framework/service-specific patterns
production/          # Production readiness and compliance
checklists/          # Executable checklists that reference the above
core/                # Core orchestration (don't change without discussion)
```

## Pull Request Checklist

- [ ] Vulnerable example is actually exploitable (not theoretical)
- [ ] Fix is actually secure (not just slightly less vulnerable)
- [ ] Code examples have language labels
- [ ] New checks are added to `core/audit-process.md` and the right `checklists/*.md`
- [ ] No personally identifiable information in examples
- [ ] Consistent formatting with existing files (headers, tables, checklist at end)

## Code of Conduct

- Be constructive and specific — "this pattern is vulnerable because X can do Y" not "this is wrong"
- Credit sources when adding content from bug bounty writeups or research
- Don't include actual exploit payloads that could harm real systems

## Issues

Use GitHub Issues to:
- Report false positives (a check that flags safe code)
- Report false negatives (a vulnerability pattern we're missing)
- Suggest new stack modules
- Discuss architectural changes to the skill

For security issues with the skill itself, open a regular issue — this is a documentation/prompt skill, not executable code.

---

Thank you for contributing! Every addition makes vibe-coded apps a little safer. 🚀
