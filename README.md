# github_workflows

This repo contains reusable GitHub Action workflows and configuration.

## pre-commit

The [pre-commit](./.github/workflows/pre-commit.yaml) workflow creates matrix jobs for every pre-commit hook.
The matrix jobs fail if the hook fails and/or if there is a diff after running the hook.

This action requires a `.pre-commit-config.yaml` file that contains the hooks as well as a `.tool-versions` file that contains (at least) the version of pre-commit to be installed.

The pre-commit workflow can be integrated using the following workflow file:

```yaml
name: pre-commit
on:
  push:
jobs:
  pre-commit:
    uses: cybcon/github_workflows/pre-commit.yaml@v2.1.0
```

## release

The release workflows provide a pull-request label based release process.
A release is created when a pull request is merged to the `main` or `master` branch.
The release version type (major, minor or patch) is determined by a corresponding release label.

The [release-label-validation](./.github/workflows/release-label-validation.yaml) workflow ensures that a pull request cannot be merged before an appropriate label is set.
The [release-from-label](./.github/workflows/release-from-label.yaml) workflow is responsible for creating the actual release after merge.

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
    uses: cybcon/github_workflows/.github/workflows/release-from-label.yaml@v2.1.0
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
    uses: cybcon/github_workflows/.github/workflows/release-label-validation.yaml@v2.1.0
```

## Container image vulnerability scan

The [container-vulnerability-scan](./.github/workflows/container-vulnerability-scan.yaml) workflows scans a given container image and scans it with the latest version of [Aquasec Trivy](https://www.aquasec.com/products/trivy/).
The scan report will be send to `GITHUB_STEP_SUMMARY`.

The workflow will take following inputs:

| Name                      | Type      | Description                                                                             | Required | Default value |
|---------------------------|-----------|-----------------------------------------------------------------------------------------|----------|---------------|
| `image_name`              | `string`  | Container image name and tag to scan.                                                   | **yes**  |               |
| `image_artifact_name`     | `string`  | Container image artifact name to identify the container image file from artifacts.      | no       |               |
| `image_artifact_filename` | `string`  | Container image file that needs to be downloaded from artifacts.                        | no       |               |
| `login_dockerhub`         | `boolean` | Login to DockerHub, requires the secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`. | no       | `false`       |
| `trivy_tag`               | `string`  | Use a special aquasec/trivy container image tag.                                        | no       | `latest`      |
| `DOCKERHUB_USERNAME`      | `string`  | The username to login to Docker Registry. (**SECRET**)                                  | no       |               |
| `DOCKERHUB_PASSWORD`      | `string`  | The password to login to Docker Registry. (**SECRET**)                                  | no       |               |

### Examples

Build a container image, upload it to the pipeline artifacts and trigger the vulnerability scan.

```yaml
jobs:
  container-build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout GIT repository
        uses: actions/checkout@v7
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v4
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4
      - name: Test build and export for further validation
        uses: docker/build-push-action@v7
        with:
          push: false
          load: true
          context: .
          tags: container-build:test
          outputs: type=docker,dest=/tmp/container.tar
      - name: Upload container image as artifact
        uses: actions/upload-artifact@v7
        with:
          name: container-build
          path: /tmp/container.tar
  scan:
    name: Container vulnerability scan
    needs: container-build
    uses: cybcon/github_workflows/.github/workflows/container-vulnerability-scan.yaml@v2.1.0
    with:
      image_name: container-build:test
      image_artifact_name: container-build
      image_artifact_filename: container.tar
      login_dockerhub: false
      trivy_tag: latest
```

## Container image build validation

The [container-image-build-validation](./.github/workflows/container-image-build-validation.yaml) workflow tests a docker build and afterwards it triggers the **Container image vulnerability scan** workflow (see above).

The workflow will take following inputs:

| Name                      | Type      | Description                                                                             | Required | Default value              |
|---------------------------|-----------|-----------------------------------------------------------------------------------------|----------|----------------------------|
| `context`                 | `string`  | The context/location where to find the file Dockerfile.                                 | no       | `.`                        |
| `platforms`               | `string`  | The supported plattforms, used for docker build.                                        | no       | `linux/amd64, linux/arm64` |
| `login_dockerhub`         | `boolean` | Login to DockerHub, requires the secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`. | no       | `false`                    |
| `trivy_tag`               | `string`  | Use a special aquasec/trivy container image tag.                                        | no       | `latest`                   |
| `DOCKERHUB_USERNAME`      | `string`  | The username to login to Docker Registry. (**SECRET**)                                  | no       |                            |
| `DOCKERHUB_PASSWORD`      | `string`  | The password to login to Docker Registry. (**SECRET**)                                  | no       |                            |

### Examples

Build a container image, upload it to the pipeline artifacts and trigger the vulnerability scan.

```yaml
jobs:
  container-image-build-validation:
    uses: cybcon/github_workflows/.github/workflows/container-image-build-validation.yaml@v2.1.0
```

## Donate

I would appreciate a small donation to support the further development of my open source projects.

[![Donate with PayPal][donate-paypal-button]][donate-paypal-link]

## License

Copyright (c) 2023-2026 Michael Oberdorf IT-Consulting

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

<!-- LINK GROUP -->
[donate-paypal-button]: https://raw.githubusercontent.com/cybcon/paypal-donate-button/refs/heads/master/paypal-donate-button_200x77.png
[donate-paypal-link]: https://www.paypal.com/donate/?hosted_button_id=BHGJGGUS6RH44
