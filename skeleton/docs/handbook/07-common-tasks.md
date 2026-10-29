# Common changes and tasks

## Set up a development environment

Install the AWS SAM CLI. From the **generated repository root**, run:

```sh
sam validate --lint
sam build
```

`sam build` fails until `template.yaml` declares at least one resource.
SAM validation and build need no deployed resources, but some commands need AWS
credentials and a region configured. Add your language toolchain when you add
code.

## Add your first resource

The starter's `Resources: {}` must contain at least one resource before the
project can build or deploy. Replace it with the resources your service needs.
For example, a DynamoDB table:

```yaml
Resources:
  DdbTable:
    Type: AWS::DynamoDB::Table
    Properties:
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: pk
          AttributeType: S
        - AttributeName: sk
          AttributeType: S
      KeySchema:
        - AttributeName: pk
          KeyType: HASH
        - AttributeName: sk
          KeyType: RANGE
```

1. Prefer generated physical names. If a resource needs an explicit name,
   include `Component` and, for branch deployments, `CodeBranch`, using
   `Fn::Sub`.
2. Grant only the permissions each resource needs.
3. Add `Outputs` for values others need after deployment, such as endpoints
   and ARNs.
4. Record new external prerequisites in [Configuration](03-configuration.md) and
   update the diagrams in [Architecture](04-architecture.md).
5. Run `sam validate --lint` and `sam build`.

## Add a function

### Function code and infrastructure

1. Choose a supported Lambda runtime and create `src/handlers/<FunctionName>/`
   with the handler and its dependency manifest, such as `requirements.txt` or
   `package.json`.
2. Keep the entry point thin and separate business logic so it can be unit
   tested without a Lambda context.
3. Add an `AWS::Serverless::Function` to `template.yaml` with its `CodeUri`,
   `Handler`, and `Runtime`. Move settings shared by several functions, such as
   runtime, timeout, memory, and environment variables, into `Globals.Function`.
4. Grant only the permissions the function needs, for example with a scoped SAM
   policy template such as `DynamoDBReadPolicy` rather than a broad managed
   policy.
5. Add its trigger with SAM `Events`, or define the event source resources and
   invoke permissions explicitly.
6. Declare its log group with a retention period; see
   [Observability](06-observability.md#logging).

```yaml
Globals:
  Function:
    Runtime: python3.13
    Timeout: 5
    MemorySize: 128

Resources:
  ExampleFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/handlers/ExampleFunction
      Handler: function.handler

  ExampleFunctionLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub "/aws/lambda/${ExampleFunction}"
      RetentionInDays: 7
```

### Dependencies

- SAM builds each function from the dependency manifest in its own `CodeUri`
  directory. Keep function-specific dependencies there rather than in shared
  code, so they do not enlarge every function's package.
- Keep local development dependencies, such as test tools, separate from
  runtime dependencies so they are not packaged.
- Pin versions, commit lockfiles, and review advisories before adding a
  package. Check that `sam build` includes the dependencies you expect; a
  working local install is not proof that a deployment artifact is complete.

### Tests

1. Add unit tests under `tests/unit/` and sample events under `data/`. Use
   `sam local generate-event` to create starting events for common sources.
2. Mock AWS calls so tests never contact a real AWS account. Use synthetic data
   and no real records or tokens.
3. Cover malformed input, error paths, and missing records as well as success.
4. Unit tests cannot prove that IAM, triggers, or the deployed dependency bundle
   work; verify those after an isolated deployment.

## Add tests to the pipeline

The workflows do not run tests. Add language setup and test steps to the
`build` job in **both** `.github/workflows/main.yml` and
`.github/workflows/branch.yml`, after workspace setup and before AWS
credentials are assumed, so a failing test stops deployment. For example, the
Python templates use:

```yaml
      - name: Setup Python environment
        uses: ServerlessOpsIO/gha-setup-python@v1
        with:
          python_version: '3.13'

      - name: Run tests
        run: pipenv run test-unit
```

Match the setup action and test command to your chosen language.

## Update catalog metadata

- Add `spec.dependsOn` for catalog resources and components the project relies
  on, and `spec.providesApis` with an `API` entity if it exposes one.
- Change `spec.type` from `iac` if the component becomes a service, website, or
  library, so catalog filters reflect what it is.
- Keep `README.md` and this handbook current as the project changes; add pages to
  `nav` in `mkdocs.yml` when you add handbook pages.

## Validate and deploy changes

Run `sam validate --lint`, `sam build`, and any tests you have added.

Review IAM permissions and packaged dependencies, commit the related template,
code, tests, and handbook updates, and open a PR under your review policy.
Remember that **pushing a feature branch starts deployment**: complete the
[branch-isolation setup](05-ci-cd.md) before pushing.

After deployment, check the stack's events and outputs, exercise the new
resources with disposable data, inspect logs and metrics, and obtain human
review.
