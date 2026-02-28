# State-of-the-Art Workflows for GPUaaS Using Run:ai in Air-Gapped Environments

**Author:** Background Research Agent (BEARY)  
**Date:** 2026-02-27

---

## Abstract

This whitepaper examines best practices for deploying GPU-as-a-Service (GPUaaS) infrastructure using NVIDIA Run:ai in heavily air-gapped environments. We analyze current industry standards for GPU orchestration, the specific challenges of disconnected deployments, and emerging patterns for self-hosted LLM model lifecycle management. The research identifies significant gaps in tooling for "on-demand" LLM serving in air-gapped contexts, while documenting practical workflows for governance, automation, and model loading. Key findings include Run:ai's mature air-gap support through official packages and installer automation, the lack of a unified "model registry" pattern for LLM weights, and the critical role of GitOps and Infrastructure-as-Code in achieving reproducible air-gapped deployments.

---

## Introduction

Organizations operating in highly regulated industries—defense, healthcare, finance, and government—increasingly require AI capabilities while maintaining strict network isolation. Air-gapped environments, physically or logically disconnected from the public internet, present unique challenges for deploying modern GPU orchestration platforms and large language models (LLMs) [6][7].

This whitepaper addresses three interconnected questions:

1. **How can Run:ai be deployed effectively in air-gapped Kubernetes environments?**
2. **What are the best practices for loading and serving LLMs with minimal manual intervention?**
3. **What governance and automation frameworks enable sustainable GPUaaS operations without internet connectivity?**

The research is motivated by a practical need: transforming heavily manual, sub-optimal approaches to loading self-hosted models into self-hosted Run:ai into a governed, automated, reusable workflow that enables "on-demand" (or near on-demand) LLM experimentation and iteration.

---

## Background

### Run:ai and GPU-as-a-Service

Run:ai is a Kubernetes-native AI workload orchestration platform designed to optimize GPU utilization and streamline ML/DL workload deployment [1]. Acquired by NVIDIA in 2024, Run:ai provides dynamic resource management, intelligent scheduling, and seamless scalability across hybrid and multicloud environments [2].

The platform consists of two primary components [1][3]:
- **Run:ai Cluster**: Provides scheduling and workload management, installed as a Kubernetes Operator
- **Run:ai Control Plane**: Provides resource management, workload submission, and cluster monitoring; can be deployed as SaaS or self-hosted

Key capabilities include GPU fractionalization (allowing multiple workloads to share GPUs through isolated "logical GPUs"), fair-share scheduling, and centralized multi-cluster management [4]. NVIDIA has open-sourced the core scheduler as "KAI Scheduler" under Apache 2.0 license [5].

### Air-Gapped Environments

An air-gapped system is physically or logically isolated from unsecured networks such as the public internet [6]. Common in defense/intelligence, healthcare, industrial control systems, finance, and R&D environments, these systems prioritize security and compliance over agility [6].

Air-gapped Kubernetes deployments face specific challenges [7]:
- **Transfer Process**: How data enters the system (disk burning, FTP, secure media)
- **Image Repository**: Core component for container image storage and distribution
- **Network Boundaries**: Firewall configuration and ingress management
- **Documentation**: Most Kubernetes documentation requires internet access

For AI workloads, additional challenges include model packaging, dependency management, and monitoring without internet connectivity [6].

### LLMOps and Self-Hosted Model Management

Large Language Model Operations (LLMOps) is a methodology for managing, deploying, monitoring, and maintaining LLMs in production environments [14]. The lifecycle includes data collection, model training/fine-tuning, testing, deployment, and continuous optimization.

Organizations choose self-hosted LLM deployments for security, compliance, customization, and avoiding vendor lock-in [12][13]. However, on-premises deployments face challenges including high upfront costs, operational complexity, slower iteration cycles, and specialized talent requirements [13].

---

## Run:ai Air-Gapped Deployment Workflows

### Official Air-Gapped Installation

Run:ai provides comprehensive air-gapped deployment support through official packages and automation tools [15][16].

**Installation Process**:

1. **Obtain Package**: Request `runai-air-gapped-<VERSION>.tar.gz` from Run:ai customer support
2. **Prerequisites**: Kubernetes cluster with kubectl access, Docker installed, private Docker registry, minimum 20GB free disk space
3. **Extract and Upload**:
   ```bash
   tar xvf runai-airgapped-package-<VERSION>.tar.gz
   kubectl create namespace runai-backend
   export REGISTRY_URL=<Docker Registry address>
   sudo -E ./setup.sh
   ```
4. **Configure Local CA**: Prepare public key of local certificate authority [15]
5. **Install Control Plane**: Use Helm with generated `custom-env.yaml` [17]

