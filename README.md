# AWS SAM Empty Backstage Template

This repository is a Backstage scaffolder template for creating a new AWS Serverless Application Model (SAM) project that starts with no infrastructure resources.

It is intended as a blank starting point for teams that want to generate a new service repository from Backstage, publish it to GitHub, and register it in the catalog without predefining Lambda functions, APIs, or databases.

## What this template creates

When used from Backstage, it generates a new repository skeleton with:

- A minimal AWS SAM `template.yaml` that is intentionally empty
- Backstage catalog metadata in `catalog-info.yaml`
- Standard project files such as `.gitignore`, `samconfig.toml`, and deployment parameter/tag config
- A starter `README.md` for the generated project
- GitHub Actions workflows for build and deployment via AWS SAM

## Repository layout

- `template.yaml` — the Backstage scaffolder template definition
- `skeleton/` — files copied into the generated project
- `pipeline/` — GitHub Actions pipeline templates used by the generated project

## Typical use in Backstage

The template prompts for component metadata such as:

- component name and description
- owning group
- domain and system
- deployment environment
- target AWS account

It then:

1. fetches related catalog entities,
2. copies the project skeleton,
3. creates the deployment pipeline files,
4. publishes the repository to GitHub, and
5. registers the generated component in the Backstage catalog.

## Next steps

After generating a service from this template:

- add Lambda functions, APIs, DynamoDB tables, or other resources to `template.yaml`
- customize build and deploy settings in the generated GitHub Actions workflow
- update the generated project README and metadata to match the service purpose

This template is intentionally minimal and intentionally empty so that teams can add the exact AWS architecture needed for each service.