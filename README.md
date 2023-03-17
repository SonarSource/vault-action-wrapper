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


> Q1: **What if the requested secret is wrongly written or does not exist ?**
> 
> The step will fail with an error similar to this:
> ```
> Error: You must provide a valid path and key. Input: "some/path/to/secret | ..."
> ```

> Q2: **What happens when the current repository try to access a secret but has not Vault policy allowing that ?**
>
> Imagine you have a repository and want to use jira API in your workflow.
> `/some/path/jira | jira-api-token`
> 
> If the repository was not granted access to it by the RE-team then the following error would pop:
> ```
> Error: You must provide a valid path and key. Input: "some/path/to/secret | ..."
> ```
> Same as it question Q1. This is due to security reason, the Vault will not tell it knows something about this 
> secret if you are not granted to reach it.

> Q3: **What happens if the Vault instance is unreachable ?**
>
> Timeout error


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

## examples

### sonarcloud scan

```yaml
jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    permissions:
      id-token: write     # OIDC auth for vault
      contents: read      # checkout
      pull-requests: read # sonarcloud scan
    steps:
      - uses: actions/checkout@<lookup latest version>
        with:
          fetch-depth: 0
      - id: secrets
        uses: SonarSource/vault-action-wrapper@<lookup latest version>
        with:
          secrets: |
            development/kv/data/sonarcloud token | sonarcloud_token;
      - uses: SonarSource/sonarcloud-github-action@<lookup latest version>
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # provided by the GitHub runner
          SONAR_TOKEN: ${{ fromJSON(steps.secrets.outputs.vault).sonarcloud_token }}
```

### real-world examples
* https://github.com/search?q=org%3ASonarSource+vault-action-wrapper+path%3A%2F.github%2Fworkflows%2F&type=code