The Run:ai Professional Services installer provides additional automation with an `--air-gapped` flag, handling certificate management, DNS configuration, Nginx Ingress, and optional components like Prometheus, GPU Operator, and Knative [16].

### Container Image Mirroring

Transferring container images into air-gapped environments has been significantly simplified by modern tooling [19].

**Recommended Tools**:

- **Hauler**: CMU Software Engineering Institute recommends this tool for drastically simplified air-gapped registry mirroring. It enables transparent image pulling where developers don't need different manifests for air-gapped vs. connected environments [19].

- **Skopeo**: Command-line tool for image operations including mirroring, saving to tar files, and loading from tar files [18].

- **Harbor**: Enterprise container registry with security scanning, replication policies, and RBAC [18].

**Best Practice**: Configure Kubernetes registry mirror options for transparent image sourcing. This eliminates the need to maintain separate Helm values files for air-gapped environments [19].

---

## LLM Model Loading and Serving

### Model Weight Transfer

Unlike container images, LLM model weights lack a standardized registry pattern for air-gapped environments. Current solutions require manual preparation on connected systems.

**NVIDIA NIM Air-Gap Methods** [20]:

1. **Create Model Store**: Use `create-model-store` command to create a local model repository that can be transferred to air-gapped environments
2. **Download to Cache**: Use `download-to-cache` to download model profiles, then transfer the cache directory

Both methods enable fully offline operation without NGC registry or Hugging Face Hub access.

**Ollama Export/Import** [21]:
```bash
# Connected system
ollama pull qwen2.5-coder:7b
ollama export qwen2.5-coder:7b > model.tar

# Air-gapped system (after secure transfer)
ollama import model.tar
```

### On-Demand LLM Serving

Achieving "on-demand" LLM serving in air-gapped environments requires addressing cold-start latency and resource efficiency.

**vLLM on Kubernetes** [22]:

vLLM provides scalable LLM serving with PagedAttention for optimal memory efficiency. Deployment options include native Kubernetes manifests, Helm charts, KServe, and KubeRay. Stripe reported 73% cost reduction after migrating to vLLM [22].

**Cold-Start Optimization Strategies**:

1. **Persistent Model Cache**: Use PersistentVolumeClaims to store model weights, avoiding re-download on pod restart
2. **Model Pre-loading**: Keep models loaded via always-on deployments or warm standby replicas
3. **Quantization**: Use INT8/INT4 quantized models for faster loading and reduced memory footprint
4. **High-Speed Storage**: Store models on NVMe with GPUDirect Storage for direct GPU-storage transfer

**Run:ai + Knative Integration**: The Run:ai installer supports optional Knative installation (`--knative` flag), enabling auto-scaling inference workloads and scale-to-zero for cost optimization [16].

### Gap Analysis: Model Registry Pattern

A significant gap exists in the ecosystem: **there is no equivalent to a container registry for LLM model weights in air-gapped environments**. Organizations must manage model artifacts separately from container images, creating operational complexity. A unified "model registry" that supports versioning, access control, and efficient transfer would significantly improve on-demand LLM serving workflows.

---

## Governance and Automation

### Run:ai RBAC and Resource Management

Run:ai provides enterprise-grade governance through its RBAC system [24]:

**Components**:
- **Subjects**: Users, Applications, Groups (SSO)
- **Roles**: System Admin, Research Manager, Department Admin, Editor, Researcher
- **Scope**: Projects, Departments, Clusters, Tenant
- **Assets**: Departments, Projects, Inference, Workspaces, Training

**AI Initiatives Framework** [25]:

Run:ai's AI Initiatives framework enables organizations to map their structure to resource allocation:
- Map cluster resources to **node pools**
- Assign quota allocation per node pool to Projects/Departments
- Set **over-quota** access rights for unused resources
- Prevent over-subscription while maximizing utilization

**Key Principle**: Sum of project quotas should not exceed department quota; sum of department quotas should not exceed cluster resources. Over-quota access allows utilization of idle resources while maintaining fairness.

### Infrastructure as Code

GPU cluster complexity demands Infrastructure-as-Code approaches [27].

**Terraform for GPU Infrastructure**:
- Manages infrastructure lifecycle with declarative configurations
- State management enables team collaboration and prevents drift
- Dependency resolution ensures correct provisioning order
- Modular architecture enables reusable GPU-specific configurations

**Ansible for GPU Software Stack**:
- Idempotent operations correct configuration drift
- Dynamic inventory discovers GPU nodes from cloud APIs or Kubernetes
- Parallel execution configures hundreds of nodes simultaneously
- Handles driver installation, performance tuning, RDMA configuration

**Integration Pattern**: Terraform provisions infrastructure → outputs inventory for Ansible → Ansible configures OS, drivers, libraries → validates cluster readiness. This combination reduces GPU cluster deployment time by 85% while eliminating configuration errors [27].

