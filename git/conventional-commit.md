# 📝 Conventional Commits

A standard format for commit messages that makes history readable, enables automated changelogs, and keeps intent clear.

**Spec:** [conventionalcommits.org](https://www.conventionalcommits.org/)

---

## 📐 Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Rules

- Use the **imperative mood** in the subject line: `add feature` not `added feature`
- Keep the subject line **≤ 72 characters**
- Do **not** end the subject with a period
- Separate subject from body with a **blank line**
- Body wraps at **72 characters** per line (optional but recommended)
- Footer uses format `Token: value` (e.g. `BREAKING CHANGE:`, `Fixes #123`)

---

## 🏷️ Commit Types

| Type | When to Use |
|------|-------------|
| `feat` | A new feature for the user |
| `fix` | A bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace — no logic change |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system or external dependencies (npm, pip, docker) |
| `ci` | CI/CD configuration changes |
| `chore` | Maintenance tasks that don't fit other types |
| `revert` | Reverts a previous commit |

---

## 🔍 Scope (Optional)

Scope narrows the change to a specific area of the codebase.

```
feat(auth): add OAuth2 login
fix(api): handle null response from server
docs(readme): add installation steps
```

Common scopes: `auth`, `api`, `ui`, `db`, `config`, `deps`

---

## ✅ Good Examples

### Simple Feature

```
feat: add user profile page
```

### Bug Fix with Scope

```
fix(auth): prevent token refresh loop on 401
```

### Documentation

```
docs: add conventional commit guide
```

### Refactor

```
refactor(api): extract validation into shared module
```

### Breaking Change (in footer)

```
feat(api): rename user endpoint response fields

BREAKING CHANGE: `userName` is now `username` in all API responses.
Clients must update their parsers accordingly.
```

### Breaking Change (with `!` in type)

```
feat(api)!: change authentication to JWT-only
```

### With Issue Reference

```
fix(db): resolve connection pool leak

Closes #42
```

### Multi-line Body

```
fix(nginx): correct upstream timeout configuration

The previous 5s timeout caused 502 errors under load.
Increased to 30s and added keepalive settings.

Fixes #87
```

---

## ❌ Bad Examples

| Bad | Why |
|-----|-----|
| `fixed bug` | No type, not imperative, too vague |
| `Feat: Added login.` | Wrong case, past tense, trailing period |
| `update stuff` | No type, meaningless description |
| `fix: fix the fix` | Circular, no context |
| `WIP` | Not descriptive, no type |

---

## 💥 Breaking Changes

Two ways to signal a breaking change:

**1. `!` after type/scope:**

```
feat(api)!: remove deprecated v1 endpoints
```

**2. Footer:**

```
feat(api): remove deprecated v1 endpoints

BREAKING CHANGE: v1 endpoints are removed. Migrate to v2 before upgrading.
```

---

## 🔗 Common Footers

| Footer | Purpose | Example |
|--------|---------|---------|
| `BREAKING CHANGE:` | Document breaking API/behavior change | `BREAKING CHANGE: env var renamed` |
| `Fixes` / `Closes` | Link and auto-close issues | `Fixes #123` |
| `Refs` | Reference without closing | `Refs #456` |
| `Co-authored-by:` | Credit co-authors | `Co-authored-by: Name <email>` |
| `Reviewed-by:` | Credit reviewers | `Reviewed-by: Name <email>` |

---

## 🛠️ Quick Cheat Sheet

```bash
# Feature
git commit -m "feat(scope): add something new"

# Bug fix
git commit -m "fix(scope): resolve specific issue"

# Docs only
git commit -m "docs: update readme"

# Breaking change
git commit -m "feat(api)!: change response format"
```

---

## 💡 Tips

- **One logical change per commit** — easier to review and revert
- **Subject = what**, **body = why** (not how — the diff shows how)
- **Be specific:** `fix(auth): expire session after 24h` beats `fix: session bug`
- **Use scopes consistently** across the project
- Tools like **commitlint**, **semantic-release**, and **standard-version** rely on this format

---

## 📦 Semantic Versioning Link

Conventional Commits map to [SemVer](https://semver.org/):

| Commit Type | Version Bump |
|-------------|--------------|
| `fix` | PATCH (`1.0.0` → `1.0.1`) |
| `feat` | MINOR (`1.0.0` → `1.1.0`) |
| `BREAKING CHANGE` | MAJOR (`1.0.0` → `2.0.0`) |
