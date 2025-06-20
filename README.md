# gh-action_sbom

Action and reusable workflow for Docker SBOM generation from GitHub workflows.

The generation of Software Bill of Material (SBOM) is implemented with [Syft](https://github.com/anchore/syft)
and [sbom-action](https://github.com/anchore/sbom-action).

The SBOM files are signed and eventually attached to the workflow and to the release.

## Usage

The BOM file is signed if the `upload-artifact` parameter is true and the GPG secrets are provided.

Repositories needs to have access to the following secrets:
  development/kv/data/sign passphrase
  development/kv/data/sign key

### GitHub Action

```yaml
jobs:
  job-calling-action:
    steps:
      - name: get secrets
        id: secrets
        uses: SonarSource/vault-action-wrapper@3996073b47b49ac5c58c750d27ab4edf469401c8 # 3.0.1
        with:
          secrets: |
            development/kv/data/sign passphrase | gpg_passphrase;
            development/kv/data/sign key | gpg_key;
      - uses: SonarSource/gh-action_sbom@v1
        with:
          image: example/image_name:tag
          filename: bom.json
          upload-artifact: true
          upload-release-assets: true
          registry-username: "username"
          registry-password: "password"
        env:
          GPG_PRIVATE_KEY_PASSPHRASE: ${{ fromJSON(steps.secrets.outputs.vault).gpg_passphrase }}
          GPG_PRIVATE_KEY_BASE64: ${{ fromJSON(steps.secrets.outputs.vault).gpg_key }}
```

### GitHub Reusable Workflow

:warning: The strategy property is not supported in any job that calls a reusable workflow.
See [reusing workflows limitations](https://docs.github.com/en/actions/using-workflows/reusing-workflows#limitations)

```yaml
jobs:
  job-calling-workflow:
    uses: SonarSource/gh-action_sbom/.github/workflows/workflow.yml@v1
    with:
      image: example/image_name:tag
      filename: bom.json
      upload-artifact: true
      upload-release-assets: true
```

## Versioning

Using the versioned semantic [tags](#tags) is recommended for security and reliability.

See [GitHub: Using tags for release management](https://docs.github.com/en/actions/creating-actions/about-custom-actions#using-tags-for-release-management)
and [GitHub: Keeping your actions up to date with Dependabot](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/keeping-your-actions-up-to-date-with-dependabot)
.

For convenience, it is possible to use the [branches](#branches) following the major releases.

### Tags

This repository is released following semantic versioning,
ie: [`1.0.0`](https://github.com/SonarSource/gh-action_sbom/releases/tag/1.0.0).

```yaml
jobs:
  job-calling-workflow:
    uses: SonarSource/gh-action_sbom/.github/workflows/workflow.yml@1.0.0

  job-calling-action:
    steps:
      - uses: SonarSource/gh-action_sbom@1.0.0
```

### Branches

The `master` branch shall not be referenced by end-users.

Branches prefixed with a `v` are pointers to the last major versions, ie: [`v1`](https://github.com/SonarSource/gh-action_sbom/tree/v1).

```yaml
jobs:
  job-calling-workflow:
    uses: SonarSource/gh-action_sbom/.github/workflows/workflow.yml@v1

  job-calling-action:
    steps:
      - uses: SonarSource/gh-action_sbom@v1
```

Note: use only branches with precaution and confidence in the provider.

## Development

The development is done on `master` and the `branch-*` maintenance branches.

### Release

Create a release from a maintained branches, then update the `v*` shortcut:

```shell
git fetch --tags
git update-ref -m "reset: update branch v1 to tag 1.0.0" refs/heads/v1 1.0.0
git push origin v1
```

## FAQ

### Warning Unexpected input

> Warning: Unexpected input(s) 'upload-artifact', 'upload-release-assets',
> valid inputs are ['path', 'image', 'registry-username', 'registry-password', 'format', 'github-token',
> 'artifact-name', 'output-file', 'syft-version', 'dependency-snapshot']

The warning can be ignored, see anchore/sbom-action#269

## References

[Xtranet/RE/Artifact Management#GitHub Actions](https://xtranet-sonarsource.atlassian.net/wiki/spaces/RE/pages/872153170/Artifact+Management#GitHub-Actions)

[Semantic Versioning 2.0.0](https://semver.org/)

[GitHub: About Custom Actions](https://docs.github.com/en/actions/creating-actions/about-custom-actions)

[Syft](https://github.com/anchore/syft)

[Syft GitHub Action for SBOM Generation](https://github.com/anchore/sbom-action)
