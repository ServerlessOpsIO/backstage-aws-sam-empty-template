<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0
- Modified principles: none.
- Modified sections: Repository-Specific Constraints now names the generated project handbook;
  Development Workflow and Quality Gates adds a handbook evaluation gate for new features.
- Added sections: none.
- Removed sections: none.
- Templates requiring updates: none; plan-template.md derives its Constitution Check from this file.
- Follow-up TODOs: template.yaml hardcodes the publish organization despite github_org input;
  generated catalog-info.yaml lacks dependency/API relations; template.yaml has no output.links;
  document the cloud-account title/AWS account ID contract next to its use.
-->

# backstage-aws-sam-empty-template Constitution

## Core Principles

### I. Every Option Combination Renders Correctly

- Every supported combination of template inputs MUST render a complete, valid project.
  A form choice MUST NOT produce a broken file.
- Content inside a `{% if %}` block MUST NOT be referenced from outside that block. Every variable
  used by a skeleton, including in file and directory names, MUST be passed in the `fetch:template`
  `values` for each template that renders it.
- Rendered JSON and YAML MUST be valid for every supported input combination, including commas and
  indentation around conditional blocks. GitHub Actions expressions `${{ }}` in pipeline
  skeletons MUST be wrapped in `{% raw %}...{% endraw %}`.
- Generated projects MUST pass their pipeline validation and build steps without hand edits. The
  current `pipeline/.github/workflows/` workflows run `sam validate --lint` and `sam build`.
- Changes to skeletons or input options MUST be verified by rendering every affected combination,
  not only the default path.

### II. The Catalog Drives the Inputs

- Ownership and placement MUST come from catalog pickers with `allowArbitraryValues: false`.
  Pickers MUST be ordered and filtered by the preceding selection: owner (`MyGroupsPicker`),
  Domain, System filtered by Domain, then Environment and cloud account filtered by System and
  Environment as applicable.
- Free-text fields MUST NOT replace values that exist as catalog entities. Values returned by
  `catalog:fetch` MUST come from the entity they belong to. Changing the order of `entityRefs` or
  adding to it MUST update every result lookup by index.
- Implicit catalog contracts MUST be documented in `template.yaml` beside their use. In
  particular, document that the selected cloud-account entity's title supplies the AWS account ID
  passed to the generated repository.

### III. Published Repositories and Entities Are Predictable

- The GitHub repository name and generated Component entity name MUST be the same value: the
  component name input lowercased, with no suffix in the current `aws-sam-empty` template.
- The GitHub organization MUST come from the hidden `github_org` parameter, defaulting to
  `ServerlessOpsIO`; it MUST NOT be hard-coded in steps or skeletons.
- Each generated root `catalog-info.yaml` MUST set `owner`, `system`, `lifecycle`, the
  `github.com/project-slug` annotation, and a `spec.type` matching the artifact. The current
  template creates a `Component` with `spec.type: iac`. It MUST also declare `dependsOn` or
  `providesApis` relations for the catalog entities the project relies on or exposes.
- Project templates MUST publish with `main` as the default branch, set repository custom
  properties for domain, system, and owner from catalog selections, and register from
  `/catalog-info.yaml` at the repository root.
- Repository visibility, branch protection, and review requirements MUST be consistent across
  templates. The current template publishes public repositories with default-branch protection
  disabled and zero required approving reviews; any difference MUST be explained in that template.

### IV. Steps Are Ordered, Optional, and Fail Safely

- Project-template steps MUST run in this order: fetch entities; render skeletons; render
  pipelines; add optional starters; publish; register; then run optional steps that act on the
  published repository or external services.
- Optional steps MUST be off unless enabled by the user, guarded with `if:`, and their inputs MUST
  be shown and required only when enabled.
- Steps that call external services, including Copilot, Spec Kit, or Datadog, MUST run after
  catalog registration so a failure leaves a registered project.
- Inline shell in a workflow step MUST start with `set -eu` and stop on unexpected state.
  Comments MUST explain non-obvious behavior next to the relevant step, including explicit values,
  external-resource matching, and filter dependencies.
