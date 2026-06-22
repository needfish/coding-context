# Project Conventions

## Git Conventions

- Commit messages: `<scope>: <description>` (50 chars or fewer). Body (72 chars or fewer per line) explains why, not what. Use trailers (`Closes #issue`).
- Branch names: `scope/description`.
- One logical change per commit. Squash WIP before push.
- Rename freely as understanding deepens. Never ship a placeholder.
- Exclude secrets, `.env`, credentials, and large generated artifacts.

## Dependencies

Every README includes a Dependencies table with Package, Version, Purpose, and Docs columns:

```markdown
## Dependencies
| Package | Version | Purpose | Docs |
|---------+---------+---------+------|
```

- Use `^` for semver-aware packages. Use `====` for exact versions.
- Keep the table updated when deps change.
- Before adding a new dependency, state the package name and purpose and ask for confirmation.

## README Conventions

After the project title, include:

```markdown
> **AI-generated code notice:** This project was written with the assistance of a large language model (DeepSeek V4 Flash).
```

Update the model name to match.
