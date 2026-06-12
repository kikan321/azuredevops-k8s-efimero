# Ephemeral Kubernetes CI/CD & GitOps Pipeline via Azure DevOps

This project implements an enterprise-grade Continuous Integration (CI) pipeline for the automated validation and deployment of Kubernetes applications. By leveraging an **ephemeral infrastructure** approach, this workflow provisions test clusters, validates manifests, collects runtime metrics, and tears down the entire environment with a **zero-cost strategy (\$0 USD)**.

## 🚀 Architecture & Philosophy

The primary objective of this project is to implement modern **Shift-Left Testing** and **GitOps** paradigms. Every time a developer pushes a change to the Kubernetes manifests on the `main` branch, an automated auditing and deployment workflow is instantly triggered within an isolated runner.

```text
[ Git Push to Main ] 
         │
         ▼
 ┌────────────────────────────────────────────────────────┐
 │ Azure DevOps Pipeline (Ephemeral Agent)                │
 │                                                        │
 │ 1. Static Analysis ──► 2. Provisioning ──► 3. Deploy   │
 │   (kube-linter)          (KinD Cluster)     (kubectl)  │
 └────────────────────────────────────────────────────────┘
         │
         ▼
[ Collect Logs & Runtime Artifacts ] ──► [ Automated Environment Teardown ]
```

## ✨ Core Technical Features

*   **Ephemeral Infrastructure:** Uses **KinD (Kubernetes in Docker)** inside the cloud-hosted Azure DevOps agent. This eliminates the need for expensive public cloud resources (like AWS EKS or Azure AKS) during testing phases.
*   **Static Analysis (Quality Gate):** Integrates `kube-linter` to check manifests early against CNCF security best practices and configuration standards.
*   **Runtime Validation:** The pipeline actively monitors application health (`rollout status`) and container availability (`condition=available`) before marking the execution as successful.
*   **Zero-Cost Strategy (\$0 USD):** Tailored entirely around Azure DevOps free-tier execution minutes and internal runner storage boundaries.

## 📁 Repository Structure

```text
├── k8s-manifests/
│   └── deployment.yaml      # Standardized and sanitized Kubernetes Deployment manifest
├── azure-pipelines.yml      # Declarative CI/CD pipeline definition for Azure DevOps
└── README.md                # Project documentation and architectural breakdown
```

## 🛠️ Tech Stack

*   **Orchestration:** Kubernetes v1.28+
*   **Local/CI Environment:** KinD (Kubernetes in Docker)
*   **CI/CD Orchestrator:** Azure Pipelines (Azure DevOps)
*   **Static Code Analysis:** kube-linter (StackRox/RedHat)
*   **Control CLI:** kubectl

---

## ⚙️ Detailed Pipeline Stages & Step Breakdown

The declarative pipeline defined in `azure-pipelines.yml` executes sequentially through the following engineered phases:

### Phase 1: Environment Initialization & Setup
*   **Step 0: Repository Checkout (`checkout: self`):** Explicitly pulls the source code into the fresh Ubuntu runner workspace to ensure a clean slate and reproducibility.
*   **Step 1: Install KinD:** Downloads and configures the latest binary of Kubernetes in Docker, enabling full-featured orchestration cluster management directly inside an unprivileged Docker host container.
*   **Step 2: Install kubectl:** Provisions the Kubernetes control CLI to allow the pipeline agent to map endpoints, authenticate configurations, and issue cluster commands.

### Phase 2: Shift-Left Security & Static Analysis
*   **Step 3: Run kube-linter:** Acts as an architectural **Quality Gate**. It analyzes the raw YAML code of `deployment.yaml` checking for critical flaws (e.g., missing CPU/Memory boundaries, unprivileged containers, or incorrect labels). If security or structural checks fail, the pipeline terminates immediately, blocking problematic configurations from advancing.

### Phase 3: Cluster Provisioning & Health Checks
*   **Step 4: Create KinD Cluster:** Automatically provisions a local Kubernetes cluster named `cluster-cv` inside Docker. This setup takes less than 40 seconds to map control-plane components.
*   **Step 5: Verify Cluster Health:** Runs `kubectl cluster-info` and `kubectl get nodes` to establish a verified, responsive control plane baseline before any workloads are applied.

### Phase 4: Workload Deployment & Runtime Verification
*   **Step 6: Deploy Application:** Issues `kubectl apply` against the standardized manifest file inside the `k8s-manifests` path.
*   **Step 7: Verify Rollout Status:** Runs an explicit blocking call to monitor deployment rollout chunks. If the container image fails to pull (e.g., `ImagePullBackOff`), this step halts execution to signal a build failure.
*   **Step 8: Verify Deployment Availability:** Evaluates if the minimum required pod replication status matches the specified metrics condition within a 120-second timeout.

### Phase 5: Observability, Artifact Storage, and Teardown
*   **Step 9: Collect Cluster State:** Gathers configuration outputs (`get deployments`, `get pods -o wide`, `get svc`, and cluster system events) into a consolidated runtime snapshot log.
*   **Step 10: Publish Pipeline Artifacts:** Uses the native platform tasks to push all telemetry log files into Azure DevOps secure storage. This guarantees full debugging transparency for developers even after the underlying ephemeral runner VM is permanently destroyed.

---
Designed and developed as a portfolio showcase to demonstrate advanced workflow automation, FinOps cost optimization, and rigorous continuous delivery standards.
