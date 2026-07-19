---
name: cloud-run
description: Deploy services, jobs, and worker pools to Cloud Run from source or container images, with env vars, secrets, scaling, and custom domains. Use when the user is shipping an app or background worker to Google Cloud serverless.
---

# Cloud Run

Three workload shapes: **services** (HTTP, scale on requests, scale to
zero), **jobs** (run to completion, parallelizable), **worker pools** (GA
2026: long-lived pull-based workers, no URL, about 40% cheaper than
services for background work).

## Deploy a service

```bash
# From source (Buildpacks build in Cloud Build, no Dockerfile needed;
# a Dockerfile in the directory is used automatically if present)
gcloud run deploy myapp --source . --region europe-west1 \
  --allow-unauthenticated

# From an image (build/push first, Artifact Registry not gcr.io)
gcloud builds submit -t europe-west1-docker.pkg.dev/PROJ/repo/myapp:SHA
gcloud run deploy myapp \
  --image europe-west1-docker.pkg.dev/PROJ/repo/myapp:SHA \
  --region europe-west1
```

The container must listen on `$PORT` (default 8080) on `0.0.0.0`. Python
source deploys support `pyproject.toml` and auto-detect FastAPI, Gradio,
and Streamlit entrypoints.

Private services: drop `--allow-unauthenticated`; callers need
`roles/run.invoker` and send an identity token
(`Authorization: Bearer $(gcloud auth print-identity-token)`).

## Env vars and secrets

```bash
gcloud run deploy myapp --source . \
  --set-env-vars LOG_LEVEL=info,MODE=prod \
  --set-secrets DATABASE_URL=db-url:latest,API_KEY=api-key:2 \
  --service-account app-sa@PROJ.iam.gserviceaccount.com
```

`--set-secrets` mounts Secret Manager values as env vars (or file paths
with `/path=secret:version`). The service's runtime SA needs
`roles/secretmanager.secretAccessor` on each secret. Always deploy with a
dedicated runtime SA; the default compute SA is over-privileged.
Pinning `:latest` reads the newest version at instance start, not live.

## Scaling and performance knobs

- `--min-instances 1` kills cold starts for latency-sensitive services
  (billed while warm); `--max-instances` caps runaway scale and cost.
- `--concurrency` (default 80): requests per instance; set to 1 for
  CPU-bound or non-thread-safe work.
- `--cpu`, `--memory`, `--cpu-boost` (faster startup);
  `--no-cpu-throttling` for background threads in a service (billed
  continuously).
- GPUs (`--gpu 1 --gpu-type nvidia-l4`): GA for services, jobs, and worker
  pools; NVIDIA RTX PRO 6000 Blackwell available in select regions; scale
  to zero applies.
- Rollouts: `gcloud run deploy ... --no-traffic` then
  `gcloud run services update-traffic myapp --to-revisions REV=10` for
  canaries; `--to-latest` to finish.

## Jobs and worker pools

```bash
gcloud run jobs create nightly --image IMG --region europe-west1 \
  --tasks 10 --parallelism 5 --max-retries 3 --task-timeout 30m
gcloud run jobs execute nightly --wait
gcloud scheduler jobs create http nightly-cron --schedule "0 3 * * *" \
  --uri "https://run.googleapis.com/apis/run.googleapis.com/v1/..." \
  --oauth-service-account-email scheduler-sa@PROJ.iam.gserviceaccount.com
```

Jobs: batch/cron work, per-task retries, no HTTP server needed. Worker
pools: continuous consumers (Pub/Sub pull, Kafka, task queues), deploy
with `gcloud run worker-pools deploy`; they never scale on HTTP and have
no endpoint, you control instance count (manual or autoscaled by lag).

## Custom domains

Preferred: a global external Application Load Balancer with a serverless
NEG backend pointing at the service (works everywhere, gives you CDN,
Cloud Armor, and multi-region). `gcloud run domain-mappings create` is the
quick path but is regional and limited; for production traffic use the
load balancer.

## Gotchas

- Requests cap at 60 min; WebSockets count against it.
- Filesystem is in-memory and per-instance (writes eat your RAM); use GCS
  mounts (`--add-volume ... type=cloud-storage`) or the newer ephemeral
  disk (preview) for scratch space.
- Logs: stdout/stderr JSON lines land in Cloud Logging automatically
  (`gcloud run services logs read myapp` or Logs Explorer).
- Deploy-from-source needs Cloud Build + Artifact Registry APIs enabled
  and the Cloud Build SA granted on first run; `gcloud` prompts, CI does
  not.
