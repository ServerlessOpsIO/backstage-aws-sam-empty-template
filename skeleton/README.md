# ${{ values.system_title }} / ${{ values.component_name }}

${{ values.component_description }}

The [project handbook](docs/handbook/00-index.md) covers features, project creation,
configuration, architecture, CI/CD, observability, and common development tasks.
It is also available in the component's **Docs** tab when Backstage TechDocs is enabled.


## New Project Getting Started
This repository was generated from an empty template. It uses the [AWS Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/) to define the infrastructure as code.

`template.yaml` starts with no resources, and `sam build` fails until it declares at least one. Add your first resource before expecting the pipeline to deploy; see [Common tasks](docs/handbook/07-common-tasks.md#add-your-first-resource).
