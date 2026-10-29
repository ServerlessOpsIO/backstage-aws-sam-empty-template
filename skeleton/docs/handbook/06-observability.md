# Observability

The starter deploys no resources, so it emits no logs or metrics of its own.
This page describes what the platform provides automatically and what to
configure as you add resources. Replace the general guidance with
service-specific details as the project grows.

## Deployment visibility

Before any resources exist, observability is limited to the pipeline:

- GitHub Actions logs for the build and deploy jobs.
- CloudFormation stack events in the deployment account, which show each
  resource's create, update, and delete status and the reason for any failure.

## Logging

Lambda functions write to `/aws/lambda/<physical-function-name>` in CloudWatch.
Find the physical name in the stack resources or Lambda console. Lambda creates
that log group on first invocation without a retention policy unless you
declare it; declare log groups in `template.yaml` with an explicit
`RetentionInDays` so retention is deliberate and the group is deleted with the
stack.

Other services, such as API Gateway access logs, Step Functions, or ECS, need
their logging configured explicitly. API Gateway REST API logging also needs the
account/region-level CloudWatch logging role; confirm the platform has
configured it before relying on execution logs.

Use structured logs with invocation context, for example Powertools for AWS
Lambda for Python, TypeScript, Java, or .NET. Never log tokens, credentials,
or unredacted personal data, and sanitize request payloads before logging them.

## Metrics

AWS publishes standard metrics for most services, for example Lambda
Invocations, Errors, Duration, and Throttles; API Gateway Count, Latency, and
5XXError; and DynamoDB throttling and latency. Filter by the actual deployed
resource names, especially when investigating branch deployments. Watch error
rates and throttles alongside latency, not just invocation counts.

The starter does not emit custom business metrics, configure alarms or
dashboards, or enable X-Ray tracing. Add these intentionally based on your
service's reliability targets.

## Availability in Datadog

The Datadog AWS integration and log shipping are **configured automatically in
all AWS accounts**. Developers do not need to enable the integration or
configure log forwarding for generated projects. Use Datadog to inspect the
project's AWS metrics and shipped logs once resources are deployed.

Baseline tagging is also automatic: `cfn-tags.json` supplies `org:domain`,
`org:system`, and `org:component`, and `ServerlessOpsIO/gha-deploy-aws-sam` adds
additional deployment tags. Use the existing tags to locate and distinguish
resources; extra tagging is only needed for service-specific requirements.

After the first deployment with resources, verify that a test log and recent
metrics appear in Datadog.

In Datadog Logs Explorer, locate the deployed service or resource identifier
and narrow the time range, then search the request ID from an error.
Use Metrics Explorer to inspect imported AWS metrics and account/region
dimensions. Exact field names and tags depend on the integration's pipelines.
If data is missing, first confirm it exists in CloudWatch and check the Datadog
time range and resource/tag filters. Treat persistent missing data as a problem
with the existing collection or shipping configuration, not a requirement to
set up a new integration.

## Investigation checklist

1. Note the timestamp, operation, resource, and any returned request ID.
2. Check whether a recent deployment changed the resource, using the
   CloudFormation stack events and GitHub Actions history.
3. Inspect the resource's CloudWatch logs and errors, without copying
   sensitive payloads into tickets.
4. Compare error, latency, and throttling metrics with recent deployments.
5. Check Datadog ingestion separately if CloudWatch data is present but missing
   from Datadog.
