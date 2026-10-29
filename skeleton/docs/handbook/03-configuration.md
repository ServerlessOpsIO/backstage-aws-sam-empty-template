# Configuration

## Stack parameters: `cfn-parameters.json`

This file supplies the deployment action with values for the `Parameters` in
`template.yaml`. The file is named **`cfn-parameters.json`**, without a leading
underscore.

| Parameter | Starter value/source | Meaning |
| --- | --- | --- |
| `Domain` | Selected catalog domain name | Application domain metadata. |
| `System` | Selected catalog system name | Application system metadata. |
| `Component` | `$env.GITHUB_REPOSITORY_NAME_PART_SLUG_CS` | Repository-derived component identifier provided through the workflow environment. |
| `CodeBranch` | `$env.GITHUB_REF_SLUG_CS` | Branch slug of the deployed ref. |

The starter template declares these parameters but no resource uses them yet.
Reference them where useful, for example in resource names, a Lambda
environment variable, or an API stage name. Keep every key in
`cfn-parameters.json` declared in `template.yaml`, and give any new parameter a
value here or a `Default` in the template.

The `$env` expressions are intentionally unquoted in the starter file. It is
deployment-action input, **not strict JSON** for direct consumption by
`json.load`, `jq`, or the SAM CLI. The same applies to expressions in
`cfn-tags.json`. The workflow passes `env_json`, `vars_json`, and
`secrets_json` to `ServerlessOpsIO/gha-deploy-aws-sam@v1` for resolution, so
values can also reference `$vars.<NAME>` or `$secrets.<NAME>`.
Consult that action's configuration when changing expression syntax or adding
environment-specific values.

For a manual deployment, supply resolved CloudFormation parameter values through
SAM's parameter overrides; do not pass this file as though it were the AWS CLI's
parameter-list format. Do not replace references with committed credentials.

## Stack tags: `cfn-tags.json`

This file is also named without a leading underscore. It defines:

| Tag | Value |
| --- | --- |
| `org:domain` | Selected domain name |
| `org:system` | Selected system name |
| `org:component` | `$env.GITHUB_REPOSITORY_NAME_PART_SLUG_CS` |

Keep these aligned with catalog metadata and your organization's inventory and
cost-allocation conventions. Add tags through the existing deployment action;
CloudFormation propagates stack tags to resources that support them.
These metadata tags are not runtime configuration or access controls.

## Deployment prerequisites

- A target AWS account and deployment region with the required IAM permissions
  for the resources you add.
- GitHub Actions secrets `AWS_CICD_ACCOUNT_ID` and `DEPLOYMENT_ACCOUNT_ID`,
  available to the relevant jobs. The scaffolder only explicitly creates
  `DEPLOYMENT_ACCOUNT_ID`; configure the other through your organization's
  repository or environment settings.
- AWS OIDC trust and build/deployment roles expected by the shared
  `gha-assume-aws-credentials` action. Account IDs alone do not grant deployment
  access. Verify role names, trust restrictions, region, and artifact storage
  against the shared action's requirements.
- GitHub environment `production` settings appropriate for **both** workflows,
  including any required approvals and branch restrictions.
- At least one resource in `template.yaml`; see
  [Common tasks](07-common-tasks.md#add-your-first-resource).

Resources you add may bring their own prerequisites, such as a hosted zone for a
custom domain or an existing identity provider. Record them in this section.

## SAM settings

`samconfig.toml` sets the default stack name to the component name, disables
changeset confirmation, and requests `CAPABILITY_NAMED_IAM`. It does not define
separate branch environments or a region. Verify the final stack name and region
used by the shared deployment action before deploying.

`CAPABILITY_NAMED_IAM` allows the stack to create IAM resources with custom
names. If you add nested applications (`AWS::Serverless::Application`) or
macros, you may also need `CAPABILITY_AUTO_EXPAND`.

`.cfnlintrc` configures CloudFormation linting for `sam validate --lint`.
It ignores check `E3034`. Add further ignores only with a comment explaining why.

The stack name is not branch-qualified by default. See
[CI/CD pipeline](05-ci-cd.md) before deploying multiple branches to avoid stack
and resource-name collisions.
