# CI/CD pipeline

## Workflow triggers

| Workflow | Automatic trigger | Manual trigger |
| --- | --- | --- |
| `.github/workflows/main.yml` (`Main`) | Push to `main` | `workflow_dispatch` |
| `.github/workflows/branch.yml` (`Branches`) | Push to any branch except `main` | `workflow_dispatch` |

There is no `pull_request` trigger. A push to a feature branch starts its
workflow without needing a PR. Manual dispatch is also supported; select the
intended workflow and ref carefully.

Both workflows run on `ubuntu-latest` and grant jobs `contents: read` and
`id-token: write`. The latter allows OIDC-based AWS role assumption rather
than committing long-lived AWS credentials.

## Build job

The build job executes these steps in order:

1. `ServerlessOpsIO/gha-setup-workspace@v2` sets up the checked-out workspace.
2. `ServerlessOpsIO/gha-assume-aws-credentials@v1` obtains credentials using
   `AWS_CICD_ACCOUNT_ID`.
3. `aws-actions/setup-sam@v3` installs SAM.
4. `sam validate --lint` validates infrastructure with CloudFormation linting.
5. `sam build` builds the deployable application.
6. `ServerlessOpsIO/gha-store-artifacts@v2`, with `use_aws_sam: true`, stores
   the build output for the deploy job.

`sam build` fails while `template.yaml` has no resources, so the build job
fails on the starter as generated.

The build job does not set up a language runtime or run tests, linters, or
TechDocs generation. When you add code, add the matching setup and test steps
before `sam build`; see
[Common tasks](07-common-tasks.md#add-tests-to-the-pipeline). Do not assume
the build exercises deployed resources.

## Deploy job

`deploy` depends on `build` and only proceeds after it succeeds. It uses the
GitHub environment named **`production`** in both workflows.

1. The workspace action runs with `checkout_artifact: true` to retrieve the
   build artifact.
2. The credential action receives both `AWS_CICD_ACCOUNT_ID` and
   `DEPLOYMENT_ACCOUNT_ID` for build/deployment account access.
3. `ServerlessOpsIO/gha-deploy-aws-sam@v1` deploys with the target account ID
   and serialized workflow environment, variables, and secrets.

The shared actions implement artifact handling, credential assumptions, and
parameter/tag expression resolution. Their internals are external to this
repository. Confirm the expected IAM roles, region, artifact storage, and
stack-name behavior against the shared actions' configuration.

See [Configuration](03-configuration.md) for parameter files, tags, secrets,
and SAM defaults. GitHub environment protection rules can require deployment
approval or restrict which branches deploy.

## Feature branch builds and deployments

Branch support means a non-main push executes **both build and deploy**, not
just CI checks. `CodeBranch` is resolved from `$env.GITHUB_REF_SLUG_CS`, but
the starter template does not use it, so it does not isolate a deployment:

- `samconfig.toml` has a component-wide default stack name.
- Both workflows select the same GitHub environment, `production`.
- Resources with explicit names, such as a fixed table name or custom domain,
  collide across stacks unless their names include the branch.

Before enabling branch deployments, establish and verify a platform policy:

1. Give each deployed branch a distinct stack name through the shared
   deployment action's supported configuration.
2. Avoid fixed physical names, or derive them from `CodeBranch`, so branch
   stacks do not collide on account-unique or region-unique names.
3. Use an appropriate account/environment and OIDC trust policy for branch
   workloads, with approval rules that prevent unintended production changes.
4. Confirm the branch slug, final stack name, and resource names in the
   deployment output before sending test traffic.
5. Define teardown for branch stacks and any external resources when a branch
   is no longer needed. There is no branch-deletion cleanup workflow.

Without this policy, a branch push may update the same stack as `main`.
Do not assume a branch push creates a safe preview environment out of the box.

## Reviewing a deployment

Inspect the GitHub Actions build and deploy logs, then check the deployed
CloudFormation stack's status, events, resources, and outputs. Exercise the
resources you added with disposable data. Use
[Observability](06-observability.md) to correlate requests, errors, and
metrics.
