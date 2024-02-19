Tag Docker Image GitHub Action
==============================

This GitHub Action tags a Docker image in a Docker registry with a specified tag, offering options to work across different AWS accounts and regions. It's perfect for workflows that involve image tagging as part of CI/CD processes, especially when dealing with multi-region deployments or multiple AWS accounts.

Features
--------

*   **Tagging Docker Images:** Easily tag an existing Docker image in a registry with a new tag.
*   **AWS Account Flexibility:** Supports tagging across different AWS accounts by assuming roles.
*   **Multi-Region Support:** Works with Docker images stored in any AWS region.
*   **Destination Registry Option:** Allows tagging in a different destination registry.

Inputs
------

| Input | Description | Required |
| --- | --- | --- |
| `registry` | The Docker registry to tag the image in. | Yes |
| `dst-registry` | The Docker registry to copy the image to from the `registry` input. | No |
| `aws-account` | The AWS account to assume role for. | Yes |
| `dst-aws-account` | The Destination AWS account to use if different from source account. | No |
| `region` | The AWS region. Defaults to `us-east-1`. | No |
| `tag` | The tag to use for the image. | No |
| `sha` | The git SHA to tag the image with. | Yes |

Usage
-----

To use this action in your workflow, follow these steps:

```yaml
steps:
  - name: Tag Docker Image   
    uses: bisnow/tag-docker-image-action@v1
    with:
      registry: "<your-registry>"
      aws-account: "<your-aws-account-id>"
      sha: "<your-git-sha>"
      # Optional inputs below
      dst-registry: "<your-destination-registry>"
      dst-aws-account: "<your-destination-aws-account>"
      region: "<aws-region>"
      tag: "<your-tag>"
```

Replace `<your-registry>`, `<your-aws-account-id>`, `<your-git-sha>`, `<your-destination-registry>`, `<your-destination-aws-account>`, `<aws-region>`, and `<your-tag>` with your specific details.

Example Workflow
----------------

Here's an example of how to incorporate this action into a GitHub Actions workflow:

```yaml
name: Example Workflow for Docker Tagging
on:
  push:
    branches:
      - main
jobs:
  tag-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Tag Docker Image
        uses: bisnow/tag-docker-image-action@v1
        with:
          registry: "your-registry"
          aws-account: "your-aws-account-id"
          sha: "${{ github.sha }}"
          dst-registry: "your-destination-registry"
          dst-aws-account: "your-destination-aws-account"
          tag: "latest"
```