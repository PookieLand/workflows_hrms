HRMS reusable workflows for HRMS

This repository contains reusable GitHub Actions workflows that can be called from other repositories in the organization to build, scan, generate SBOMs, and push container images to ECR.

Usage

- To call the reusable workflow from another repository use the `uses` syntax pointing to this repository and the workflow path. Example:

```yaml
jobs:
  build-and-push:
    uses: PookieLand/workflows_hrms/.github/workflows/reusable-build-push.yml@v1.0.0
    with:
      servicePath: '.'                # relative to the calling repository root
      imageName: 'hrms/employee-service'
      registry: '475936984863.dkr.ecr.ap-south-1.amazonaws.com'
      region: 'ap-south-1'
      push_image: true
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      GITOPS_REPO_TOKEN: ${{ secrets.GITOPS_REPO_TOKEN }}
```

Inputs

- `servicePath` (string, required): Path inside the caller repo where the Dockerfile/project lives.
- `imageName` (string, required): Image name used locally (e.g. `hrms/employee-service`).
- `registry` (string, required): ECR registry host (e.g. `123456789012.dkr.ecr.region.amazonaws.com`).
- `region` (string, required): AWS region for ECR.
- `gitops_repo` (string, optional): Repo to update with new image tag (e.g. `org/gitops-manifests`).
- `push_image` (boolean, optional): Whether to tag & push image to ECR and update gitops manifests.

Secrets

- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (required when pushing)
- `GITOPS_REPO_TOKEN` (required if `gitops_repo` provided and `push_image` is true)

# Notes and migration advice

- After splitting `HRMS` into multiple repositories, keep this repository as the single source of truth for the build-and-push workflow.
- Each service repo should include a lightweight workflow which calls this workflow using `uses:` above.
- If you prefer to keep the central matrix detection (build multiple services from one repo), create a separate repository that contains a matrix workflow which calls this reusable workflow multiple times via `uses:` and a `strategy.matrix`.
-

Versioned releases

This repository uses a version-file driven release model for the reusable workflow. Bump the `WORKFLOW_VERSION` file in the repository root to publish a new tagged release for the reusable workflow; the workflow `.github/workflows/release-on-version-change.yml` will create the tag `v<version>` and a GitHub release for that tag.

Callers should reference the generated tag in their `uses:` lines for reproducible builds, for example:

```yaml
uses: PookieLand/workflows_hrms/.github/workflows/reusable-build-push.yml@v1.0.0
```

Latest release

- **`v1.0.0`** — created 04 Dec 2025. Use this tag in callers for a stable, reproducible workflow reference.

Contact

If you want changes to the workflow (inputs, scans, policies), update the workflow or action in this repo and then bump the reference in callers (prefer tag/sha for reproducibility).

Examples

Example workflows and templates are kept in the `examples/` folder to avoid accidental execution by GitHub. Copy the template you want into your repository's `.github/workflows/` directory (or reference the released tag directly with `uses:`) and update secrets/inputs as required.

Path: `examples/example-usage.yml` — a template showing how to call the reusable workflow and where to place secrets.
