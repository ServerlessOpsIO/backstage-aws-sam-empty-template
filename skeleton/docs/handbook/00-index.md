# Project handbook

This handbook describes the empty AWS SAM project generated for
**${{ values.component_name }}**.
Keep it current as the project's infrastructure, code, and workflows grow.

- [Features](01-features.md): what the empty starter provides and what it leaves to you.
- [Creating a component](02-creating-a-component.md): the Backstage form and generated outputs.
- [Configuration](03-configuration.md): deployment parameters, tags, and prerequisites.
- [Architecture](04-architecture.md): deployment model, template structure, and repository layout.
- [CI/CD pipeline](05-ci-cd.md): build, deployment, and feature branches.
- [Observability](06-observability.md): logs, metrics, and automatic Datadog integration.
- [Common tasks](07-common-tasks.md): add resources, functions, tests, and pipeline steps.

## Read this first

The generated `template.yaml` declares **no resources** (`Resources: {}`).
`sam validate --lint` accepts it, but `sam build` rejects it with
`'Resources' section is required`, and CloudFormation requires at least one
resource. The initial push to `main` therefore fails in the build job and
deploys nothing. Add your first resource, as described in
[Common tasks](07-common-tasks.md#add-your-first-resource), before expecting a
successful pipeline run.

## Reading in Backstage

The component's `catalog-info.yaml` contains `backstage.io/techdocs-ref: dir:.`.
TechDocs uses the repository-root `mkdocs.yml` and the Markdown files in
`docs/handbook/`.
Open the component in the catalog and select **Docs**.

The platform must enable the TechDocs frontend and backend and configure either
on-demand documentation generation or an external builder and publisher. The
builder needs MkDocs and `mkdocs-techdocs-core` for the `techdocs-core` plugin.
The generated application workflows do not build or publish documentation.
With the Backstage TechDocs CLI and its Docker prerequisites installed, preview
from the generated repository root with `techdocs-cli serve`.

The pages can also be read directly in GitHub. Paths shown in backticks are
relative to the generated repository root.

## Content review

This handbook describes the starter as generated. Once you add resources and
code, update the [Architecture](04-architecture.md),
[Observability](06-observability.md), and [Common tasks](07-common-tasks.md)
pages to describe your service before treating the handbook as its operational
reference. Template-generated documentation does not constitute human approval.
