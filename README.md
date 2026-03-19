# GitHub Self‑Hosted Runner on Kubernetes

This project demonstrates how to deploy a **GitHub Actions self‑hosted runner** inside a Kubernetes cluster.  
The runner is configured to work with an **organization or repository** and uses Kubernetes primitives for lifecycle management.

---

## Prerequisites
- A Kubernetes cluster (minikube, kind, or cloud provider).
- `kubectl` configured to access your cluster.
- Admin access to your GitHub organization or repository.
- A GitHub Personal Access Token (PAT) with required scopes:
  - `repo`
  - `admin:repo_hook`
  - `workflow`

---

##  Steps

### 1. Create a Namespace
```bash
kubectl create namespace demo
```
---
### 2. Generate a PAT with required scopes
  - `repo`
  - `admin:repo_hook`
  - `workflow`
---

### 3. Create a Kubernetes Secret
Replace the organization name and PAT in git_secret.yaml
Store GitHub configuration and token securely:

```bash
apiVersion: v1
kind: Secret
metadata:
    name: github-secrets
type: Opaque
stringData:
  GITHUB_ORGANIZATION: "org-name"
  GITHUB_TOKEN: "git-token"
  GITHUB_URL: "https://api.github.com"
```
```bash
kubectl apply -f git_secret.yaml -n demo
```
---

### 4. Deploy the Runner
Apply the provided Deployment YAML:

```bash
kubectl apply -f github-runner-deployment.yaml -n demo
```

This will:
- Run an init container to fetch the runner token.
- Start the runner container with the correct configuration.
- Register the runner with GitHub.
- Handle cleanup on pod termination.

Check the logs of pod for succuessful authentication 

<img width="1355" height="239" alt="image" src="https://github.com/user-attachments/assets/c515d05a-4d9b-4e47-8901-e4119a488d43" />

<img width="1318" height="558" alt="image" src="https://github.com/user-attachments/assets/ef0aeb88-8fc7-4afd-ab23-6ff49659b611" />


---

### 5. Verify Runner Registration
- Go to your GitHub repository or organization.
- Navigate to **Settings → Actions → Runners**.
- You should see your Kubernetes runner listed as `self-hosted`.

---

### 6. Run a Sample Workflow
Create a `.github/workflows/test-runner.yml` in your repo:

```yaml
name: Test Runner

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Hello World
        run: echo "Hello from self-hosted runner!"
```

Push to `main` and check the Actions tab — the workflow should execute on your Kubernetes runner.

---

## Best Practices
- Use **labels** to differentiate runners (e.g., `self-hosted`, `linux`, `k8s`).
- Scale replicas based on workload demand.
- Rotate tokens regularly for security.
- Use resource requests/limits to avoid cluster contention.

---

## Benefits
- Fully automated GitHub Actions runner lifecycle in Kubernetes.
- Secure token management via Kubernetes Secrets.
- Scalable and reproducible CI/CD infrastructure.
---
