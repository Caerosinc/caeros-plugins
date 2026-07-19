---
name: aws-serverless
description: Build serverless systems on Lambda, API Gateway, Step Functions, and EventBridge, with SAM for local dev and deploys. Use when the user is writing Lambda functions, event-driven pipelines, or serverless APIs on AWS.
---

# AWS Serverless

## Lambda

Current runtimes to target (mid-2026): `nodejs22.x`, `nodejs24.x`,
`python3.13`, `python3.14`, `java21`, `provided.al2023`. `nodejs20.x` hit
deprecation April 2026 (creation blocked from August 2026); anything on an
Amazon Linux 2 base is on the way out (AL2 EOL June 2026). Check the live
runtime table before pinning: invocations never stop, but updates do.

Defaults that matter:

- `arm64` (Graviton) is cheaper per ms than `x86_64` and usually faster;
  pick it unless a native dependency blocks you.
- Memory sets CPU share: tune with power tuning, not guesswork. 128 MB to
  10,240 MB; 15 min max duration; 6 MB sync request/response payload;
  `/tmp` configurable 512 MB to 10 GB.
- Cold starts: keep bundles small, init outside the handler, use
  provisioned concurrency only for latency-critical paths, or Lambda
  SnapStart (Java, Python, .NET) which is cheaper than provisioned.
- Grant per-function IAM roles; never share one broad role.

```bash
aws lambda invoke --function-name fn --payload '{"a":1}' \
  --cli-binary-format raw-in-base64-out /dev/stdout
aws logs tail /aws/lambda/fn --follow
```

## API Gateway

- **HTTP API** (`apigatewayv2`): cheaper, lower latency, JWT authorizers
  built in. Default choice for Lambda-backed REST-ish APIs.
- **REST API** (`apigateway` v1): pick only for usage plans + API keys,
  request validation models, WAF on edge-optimized, or private endpoints.
- Function URLs: zero-config HTTPS for a single function (IAM or public
  auth), good for webhooks; no routing, throttling, or custom authorizers.

## Step Functions

- **Standard**: exactly-once, up to 1 year, priced per state transition.
  Use for long-lived orchestration and human-in-the-loop.
- **Express**: at-least-once, 5 min max, priced on duration/memory. Use for
  high-volume event processing; can be invoked synchronously.
- Author in JSONata mode (`"QueryLanguage": "JSONata"`) for transforms
  instead of chained intrinsic functions; use variables to avoid threading
  state through every step.
- Prefer direct SDK integrations (`arn:aws:states:::aws-sdk:...`) over
  glue Lambdas that only call one API.

## EventBridge

- Custom buses per domain; rules match on event pattern, fan out to Lambda,
  Step Functions, SQS, API destinations.
- **Pipes**: point-to-point source -> enrich -> target (SQS to Lambda with
  filtering) without glue code.
- **Scheduler**: cron and one-time schedules with flexible time windows;
  use it instead of legacy CloudWatch Events rules for cron.
- Always configure DLQs on rule targets and set `MaximumEventAgeInSeconds`;
  silent event drops are the classic failure mode.

## SAM workflow

```bash
brew install aws-sam-cli           # or pip install aws-sam-cli
sam init                           # scaffold from template
sam build                          # containerized builds: --use-container
sam local invoke MyFn -e event.json
sam local start-api                # local API Gateway emulation
sam deploy --guided                # first time; writes samconfig.toml
sam sync --stack-name dev-stack --watch   # fast iterate against a dev stack
sam logs -n MyFn --stack-name dev-stack --tail
```

`sam sync --watch` is the dev loop (infra + code sync in seconds);
`sam deploy` through CI is the production path. SAM templates are
CloudFormation with the `AWS::Serverless::*` transform, so anything from
the CDK/CloudFormation skill applies.

## Gotchas

- Async invokes (S3, SNS, EventBridge -> Lambda) retry twice then drop:
  configure `OnFailure` destinations or DLQs everywhere.
- SQS -> Lambda: set queue `visibilityTimeout` to at least 6x the function
  timeout, and use `ReportBatchItemFailures` for partial batch success.
- API Gateway hard 29 s integration timeout (raisable via quota request on
  REST APIs): stream or go async for slow work.
- Idempotency is on you: at-least-once delivery is the norm end to end.
