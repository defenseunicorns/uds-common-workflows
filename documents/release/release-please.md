# Release Please

A reuseable workflow used to aid the release process and documenting the changes with less effort from the maintainer and/or team.

## Init

Release Please requires a repository to be initialized first. Todo so, must one have NPM installed and one must run: `npm -i release-please`. The documentation wants the `-g` switch at the end for a global install (required to use from a common system path).

Once the installation is completed, initialize repository with the following:
```bash
release-please bootstrap \
  --token=$GITHUB_TOKEN \
  --repo-url=<owner>/<repo> \
  --release-type=simple
```

The GitHub token being your personal access token. You will know it was successful when GitHub has a PR ready for you with 2 files added. They are:
* `.release-please-manifeset.json`
* `release-please-config.json`

The `release-please-config.json` can be edited manually with one of the [resources](#resources) listed. An example of a simple config for Release-Please is:
```json
{
  "packages": {
    ".": {
      "changelog-path": "CHANGELOG.md",
      "changelog-type": "github",
      "release-type": "simple",
      "bump-minor-pre-major": false,
      "bump-patch-for-minor-pre-major": false,
      "draft": false,
      "prerelease": false
    }
  },
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json"
}

```

## Reuseable Workflows

A reuseable workflow has been set in this repository for use for Terraform. This workflow is designed right now to be imported and run from runners associated with host project calling the runners. Because the workflows do live outide of the local repository, some configurations must be made to the repository settings allowing the use of a remote GitHub Workflow. The goal of a reuseable pipeline is to leverage the work and knowledge of others and reduce the cognitive load for development tasks.

### Repository Permissions

Before including a workflow from another respository, first set go to the repository in GitHub (with admin or owner permissions). Click on **Settings** --> **Actions** --> **General**.

In this screen, ensure the following sections have at least 1 of the mentioned settings.

**Actions permissions**:
* *Allow all actions and reuseable workflows*
* *Allow DefenseUnicorns actions and reuseable workflows*

**Workflow permissions**
* *Read and write permissions*
* [x] *Allow GitHub Actions to create and approve pull requests*

### Common Release Workflow

Finally, assuming the [init](#init) has been set for the repository and the [repository permissions](#Repository-Permissions) have both been set. Simply perform the following:

1. Create a worklows folder if it does not exist already `mkdir -p .github/workflows`.
1. Copy the following contents into `.github/workflows/please-release.yml`.

```yaml
name: Release Please Merge
on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write
  packages: write
  repository-projects: read
 

jobs:
  release-terraform:
    uses: defenseunicorns/uds-common-workflows/.github/workflows/release-terraform.yml@main
    with:
      command: manifest
      release-type: simple
```



## Resources
* [Release Please GitHub](https://github.com/googleapis/release-please)
* [Init Release Please Docs](https://github.com/googleapis/release-please/blob/main/docs/cli.md)
* [Youtube: Reuseable Workflows](https://youtu.be/bCqPXUcBfJQ)
* [GitHub Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