### GitOps for Air-Gapped Environments

GitOps principles apply well to air-gapped GPU orchestration, with some adaptations [26].

**Flux CD** is particularly well-suited for air-gapped environments:
- Bootstraps from local Git mirror
- Reconciles without public endpoint
- Service-account impersonation fits least-privilege policies
- Every change results in signed commit for audit trail

**Air-Gapped GitOps Requirements**:
- Mirror Git repositories internally
- Store Terraform providers and modules locally
- Cache Ansible collections and roles
- Use local artifact repositories (Nexus, Artifactory)

---

## Discussion

### Maturity Assessment

The ecosystem for air-gapped GPUaaS with Run:ai is **maturing but fragmented**:

| Component | Maturity | Notes |
|-----------|----------|-------|
| Run:ai Air-Gap Support | High | Official packages, installer automation, comprehensive docs |
| Container Image Mirroring | High | Hauler, Skopeo, Harbor provide robust solutions |
| LLM Model Management | Low | No unified registry pattern; manual processes required |
| Governance (RBAC, Quotas) | High | Run:ai provides enterprise-grade controls |
| GitOps/IaC | Medium | Tools exist but require manual integration |

### Achieving "On-Demand" LLM Serving

True "on-demand" LLM serving in air-gapped environments faces a fundamental tension:

- **Cold-start latency**: Loading large models (7B+ parameters) takes minutes, not seconds
- **Resource efficiency**: Always-on deployments waste GPU resources during idle periods

**Practical Approaches**:

1. **Tiered Model Availability**: Keep frequently-used models always-on; less-used models available with cold-start delay
2. **Model Pooling**: Share GPU resources across multiple models using Run:ai fractionalization
3. **Predictive Scaling**: Use workload patterns to pre-warm models before expected demand
4. **Quantization Trade-offs**: Accept some quality degradation for faster loading via INT4 quantization

### Recommended Workflow Architecture

Based on the research, a state-of-the-art workflow for air-gapped GPUaaS with Run:ai should include:

1. **Infrastructure Layer**: Terraform-managed Kubernetes clusters with Ansible-configured GPU nodes
2. **Orchestration Layer**: Run:ai with air-gapped installation, RBAC, and quota management
3. **Registry Layer**: Harbor for containers; MinIO or similar for model weights (manual pattern)
4. **Serving Layer**: vLLM or TGI on Kubernetes with persistent model caches
5. **GitOps Layer**: Flux CD with local Git mirror for declarative configuration
6. **Transfer Layer**: Hauler/Skopeo for images; NVIDIA NIM or custom scripts for models

---

## Conclusion

This research reveals that deploying GPUaaS with Run:ai in air-gapped environments is achievable with current tooling, but requires significant integration effort. Run:ai has invested substantially in air-gap support, and the Kubernetes ecosystem provides mature solutions for container image mirroring.

**Key Takeaways**:

1. **Run:ai is air-gap ready**: Official packages and the Professional Services installer provide comprehensive automation for disconnected deployments.

2. **LLM model management is the primary gap**: Unlike containers, model weights lack a standardized registry pattern. Organizations must develop custom workflows for model transfer, versioning, and deployment.

3. **True "on-demand" requires trade-offs**: Cold-start latency for large models conflicts with resource efficiency. Practical solutions involve tiered availability, model pooling, and predictive scaling.

4. **Governance tooling is mature**: Run:ai's RBAC and AI Initiatives frameworks, combined with GitOps and IaC, enable enterprise-grade governance in air-gapped environments.

5. **Integration is manual**: No unified platform combines GPU scheduling, model management, and GitOps. Organizations must integrate Run:ai, vLLM/TGI, Flux CD, and Terraform/Ansible manually.

**Recommendations for the User's Purpose**:

To establish governance and automation for "on-demand" LLM serving in air-gapped Run:ai:

1. Deploy Run:ai using the Professional Services installer with `--air-gapped` flag
2. Establish a model weight transfer workflow using NVIDIA NIM's create-model-store or download-to-cache methods
3. Implement vLLM on Kubernetes with persistent model caches and Run:ai quota management
4. Use Flux CD with a local Git mirror for GitOps-based configuration management
5. Accept that "on-demand" will involve some cold-start latency; optimize with tiered availability and quantization

**Areas for Further Research**:

- Development of a standardized "model registry" pattern for air-gapped LLM deployments
- Integration of model lifecycle management into Run:ai's control plane
- Automated model weight transfer pipelines analogous to container image mirroring

---

## References

See `runai-gpuaas-airgapping-references.md` for the full bibliography with 27 sources covering Run:ai documentation, industry best practices, and technical guides for air-gapped deployments.

