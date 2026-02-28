# Governance and Automation for Air-Gapped GPUaaS — Subtopic Notes

<!-- Subtopic notes for governance and automation in air-gapped GPU environments -->
<!-- Citations are stored in whitepaper/runai-gpuaas-airgapping-references.md -->

## Q1: What governance frameworks and policies should be implemented for GPU resource allocation in air-gapped enterprise environments?

**Run:ai RBAC Components** [24]:

1. **Subjects**: Entities that receive access rules
   - Users
   - Applications
   - Groups (for SSO authentication)

2. **Roles**: Groups of permissions that can be granted
   - System Admin: Full access to all managed objects and scopes
   - Research Manager: Access to assigned projects
   - Department Admin: Manages department-level resources
   - Editor: Limited scope access
   - Researcher: Workload submission access

3. **Scope**: Organizational components with role-based access
   - Projects
   - Departments
   - Clusters
   - Tenant (all clusters)

4. **Assets**: Entities with RBAC rules
   - Departments, Projects
   - Inference, Workspaces
   - Environments
   - Quota management dashboard
   - Training workloads

**Run:ai AI Initiatives Framework** [25]:

**Organizational Mapping Options**:
- Based on individuals (personal projects)
- Based on business units (team-based)
- Based on organizational structure (hierarchical)

**Resource Management**:
- Map cluster resources to **node pools**
- Assign quota allocation per node pool to Projects/Departments
- Set **over-quota** access rights for unused resources
- Prevent over-subscription (sum of quotas exceeding physical resources)

**Key Governance Principles**:
- Sum of project quotas should NOT exceed department quota
- Sum of department quotas should NOT exceed cluster resources
- Over-quota access allows utilization of idle resources while maintaining fairness

**Enterprise Security Standards** [10]:
- Role-based access control (RBAC)
- Encrypt data at rest and in transit (TLS, IPsec)
- Regular patching of CUDA, drivers, OS packages
- Audit logging for compliance

---

## Q2: How can GitOps and Infrastructure-as-Code principles be applied to air-gapped GPU orchestration?

**GitOps Tools for Air-Gapped Environments**:

1. **Flux CD** [26]:
   - Bootstraps from local Git mirror
   - Reconciles without public endpoint
   - Service-account impersonation fits least-privilege policies
   - Every change results in signed commit for audit trail
   - Well-suited for air-gapped sites and compliance programs

2. **Argo CD** [26]:
   - Requires more configuration for air-gapped deployment
   - Strong UI for visualization
   - Supports multi-cluster management

**Infrastructure as Code for GPU Clusters** [27]:

**Terraform for GPU Infrastructure**:
- Manages infrastructure lifecycle with declarative configurations
- State management in remote backends enables team collaboration
- Dependency resolution ensures correct provisioning order
- Modular architecture for reusable GPU-specific configurations

Example GPU cluster module:
```hcl
module "gpu_cluster" {
  source = "./modules/gpu-cluster"
  gpu_nodes = {
    training = {
      instance_type = "p5.48xlarge"  # 8x H100 GPUs
      count = 16
      network_config = {
        efa_enabled = true  # RDMA
        bandwidth_gbps = 3200
      }
    }
  }
}
```

**Ansible for GPU Software Stack** [27]:
- Idempotent operations correct configuration drift
- Dynamic inventory discovers GPU nodes
- Parallel execution configures hundreds of nodes simultaneously

Key tasks:
- Install NVIDIA GPU drivers with specific versions
- Configure GPU performance settings (persistence mode, power limits)
- Setup InfiniBand/RDMA configuration
- Configure NCCL environment variables
- Install container runtime with NVIDIA toolkit

**Integration Pattern** [27]:
1. Terraform creates infrastructure (VPCs, compute, storage, networking)
2. Terraform outputs inventory for Ansible consumption
3. Ansible configures OS and base software
4. Ansible installs GPU drivers and libraries
5. Ansible validates cluster readiness
6. Monitoring agents deploy automatically

**Day-2 Operations**:
- Driver updates via Ansible playbooks
- Terraform scales clusters based on workload
- Configuration changes propagate through Git commits
- Rollbacks execute automatically on validation failures

**Air-Gapped GitOps Considerations**:
- Mirror Git repositories internally
- Store Terraform providers and modules locally
- Cache Ansible collections and roles
- Use local artifact repositories (Nexus, Artifactory)

---

## Summary

Governance in air-gapped GPUaaS environments centers on Run:ai's RBAC system, which provides fine-grained control over users, projects, departments, and resource quotas. The AI Initiatives framework enables organizations to map their structure to Run:ai's resource management model, ensuring fair allocation while maximizing utilization through over-quota access.

Infrastructure as Code is essential for managing GPU cluster complexity. Terraform handles infrastructure provisioning with state management and dependency resolution, while Ansible configures the GPU software stack idempotently. The combination reduces deployment time by 85% and eliminates configuration errors.

For air-gapped environments, Flux CD is particularly well-suited due to its ability to bootstrap from local Git mirrors and reconcile without external endpoints. Organizations should establish local mirrors for all Git repositories, Terraform providers, and Ansible collections to enable fully offline GitOps workflows.

The key gap is the lack of standardized tooling that combines GPU-aware scheduling (Run:ai), model management (weights, versions), and GitOps workflows into a unified air-gapped platform. Organizations must currently integrate multiple tools manually.

