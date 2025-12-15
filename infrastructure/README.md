# Kreuzberg GKE Infrastructure

OpenTofu/Terraform infrastructure for GKE-based GitHub Actions runners.

## Configuration

- **Project**: kreuzberg-481219
- **Region**: europe-west3 (Frankfurt, Germany)
- **Cluster**: gh-runner-kreuzberg-gke-runners
- **Node Pool**: runner-pool (e2-medium: 2 vCPU, 4GB RAM)
- **Autoscaling**: 1-4 nodes across europe-west3-a and europe-west3-b
- **Purpose**: Self-hosted GitHub Actions runners for Docker builds and Rust compilation

## Deployment Status

### ✓ Infrastructure Deployed
- GKE Cluster: Running (1.33.5-gke.1308000)
- VPC Network: runner-network (10.0.0.0/17)
- Node Pool: runner-pool with autoscaling enabled
- Service Account: Configured with monitoring/logging permissions
- Credentials: Stored in GCP Secrets Manager + GitHub repository secrets

### ⏳ Kubernetes Resources (Ready to deploy)
Namespaces, Helm charts, and GitHub Actions runner pods are configured in Terraform and ready for deployment once fresh authentication is obtained.

## Prerequisites

```bash
# Install tools
task terraform:install

# Authenticate to GCP
gcloud auth login
gcloud config set project kreuzberg-481219
```

## Credentials Setup

GitHub App credentials are stored securely:

**GCP Secrets Manager** (kreuzberg-481219):
- gh-app-id
- gh-app-installation-id
- gh-app-private-key

**GitHub Repository Secrets** (kreuzberg-dev/kreuzberg):
- GH_APP_ID
- GH_APP_INSTALLATION_ID
- GH_APP_PRIVATE_KEY

## Deployment

```bash
cd infrastructure

# Deploy/update infrastructure (GCP resources + Kubernetes)
export TF_VAR_gh_app_id=$(gcloud secrets versions access latest --secret=gh-app-id --project=kreuzberg-481219)
export TF_VAR_gh_app_installation_id=$(gcloud secrets versions access latest --secret=gh-app-installation-id --project=kreuzberg-481219)
export TF_VAR_gh_app_private_key=$(gcloud secrets versions access latest --secret=gh-app-private-key --project=kreuzberg-481219)

tofu apply -auto-approve
```

## Verification

```bash
# Validate configuration
task terraform:validate

# Lint with tflint
task terraform:lint

# Format files
task terraform:fmt

# Check cluster status
kubectl cluster-info
kubectl get nodes

# Check runner namespaces
kubectl get namespaces | grep arc
kubectl get pods -n arc-systems
kubectl get pods -n arc-runners

# List GitHub Actions runners in repository
gh repo view kreuzberg-dev/kreuzberg --json description
```

## Architecture

- **Module**: terraform-google-modules/github-actions-runners/google v5.1.0
- **Submodule**: gh-runner-gke
- **Authentication**: GitHub App (not PAT tokens)
- **Networking**: Private VPC with secondary IP ranges for pods/services
- **Security**: Workload Identity enabled, Shielded nodes
