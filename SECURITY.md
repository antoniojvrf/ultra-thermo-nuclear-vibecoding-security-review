# Security Policy

## Scope

This repository contains documentation and prompt instructions — not executable code. There is no traditional attack surface (no running server, no dependencies to exploit, no user data).

The main security concerns for this project are:
- **Incorrect security advice**: A check or fix that is itself insecure or misleading
- **False negatives**: Missing a vulnerability pattern that AI assistants commonly introduce

## Reporting Incorrect Security Guidance

If you find that this skill gives incorrect, incomplete, or actively harmful security advice:

1. **Open a GitHub Issue** describing:
   - The file and section containing the incorrect guidance
   - What the correct guidance should be
   - Why the current guidance is wrong or misleading
   - (If applicable) A reference to authoritative documentation

2. We'll review and update the affected file(s) promptly.

## Out of Scope

- Vulnerabilities in third-party tools or frameworks *mentioned* in this skill
- Theoretical concerns about prompt injection into the skill itself
- Feature requests (use Issues for those)

## Acknowledgements

Contributors who identify and report incorrect security guidance will be credited in the relevant commit messages and release notes.
