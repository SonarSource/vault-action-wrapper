# vault-action-wrapper

Ease the usage of hashicorp/vault-action within Sonar

## usage

This wrapper will select <https://vault.sonar.build:8200> automatically.

```yaml
- name: get secrets
  id: secrets
  uses: SonarSource/vault-action-wrapper@ref...
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

For further information, see
[HashiCorp Vault GitHub Action](https://github.com/hashicorp/vault-action).
