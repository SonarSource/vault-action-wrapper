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

On a protected ref other than `main` and `master`, a third attempt uses `github-{org}-{repo}` when both attempts with the
protected role failed (for example a repository whose protected role only binds `master`). It only returns the secrets
that the base role can read: a pattern that includes a protected-only path still fails, with the diagnosis of the base
role. `main` and `master` never fall back.

For pull request workflows, `GITHUB_REF` is `refs/pull/*/merge`, which does not match any protected patterns, so PRs
always use the non-protected role.

`issue_comment` always uses the non-protected role: the workflow runs on the default branch, but the protected JWT role
does not bind that event.

Override with explicit `role` input for backward compatibility.

## Release

```bash
git fetch --tags
git update-ref -m "reset: update branch v3 to tag X.Y.Z" refs/heads/v3 X.Y.Z
git push origin v3
```
