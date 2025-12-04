HRMS reusable workflows for HRMS

This repository contains reusable GitHub Actions workflows that can be called from other repositories in the organization to build, scan, generate SBOMs, and push container images to ECR.

Usage

- To call the reusable workflow from another repository use the `uses` syntax pointing to this repository and the workflow path. Example:

```yaml
jobs:
  build-and-push:
    uses: PookieLand/workflows_hrms/.github/workflows/reusable-build-push.yml@main
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

Composite action: `build-push`

This repository also exposes a composite action at `.github/actions/build-push` you can call from inside a job when you need step-level reuse instead of a whole reusable workflow. Use it when you want the build and push steps to run inside the caller job and share the caller's runner environment.

Example (call action inside a job):

```yaml
jobs:
  test-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Run build-push steps
        uses: PookieLand/workflows_hrms/.github/actions/build-push@main
        with:
          servicePath: '.'
          imageName: 'hrms/example-service'
          registry: '475936984863.dkr.ecr.ap-south-1.amazonaws.com'
          region: 'ap-south-1'
          push_image: false
```

Version files and release automation

You can now control releases by bumping version files in the repository root:

- `ACTION_VERSION` — bump this to release a new composite action version (creates tag `action-v<version>` and a GitHub release).
- `WORKFLOW_VERSION` — bump this to release a new reusable workflow version (creates tag `workflow-v<version>` and a GitHub release).

When you update either file and push to `main`, the repository workflow `release-on-version-change.yml` will create the corresponding tag and a GitHub release. Callers should reference the tagged ref in their `uses:` lines for reproducible builds, for example:

```yaml
uses: PookieLand/workflows_hrms/.github/actions/build-push@action-v1.0.0
```

Or for the workflow:

```yaml
uses: PookieLand/workflows_hrms/.github/workflows/reusable-build-push.yml@workflow-v1.0.0
```

Contact

If you want changes to the workflow (inputs, scans, policies), update the workflow or action in this repo and then bump the reference in callers (prefer tag/sha for reproducibility).
