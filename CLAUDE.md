# CLAUDE.md

## Project Overview

GitHub Composite Action wrapping `hashicorp/vault-action` for SonarSource. All logic lives in `action.yaml`.

## Development

```bash
pre-commit run --all-files  # Linting and YAML validation
```

No build/compile steps - pure YAML action.

## Testing

Integration tests in `.github/workflows/test.yaml` run against staging Vault on PRs.

## Role Selection

Auto-selects Vault JWT role based on `GITHUB_REF`:

- Protected refs (`main`, `master`, `branch-*`, `tags/*`) → `github-{org}-{repo}-protected`
- Other refs → `github-{org}-{repo}`

For pull request workflows, `GITHUB_REF` is `refs/pull/*/merge`, which does not match any protected patterns, so PRs
always use the non-protected role.

Override with explicit `role` input for backward compatibility.

## Release

```bash
git fetch --tags
git update-ref -m "reset: update branch v3 to tag X.Y.Z" refs/heads/v3 X.Y.Z
git push origin v3
```
