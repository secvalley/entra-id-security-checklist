# Contributing to Entra ID Security Checklist

Contributions are welcome. If you have found a control that should be on this list, or if a Microsoft docs link has gone stale, open a PR or file an issue.

## How to Contribute

1. Fork the repository
2. Find the appropriate category in README.md
3. Add your control following the existing format
4. Submit a pull request

## Control Format

```markdown
- [ ] **[Critical|High|Medium|Low]** Description of what to check, why it matters, and what to do about it.
  - [Relevant Microsoft Docs](https://learn.microsoft.com/...)
```

## Guidelines

- Keep entries practical and grounded in real-world Entra ID configuration
- Every control should be something an admin can actually implement
- Include a link to the relevant Microsoft documentation
- One PR per logical change

## Severity Levels

| Level | Meaning |
|-------|---------|
| **Critical** | Direct path to tenant compromise |
| **High** | Significant identity security gap |
| **Medium** | Reduces attack surface |
| **Low** | Good identity hygiene |

## Questions?

Open an issue.
