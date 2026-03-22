# sdk-docker-build-action

GitHub Action used by the SDK team to build and publish Docker images for FIT/SIT tests.

Builds a Docker image and publishes it to `ghcr.io`.

## Build FIT performer images

Here's an example workflow using this action:

```yaml
name: Build FIT performer

run-name: ${{ inputs.ref }}

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
      - uses: couchbaselabs/sdk-docker-build-action@v1
        with:
          sdk: <my-sdk-name> # like 'rust' or 'analytics-python'
          ref: ${{ inputs.ref }}
          dockerfile: path/to/Dockerfile # defaults to `Dockerfile`
        env:
          DOCKER_BUILD_SUMMARY: false # if you don't want the summary
```

## Prune stale images

Images built from PRs and specific commit hashes have names starting with `refs-` and `sha-`.
These should be pruned after some time.

Every time an image is built for a branch, the branch's previous image becomes untagged.
These untagged images should be pruned after some time.

Here's an example workflow:

```yaml
name: Prune stale images

on:
  schedule:
      # Change this to a random time of day to avoid stampede  
      - cron: '43 3 * * *'
  workflow_dispatch:

jobs:
  prune:
    runs-on: ubuntu-latest
    permissions:
      packages: write

    env:
      # Change this to match your SDK name
      SDK: rust

    steps:
      - name: Prune old temporary images
        uses: vlaurin/action-ghcr-prune@0cf7d39f88546edd31965acba78cdcb0be14d641 # v0.6.0
        with:
          token: ${{ github.token }}
          organization: ${{ github.repository_owner }}
          container: ${{ env.SDK }}-fit-performer
          keep-younger-than: 7 # days
          prune-untagged: true
          prune-tags-regexes: |
            ^refs-
            ^sha-
```