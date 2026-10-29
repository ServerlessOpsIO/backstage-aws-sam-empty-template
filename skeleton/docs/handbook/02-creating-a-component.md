# Creating a component via Backstage

Choose **Create** in Backstage and select **AWS Empty**.
The platform needs GitHub publishing, catalog registration, the template's
custom form fields, and catalog entities for the choices below.

## Page 1: Component Information

Paths below are relative to the **newly generated repository root**, not the
template source repository. Catalog selections generally contribute the entity's
name rather than its display title; the exceptions are noted below.

| Input | Purpose | Where to see it after creation |
| --- | --- | --- |
| Component name | Unique component and repository name. It is lowercased before use. | GitHub repository name; Component `metadata.name` and the `github.com/project-slug` annotation in `catalog-info.yaml`; `stack_name` in `samconfig.toml`; the first line of the top-level `Description` in `template.yaml`; headings in `README.md` and `docs/handbook/00-index.md`; `site_name` in `mkdocs.yml`. `Component` in `cfn-parameters.json` and `org:component` in `cfn-tags.json` resolve from the repository-name workflow variable rather than a literal form value. |
| Description | Describes the repository, component, and SAM application. | GitHub repository description; Component `metadata.description` in `catalog-info.yaml`; top-level `Description` in `template.yaml`; introductory text in `README.md`. |
| Owner | Owning group selected from your groups. | Component `spec.owner` in `catalog-info.yaml` uses `group:<selected-group-name>`; GitHub repository Settings → Custom properties → `owner` contains the selected group's **title**, not its entity name. |
| Domain | Catalog domain, also used for deployment metadata and tagging. | `Domain` in `cfn-parameters.json`; `org:domain` in `cfn-tags.json`; GitHub Settings → Custom properties → `domain`. There is no direct domain relationship in the generated `catalog-info.yaml`. |
| System | Catalog system filtered by the chosen domain. | Component `spec.system` in `catalog-info.yaml`; `System` in `cfn-parameters.json`; `org:system` in `cfn-tags.json`; GitHub Settings → Custom properties → `system`. The `README.md` heading uses the system's **title**. |
| Environment | Catalog environment filtered by the chosen system; narrows the cloud-account picker. | Scaffolder selection only: the chosen environment is not written to a generated file. `.github/workflows/main.yml` and `.github/workflows/branch.yml` independently set `jobs.deploy.environment: production`. |
| Account ID | Catalog cloud-account resource related to the environment; its `metadata.title` must contain the AWS account ID. | GitHub Settings → Secrets and variables → Actions → repository secret `DEPLOYMENT_ACCOUNT_ID`; both workflow files reference it in deployment steps. The account ID is not written into project files. GitHub shows the secret's name but does not reveal its saved value; confirm the source catalog entity's title before creation. |

Hidden defaults are lifecycle `production`, namespace `default`, and GitHub
organization `ServerlessOpsIO`.

Inspect lifecycle in the Component's `spec.lifecycle` in `catalog-info.yaml`.
The GitHub organization value appears in the
`metadata.annotations.github.com/project-slug` annotation; the publish step
itself always publishes to `ServerlessOpsIO`. Namespace is passed to the
skeleton but not written as `metadata.namespace`; it remains a scaffolder input
only.

Choosing an Environment does not configure a matching GitHub environment:
the generated workflows both explicitly use `production`.

The empty template has no further form pages: there are no software
configuration inputs and no optional Copilot or Spec Kit steps.

## Review and create

Review the form summary, especially component name and account, before
submitting. The scaffolder then:

1. Fetches the selected domain, system, owner, and account catalog entities.
2. Copies the project skeleton, including this handbook, and the GitHub
   Actions workflows.
3. Publishes a **public** repository with `main` as the default branch, sets
   `DEPLOYMENT_ACCOUNT_ID`, and sets the `system`, `domain`, and `owner` custom
   properties.
4. Registers the generated Component in Backstage from `/catalog-info.yaml`.

The template defines no output links; find the new repository under the
`ServerlessOpsIO` GitHub organization and the component in the catalog.
Default branch protection is disabled and the required approving review count
is zero in the publisher configuration; apply your review policy before
development.

Publishing the initial commit to `main` triggers the main workflow. Because
the starter declares no resources, that first run fails at `sam build`; see
[Handbook](00-index.md#read-this-first).

## After creation

- Confirm catalog ownership and relationships.
- Complete [deployment configuration](03-configuration.md) before expecting a
  deployment to succeed.
- Review [branch deployment isolation](05-ci-cd.md) before pushing additional branches.
- Open the component's **Docs** tab, or read `docs/handbook/00-index.md` in the repository.
- Add your first resource using [Common tasks](07-common-tasks.md), update
  `README.md` and this handbook to describe the service, and request human
  review of the result.
