---
description: "Commit message generation guidelines"
applyTo: "**"
---

# Commit Message Guidelines

## Format

Use conventional commit format with clear, concise descriptions:

```
type(scope): brief description

Optional detailed explanation of what and why.
Optional breaking change note.
```

## Types

- **feat**: New feature for the user
- **fix**: Bug fix for the user
- **security**: Security-related changes (vulnerabilities, authentication, etc.)
- **docs**: Documentation changes
- **style**: Code formatting, missing semicolons, etc.
- **refactor**: Code restructuring without functionality changes
- **test**: Adding or updating tests
- **chore**: Maintenance tasks, dependency updates

## Scopes (when applicable)

- **auth**: Authentication/authorization changes
- **ui**: User interface components
- **api**: API-related changes
- **crypto**: Cryptography/password handling
- **storage**: Local storage functionality
- **validation**: Input validation
- **config**: Configuration changes

## Examples

- `feat(auth): implement password strength validation`
- `fix(crypto): resolve password encryption vulnerability`
- `security(storage): encrypt local storage data`
- `docs(readme): update installation instructions`
- `refactor(ui): simplify password entry form`

## Security Focus

For this password manager application:

- Always use `security:` type for security-related changes
- Explain the security impact in the body
- Reference any security standards or OWASP guidelines addressed
- Mention if the change affects password storage, encryption, or authentication

## Best Practices

- Keep first line under 50 characters
- Use imperative mood ("add" not "added")
- Don't end with a period
- Include issue/ticket references: `fixes #123`
- Explain **what** and **why**, not **how**
