# Features

The empty template is a blank starting point. It supplies the project
scaffolding, catalog registration, and deployment pipeline that every
ServerlessOps AWS SAM project shares, and deliberately leaves the AWS
architecture, application code, and language tooling to you.

| Provided | Not provided |
| --- | --- |
| A SAM `template.yaml` with standard metadata parameters | Any AWS resources, including functions, APIs, or databases |
| Backstage `Component` registration and this handbook | API entities or catalog dependency relations |
| GitHub Actions build and deploy workflows | Unit, integration, or end-to-end tests or test steps |
| Deployment parameter and tag files | Language runtime, package manager, or dependency files |
| CloudFormation lint configuration | Logging, metrics, alarms, or dashboards configuration |

## AWS SAM project

`template.yaml` uses the `AWS::Serverless-2016-10-31` transform, so you can
declare both AWS SAM resources (such as `AWS::Serverless::Function`) and plain
CloudFormation resources. It declares the `Domain`, `System`, `Component`, and
`CodeBranch` parameters that the pipeline supplies on every deployment, and an
empty `Resources` section.

The empty `Resources` section must be filled before the project can build or
deploy. See [Common tasks](07-common-tasks.md#add-your-first-resource).

`samconfig.toml` sets the default stack name to the component name and the
deployment options the pipeline expects. See [Configuration](03-configuration.md).

## Backstage catalog and TechDocs

`catalog-info.yaml` registers a `Component` of type `iac` with the selected
owner, system, and lifecycle, plus a GitHub project-slug annotation. It also
points TechDocs at this handbook. As the project grows, add catalog relations
such as `dependsOn` or `providesApis` for the entities it relies on or exposes;
see [Common tasks](07-common-tasks.md#update-catalog-metadata).

## CI/CD and feature branches

GitHub Actions supplies separate main and non-main branch workflows. Both
validate and build with SAM, and deploy after a successful build.
Branch deployments need deliberate stack isolation; both workflows currently
select the `production` GitHub environment.
See [CI/CD pipeline](05-ci-cd.md) before pushing a feature branch.

## Deployment metadata and tagging

`cfn-parameters.json` passes the selected domain and system, the repository
name, and the branch slug into the stack's parameters. `cfn-tags.json` applies
`org:domain`, `org:system`, and `org:component` stack tags, which
CloudFormation propagates to resources that support tagging. These tags also
identify the project's resources in Datadog.
See [Configuration](03-configuration.md) and [Observability](06-observability.md).

## Language and tooling choices

The empty template does not choose a programming language. Its `.gitignore`
includes Python, virtualenv, and editor entries, plus `.aws-sam/`, but there is
no `Pipfile`, `requirements.txt`, package manifest, or test configuration.
When you add code, choose the runtime, dependency manager, and test framework
deliberately, and add a matching pipeline test step. See
[Common tasks](07-common-tasks.md#add-a-function).
