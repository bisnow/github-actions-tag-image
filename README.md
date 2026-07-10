Tag Docker Image GitHub Action
==============================

This GitHub Action tags an existing Docker image in a registry with a new tag. It supports both **Amazon ECR** (assuming a role in one or more AWS accounts) and **Harbor** (`harbor.bisnow.cloud`). It's designed for CI/CD workflows that move an environment/release tag (e.g. `non-prod`, `prod`) onto an already-built image, including multi-region / multi-account setups.

Features
--------

*   **Tagging Docker Images:** Retag an existing image (`registry:sha`) with a new tag using `docker buildx imagetools` (no pull-and-repush of layers).
*   **ECR support:** Assumes the CI/CD role and logs into Amazon ECR.
*   **Harbor support:** With `only-harbor: 'true'`, logs into `harbor.bisnow.cloud` with the runner's robot credential instead of ECR.
*   **AWS Account Flexibility:** Supports tagging across different AWS accounts by assuming roles (ECR path).
*   **Destination Registry Option:** Allows copying/tagging into a different destination registry (ECR path).

> **Harbor support (`only-harbor`) requires v2 or later.** `v1` is ECR-only.

Inputs
------

| Input | Description | Required |
| --- | --- | --- |
| `registry` | The Docker registry/repository to tag the image in (e.g. `harbor.bisnow.cloud/bisnow/my-app` or an ECR URL). | Yes |
| `sha` | The existing image tag/git SHA to retag from (`registry:sha`). | Yes |
| `only-harbor` | Set to `'true'` to tag in Harbor: skips AWS/ECR auth and logs into `harbor.bisnow.cloud` with the runner robot credential. Defaults to `'false'` (ECR path). | No |
| `tag` | The new tag to apply to the image. | No |
| `dst-registry` | The Docker registry to copy the image to from the `registry` input (ECR path). | No |
| `aws-account` | The AWS account to assume a role for. Required for the ECR path; ignored when `only-harbor` is `'true'`. | No |
| `dst-aws-account` | The destination AWS account, if different from the source account (ECR path). | No |
| `region` | The AWS region. Defaults to `us-east-1`. | No |

Usage
-----

### Harbor (`only-harbor`)

```yaml
steps:
  - name: Tag image with environment
    uses: bisnow/github-actions-tag-image@v2
    with:
      registry: harbor.bisnow.cloud/bisnow/my-app
      only-harbor: 'true'
      sha: ${{ inputs.tag }}          # existing image, e.g. rc-42
      tag: ${{ inputs.environment }}  # moving tag, e.g. non-prod / prod
```

The runner must already have the `harbor.bisnow.cloud` robot credential in
`~/.docker/config.json` (provisioned on the `arc-runners-bisnow` pools). The
action performs an explicit `docker login` so `buildx imagetools` can push the
tag to Harbor.

### ECR (default)

```yaml
steps:
  - name: Tag Docker Image
    uses: bisnow/github-actions-tag-image@v2
    with:
      registry: "<your-ecr-registry>"
      aws-account: "<your-aws-account-id>"
      sha: "${{ github.sha }}"
      tag: "latest"
      # Optional (cross-account / destination registry)
      dst-registry: "<your-destination-registry>"
      dst-aws-account: "<your-destination-aws-account>"
      region: "us-east-1"
```

Example Workflow
----------------

```yaml
name: Example Workflow for Docker Tagging
on:
  push:
    branches:
      - main
permissions:
  id-token: write
  contents: read
jobs:
  tag-and-push:
    runs-on: arc-runners-bisnow
    steps:
      - name: Tag Docker Image (ECR)
        uses: bisnow/github-actions-tag-image@v2
        with:
          registry: "your-ecr-registry"
          aws-account: "your-aws-account-id"
          sha: "${{ github.sha }}"
          tag: "latest"
```
