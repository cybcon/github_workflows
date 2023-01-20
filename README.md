# github_workflows

This repo contains reusable GitHub Action workflows and configuration.

## pre-commit

The [pre-commit](./github/workflows/pre-commit.yaml) workflow creates matrix jobs for every pre-commit hook.
The matrix jobs fail if the hook fails and/or if there is a diff after running the hook.

This action requires a `.pre-commit-config.yaml` file that contains the hooks as well as a `.tools-versions` file that contains (at least) the version of pre-commit to be installed.

The pre-commit workflow can be integrated using the following workflow file:

```yaml
name: pre-commit
on:
  push:
jobs:
  pre-commit:
    uses: cybcon/github_workflows/.github/workflows/pre-commit.yaml@v2.0.0
```

## release

The release workflows provide a pull-request label based release process.
A release is created when a pull request is merged to the `main` or `master` branch.
The release version type (major, minor or patch) is determined by a corresponding release label.

The [release-label-validation](./github/workflows/release-label-validation.yaml) workflow ensures that a pull request cannot be merged before an appropriate label is set.
The [release-from-label](./github/workflows/release-from-label.yaml) workflow is responsible for creating the actual release after merge.

Possible release labels are
- `major` - Release a major version
- `minor` - Release a minor version
- `patch` - Release a patch version

Additionally the `pre-release` label can be set to create a pre-release instead of a production one.
If no release should be created the `chore` label must be used.

The release workflows can be integrated using the following workflow files:

```yaml
name: release
on:
  pull_request:
    types:
      - closed
jobs:
  release:
    uses: cybcon/github_workflows/.github/workflows/release-from-label.yaml@v2.0.0
```

```yaml
name: release-label-validation
on:
  pull_request:
    types:
      - opened
      - edited
      - synchronize
      - reopened
      - labeled
      - unlabeled
jobs:
  release-label-validation:
    uses: cybcon/github_workflows/.github/workflows/release-label-validation.yaml@v2.0.0
```
