# Security Policy

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

Email: shauryapunj404@gmail.com
Subject: `[Newspapering SECURITY] <brief description>`

Provide repro + impact + suggested fix. Acknowledgement within 48 hours. GitHub's "Security › Report a vulnerability" tab is also accepted.

## Security Controls

This repository contains technical design documents (Markdown and PDF) and no executable application source. Controls are scoped accordingly.

- Secret scanning and push protection enabled at the repository level.
- Dependabot security updates enabled; weekly version updates configured for GitHub Actions in `.github/dependabot.yml`.
- Branch protection on `main`: no force-push, no deletion, with required status checks kept strict.
- CodeQL code scanning is intentionally not configured because the repository holds no CodeQL-supported source code (Markdown and PDF only).