- The current `aws-sam-empty` template has no optional steps; any added optional step MUST follow
  these rules.

### V. Credentials and Secrets Stay Out of Templates

- Secrets and account-specific values MUST reach generated repositories only through scaffolder
  secrets and the `secrets` input of `publish:github`. They MUST NOT be written into skeleton
  files, template defaults, or this repository. Generated pipelines MUST refer to secrets by name
  for runtime resolution.
- Actions taken on a user's behalf, including future Copilot or Spec Kit steps and work on existing
  repositories, MUST use that user's GitHub credentials requested with `SoRequestUserCredentials`
  into `USER_GITHUB_TOKEN`, with only the required scopes.
- Tokens MUST NOT be written to files or git remotes, or included in logs or pull-request text.
  Commits MUST be authored as the user running the template, using their catalog profile name and
  email.

### VI. The Form Is Honest and Templates Are Consistent

- Parameter titles and descriptions MUST accurately describe their fields, including hidden fields.
  Defaults MUST be sensible values users can accept as-is. Fields applying only to particular
  choices MUST be shown and required through JSON Schema `dependencies`. An enum option MUST NOT be
  offered unless it renders a working project.
- Step ids and names MUST describe their actions. Templates MUST share the same first parameter
  page, metadata conventions for owner, tags, and description format, and `publish:github` settings.
  A change to a shared field MUST be applied to every template that has it unless the difference is
  explained.
- Every template's `output.links` MUST link to what it created: the repository or pull request, the
  catalog entity, and any agent task or external resource created by an optional step.

## Repository-Specific Constraints

- This repository contains one project template, `template.yaml`, named `aws-sam-empty`. It
  creates a new GitHub repository from `skeleton/`, whose generated catalog entity is a Backstage
  `Component` of type `iac`.
- `skeleton/` contains the SAM starter project, root `catalog-info.yaml`, and the generated
  project handbook in `skeleton/docs/handbook/`, published through `skeleton/mkdocs.yml` and
  TechDocs. `pipeline/` contains the generated GitHub Actions workflows under
  `.github/workflows/`; the `main.yml` workflow builds on `main` and deploys, while `branch.yml`
  handles non-`main` branches.
- The form's catalog selections are owner, Domain, System, Environment, and cloud account. The
  template currently has no workflow template and no optional external-service steps.

## Development Workflow and Quality Gates

- Before merge, run affected templates through the Backstage template editor dry run and render
  every affected input combination to verify Principle I. Changes to publishing, registration,
  optional external steps, or workflow-template git behavior MUST be exercised in a real run
  against a test component or sandbox. The pull request MUST list what was run.
- Changes to template inputs, generated files, required catalog entities, required secrets,
  required Backstage actions, or other prerequisites MUST update `README.md` in the same change,
  and the generated project README when affected. A new or changed scaffolder action MUST be named
  in the README along with the Backstage module that provides it.
- When creating a new feature, its specification and plan MUST evaluate the feature against the
  generated project handbook in `skeleton/docs/handbook/` and record whether the handbook requires
  updating. Features that change generated files, form inputs, prerequisites, pipelines, or
  developer workflows MUST update the affected handbook pages, and `skeleton/mkdocs.yml` when
  pages are added or removed, in the same change. A feature needing no handbook change MUST state
  why.
- Generated, operating-system, and editor files, including `.DS_Store`, `__pycache__`, and
  `node_modules`, MUST NOT be committed.
- Work on `feature/` or `bug/` branches that reference an issue number when one exists. Use
  conventional-commit messages such as `feat:`, `fix:`, or `refactor:`, and merge through pull
  requests.

## Governance

This constitution supersedes conflicting conventions in this repository. Every specification,
plan, and pull request MUST check compliance with it. Amendments MUST be made in the same change
that introduces the new rule and MUST include a Sync Impact Report.

Versioning follows semantic versioning: MAJOR removes or incompatibly redefines a principle, MINOR
adds a principle or section or materially expands guidance, and PATCH clarifies wording or makes
non-semantic refinements.

**Version**: 1.1.0 | **Ratified**: 2026-10-29 | **Last Amended**: 2026-10-29
