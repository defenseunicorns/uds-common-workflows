# UDS Common GitHub Workflows

Repository containers common GitHub workflows and actions for UDS
  
## Creating releases

The pipeline uses [release-please-action](https://github.com/google-github-actions/release-please-action) for versioning and releasing OCI packages. This will automatically update `metadata.version` field in zarf.yaml to the same version number that is used for the release tag. To make this work, the `version` field must be surrounded by Release Please's annotations,

```yaml
  # x-release-please-start-version
  version: "0.2.1"
  # x-release-please-end
```

More information on this can be found in [Release Please's documentation](https://github.com/googleapis/release-please/blob/main/docs/customizing.md#updating-arbitrary-files).

### How should I write my commits?

Release Please assumes you are using [Conventional Commit messages](https://www.conventionalcommits.org/).

The most important prefixes you should have in mind are:

- `fix:` which represents bug fixes, and correlates to a [SemVer](https://semver.org/)
  patch.
- `feat:` which represents a new feature, and correlates to a SemVer minor.
- `feat!:`,  or `fix!:`, `refactor!:`, etc., which represent a breaking change
  (indicated by the `!`) and will result in a SemVer major.

When the change is merged to the trunk, Release Please will calculate what changes are included and will create another PR to increase the version and tag a new release. This will also automatically generate a new set of packages in the OCI registry.

### Modifying the Monorepo

Use the [release-please-config.json](release-please-config.json) file to add or remove packages for publishing.
