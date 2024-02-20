# vault-action-wrapper

Ease the usage of hashicorp/vault-action within Sonar

## Usage

This wrapper will select <https://vault.sonar.build:8200> automatically.

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

### Permissions
The action is using OIDC to authenticate. This requires write permissions for `id-token` to fetch a JWT.

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
      - uses: actions/checkout@v4
        with:
          # Disabling shallow clone is recommended for improving relevancy of reporting
          fetch-depth: 0
      - id: secrets
        uses: SonarSource/vault-action-wrapper@v3
        with:
          secrets: |
            development/kv/data/sonarcloud token | sonarcloud_token;
      - uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ fromJSON(steps.secrets.outputs.vault).sonarcloud_token }}
```

### Real-world examples
* https://github.com/search?q=org%3ASonarSource+vault-action-wrapper+path%3A.github%2Fworkflows%2F&type=code

## FAQ

### Error: You must provide a valid path and key. Input "some/path/to/secret | ..."
This error can be raised for multiple reasons:
* the requested secret is wrongly written or does not exist
* the repository is not granted access to this secret by the RE-team

  Due to security reason, the Vault will not tell it knows something about a
  secret if the user is not granted to reach it.

### Timeout error
Such error could be raised in case the Vault instance is unreachable.

### Error: Unable to get ACTIONS_ID_TOKEN_REQUEST_URL env variable
`id-token: write` permission is missing.

## Releasing

- Update the v3 branch to the newly created tag.
