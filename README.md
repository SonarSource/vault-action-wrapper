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
      development/artifactory/token/{REPO_OWNER_NAME_DASH}-test access_token | jf_access_token;
- run: login-command ${{ fromJSON(steps.secrets.outputs.vault).jf_access_token }}
```

The `secrets` parameter will be pre-processed before passing it to the
`vault-action`. The following placeholders will be replaced:

* `{GITHUB_REPOSITORY}` => `octocat/Hello-World`
* `{GITHUB_REPOSITORY_OWNER}` => `octocat`
* `{REPO_NAME}` => `Hello-World`
* `{REPO_OWNER_NAME_DASH}` => `octocat-Hello-World`

The secrets can be accessed via `fromJSON(steps.secrets.outputs.vault).name`,
where `name` is the variable at the end of every line of the secrets
(`jf_access_token` in the above example).

### Role Selection

The action automatically selects the Vault JWT role based on `GITHUB_REF`:

* **Protected refs** (`refs/heads/main`, `refs/heads/master`, `refs/heads/branch-*`, `refs/tags/*`) use: `github-{org}-{repo}-protected`
* **Other refs** (feature branches, pull request refs such as `refs/pull/*/merge`) use: `github-{org}-{repo}`

Note that pull requests always use the non-protected role, even when targeting protected branches like `main` or
`master`, because their ref (`refs/pull/*/merge`) does not match any protected ref pattern.

This enables branch-based secret protection where sensitive secrets are only accessible from protected branches.

To override automatic role selection, use the `role` input:

```yaml
- uses: SonarSource/vault-action-wrapper@v3
  with:
    role: my-custom-role
    secrets: |
      development/kv/data/example token | example_token;
```

### Optional Parameters

#### `jwtTtl`

Controls the lifetime of the GitHub OIDC JWT token used for Vault
authentication. This directly affects how long the Vault authentication token
(and any child secret leases) remain valid.

* **Type**: `string`
* **Default**: Not set (uses hashicorp/vault-action default of 3600 seconds =
  1 hour)
* **Example values**: `"10800"` (3 hours), `"7200"` (2 hours)

**When to use**: Set this parameter when your workflow needs to run for longer
than 1 hour and requires access to Vault secrets throughout its execution.

```yaml
- name: get secrets for long-running workflow
  id: secrets
  uses: SonarSource/vault-action-wrapper@v3
  with:
    jwtTtl: "10800"  # 3 hours
    secrets: |
      development/artifactory/token/{REPO_OWNER_NAME_DASH}-private-reader access_token | artifactory_token;
```

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
          SONAR_TOKEN: ${{ fromJSON(steps.secrets.outputs.vault).sonarcloud_token }}
```

### Real-world examples

* [View usage within SonarSource GitHub Organization](https://github.com/search?q=org%3ASonarSource+vault-action-wrapper+path%3A.github%2Fworkflows%2F&type=code)

## FAQ

### Error: You must provide a valid path and key. Input "some/path/to/secret | ..."

This error can be raised for multiple reasons:

* the requested secret is wrongly written or does not exist
* the repository is not granted access to this secret by the engineering
  experience squad

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
