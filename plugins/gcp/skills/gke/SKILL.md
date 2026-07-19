---
name: gke
description: Run Kubernetes on GKE, choosing Autopilot vs Standard, wiring kubectl auth, Workload Identity for keyless pod auth, and node pool management. Use when the user is creating, operating, or debugging a GKE cluster.
---

# GKE (Google Kubernetes Engine)

## Autopilot vs Standard

- **Autopilot** (default and recommended): Google manages nodes, you pay
  per pod resource request, security hardening is on by default, no
  SSH/daemon-level node access. Pick it unless you need something it
  forbids.
- **Standard**: you manage node pools and pay per node. Pick for GPUs/TPUs
  with exotic configs, privileged DaemonSets, custom node images, or
  squeezing bin-packing efficiency beyond per-pod billing.

```bash
gcloud container clusters create-auto my-cluster --region europe-west1
gcloud container clusters create my-std --region europe-west1 \
  --num-nodes 1 --enable-autoscaling --min-nodes 0 --max-nodes 5
```

Autopilot enforces resource requests (it sets defaults if you omit them)
and rejects privileged pods and most host-level access; if a manifest
fails admission, read the constraint in the error, it is usually the
point.

## kubectl auth

```bash
gcloud components install gke-gcloud-auth-plugin   # once per machine
gcloud container clusters get-credentials my-cluster --region europe-west1
kubectl get nodes
```

`get-credentials` writes a kubeconfig entry that delegates to the
`gke-gcloud-auth-plugin`, so kubectl rides your gcloud login. The old
in-kubectl auth provider is gone; `gke-gcloud-auth-plugin: not found`
means install the component (or apt package `google-cloud-cli-gke-gcloud-auth-plugin`).
Cluster RBAC then maps your Google identity: grant teammates
`roles/container.developer` (project level) plus Kubernetes RBAC
RoleBindings for namespace scoping.

## Workload Identity Federation for GKE (keyless pod auth)

Never mount service account keys into pods. WIF for GKE is enabled by
default on Autopilot; on Standard enable `--workload-pool=PROJ.svc.id.goog`.

Modern direct binding (no intermediate Google SA needed): grant IAM roles
straight to the Kubernetes ServiceAccount principal:

```bash
kubectl create serviceaccount app-ksa -n prod
gcloud projects add-iam-policy-binding PROJ \
  --role roles/storage.objectViewer \
  --member "principal://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/PROJ.svc.id.goog/subject/ns/prod/sa/app-ksa"
```

Then set `serviceAccountName: app-ksa` in the pod spec; ADC inside the pod
just works. Legacy pattern (KSA annotated with
`iam.gke.io/gcp-service-account` + `roles/iam.workloadIdentityUser` on a
Google SA) still functions and is what most older docs show.

## Node pools (Standard only)

```bash
gcloud container node-pools create spot-pool --cluster my-std \
  --region europe-west1 --spot --machine-type e2-standard-4 \
  --enable-autoscaling --min-nodes 0 --max-nodes 10
gcloud container node-pools update default-pool --cluster my-std \
  --region europe-west1 --enable-autoupgrade --enable-autorepair
```

- Separate pools per shape: general, spot (with taints + tolerations for
  interruptible work), GPU (`--accelerator type=nvidia-l4,count=1`).
- Scale-to-zero pools work; pair with cluster autoscaler. Node
  auto-provisioning (NAP) creates pools on demand from pod requirements.
- Upgrades: clusters ride release channels (rapid/regular/stable); use
  maintenance windows, and PodDisruptionBudgets so drains do not take out
  your service.

## Day-2 essentials

- Deploy images from Artifact Registry in the same region; grant the node
  SA or workload `roles/artifactregistry.reader`.
- `kubectl describe pod` first, then `kubectl logs --previous` for crash
  loops; GKE ships container logs to Cloud Logging automatically.
- Cost: Autopilot bills requests, so right-size them; Standard needs
  bin-packing review (`kubectl top`, GKE cost insights). Spot for anything
  restartable.
- Gateway API is the modern ingress on GKE (`gke-l7-global-external-managed`
  GatewayClass); prefer it over legacy Ingress for new work.
