# publish-fit-performer

Builds a FIT performer Docker image and publishes it to `ghcr.io`.

Example usage:

```yaml
name: Publish FIT performer

run-name: ${{ github.event.head_commit.message || inputs.ref }}

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}-${{ inputs.ref }}
  cancel-in-progress: true

on:
  push:
    branches:
      - 'main'
      - '[0-9]+.[0-9]+.x' # release branches
    tags:
      - '*'
    paths-ignore:
      - path/to/unrelated/files

  workflow_dispatch:
    inputs:
      ref:
        type: string
        description: "Performer version. Specify a branch, tag, full commit hash, GitHub PR like 'refs/pull/37/merge', or Gerrit PR like 'refs/changes/58/240858/6'"
        required: true

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: dnault/publish-fit-performer@v1
        with:
          sdk: <my-sdk-name> # like 'rust' or 'analytics-python'
          ref: ${{ inputs.ref }}
          dockerfile: path/to/Dockerfile # defaults to `Dockerfile`
        env:
          DOCKER_BUILD_SUMMARY: false # if you don't want the summary
```
