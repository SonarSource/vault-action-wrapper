# vault-action-wrapper

Ease the usage of hashicorp/vault-action within Sonar

## Usage

This wrapper will select <https://vault.sonar.build> automatically.

```yaml
- name: get secrets
  id: secrets
  uses: SonarSource/vault-action-wrapper@v3
  with:
    secrets: |
      development/artifactory/token/${{ github.repository_owner }}-${{ github.event.repository.name }}-test
        access_token |
        jf_access_token;
- name: use secrets
  env:
    JF_ACCESS_TOKEN: ${{ steps.secrets.outputs.jf_access_token }}
  run: echo "scripts can use $JF_ACCESS_TOKEN"
```

### Secret Path Patterns

**Recommended:** Use GitHub context variables directly in your secret paths:

- `${{ github.repository_owner }}` - Repository owner (e.g., `SonarSource`)
- `${{ github.event.repository.name }}` - Repository name (e.g., `vault-action-wrapper`)

Example recommended pattern:

```text
development/artifactory/token/${{ github.repository_owner }}-${{ github.event.repository.name }}-test
```

**Legacy:** The wrapper also supports placeholder replacement for backward compatibility.
The `secrets` parameter will be pre-processed before passing it to the `vault-action`.
The following placeholders will be replaced:

- `{GITHUB_REPOSITORY}` => `SonarSource/vault-action-wrapper`
- `{GITHUB_REPOSITORY_OWNER}` => `SonarSource`
- `{REPO_NAME}` => `vault-action-wrapper`
- `{REPO_OWNER_NAME_DASH}` => `SonarSource-vault-action-wrapper`

### Accessing Secrets

**New (Recommended):** Secrets are now available as direct outputs:

```yaml
env:
  JF_ACCESS_TOKEN: ${{ steps.secrets.outputs.jf_access_token }}
```

**Legacy:** Secrets can also be accessed via the vault output object:

```yaml
env:
  JF_ACCESS_TOKEN: ${{ fromJSON(steps.secrets.outputs.vault).jf_access_token }}
```

The variable name corresponds to the identifier at the end of each secrets line (`jf_access_token` in the above example).

### Permissions

The action is using OIDC to authenticate.
This requires write permissions for `id-token` to fetch a JWT.

```yaml
jobs:
  foo:
    permissions:
      id-token: write
      ...
```

For further information, see
[HashiCorp Vault GitHub Action](https://github.com/hashicorp/vault-action).

## Examples

### SonarCloud Scan

```yaml
jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    permissions:
      id-token: write     # required by SonarSource/vault-action-wrapper
      contents: read      # required by actions/checkout
      pull-requests: read # required by SonarSource/sonarcloud-github-action
    steps:
      - uses: actions/checkout@692973e3d937129bcbf40652eb9f2f61becf3332 # v4.1.7
        with:
          # Disabling shallow clone is recommended for improving relevancy of reporting
          fetch-depth: 0
      - id: secrets
        uses: SonarSource/vault-action-wrapper@v3
        with:
          secrets: |
            development/kv/data/sonarcloud token | sonarcloud_token;
      - uses: SonarSource/sonarcloud-github-action@ffc3010689be73b8e5ae0c57ce35968afd7909e8 # v5.0.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ steps.secrets.outputs.sonarcloud_token }}
```

### Real-world examples

- [View usage within SonarSource GitHub Organization](https://github.com/search?q=org%3ASonarSource+vault-action-wrapper+path%3A.github%2Fworkflows%2F&type=code)

## FAQ

### Error: You must provide a valid path and key. Input "some/path/to/secret | ..."

This error can be raised for multiple reasons:

- the requested secret is wrongly written or does not exist
- the repository is not granted access to this secret

  Due to security reason, the Vault will not tell it knows something about a
  secret if the user is not granted to reach it.

### Timeout error

Such error could be raised in case the Vault instance is unreachable.

### Error: Unable to get ACTIONS_ID_TOKEN_REQUEST_URL env variable

`id-token: write` permission is missing.

## Release

Create a release from a maintained branches, then update the v* shortcut:

```bash
git fetch --tags
git update-ref -m "reset: update branch v3 to tag 3.0.0" refs/heads/v3 3.0.0
git push origin v3
```
