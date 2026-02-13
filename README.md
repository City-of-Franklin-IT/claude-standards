# Claude Standards

Coding standards and conventions for use with [Claude Code](https://docs.anthropic.com/en/docs/claude-code). These standards are referenced via `CLAUDE.md` to ensure consistent code generation across projects.

## Standards

| Standard | Description |
|---|---|
| [TypeScript](typescript/TYPESCRIPT_STANDARDS.md) | Code style, type definitions, naming conventions, API patterns, and path aliases |
| [React](react/REACT_STANDARDS.md) | Component structure, folder organization, state management, forms, error handling, and import ordering |
| [Git](git/GIT_STANDARDS.md) | Deployment scripts, version control workflow, and `.gitignore` template |

## Usage

Reference this repo from your `CLAUDE.md`:

```
Use standards defined in /opt/claude-standards when working with codebases
```

Claude Code will then follow these standards when generating or modifying code in your projects.
