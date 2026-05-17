# Security Policy

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

Email: shauryapunj404@gmail.com
Subject: `[Newspapering SECURITY] <brief description>`

Provide repro + impact + suggested fix. Acknowledgement within 48 hours. GitHub's "Security › Report a vulnerability" tab is also accepted.

## Security Controls

- CodeQL `security-extended` on every push, PR, and weekly schedule.
- Dependabot weekly security + version updates; `semver-major` version-updates ignored.
- Branch protection on `main`: required CodeQL status check, linear history, no force-push, no deletion, conversation resolution required.
