# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability, please **do not** open a public issue.

Instead, please report it privately by:

1. Emailing the maintainer directly
2. Using GitHub's private vulnerability reporting feature

We take security seriously and will respond within 48 hours.

## Credential Handling

This project handles WeChat Official Account API credentials. Please ensure:

- Never commit real API keys or secrets to the repository
- Use `data/credentials.yaml` for local configuration (already gitignored)
- Keep `data/credentials.yaml` outside version control at all times
