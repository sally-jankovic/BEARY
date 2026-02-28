# RunAI GPUaaS Airgapping — Notes

<!-- Notes for the RunAI GPUaaS Airgapping topic. Organized by question, not source. -->
<!-- See .windsurf/skills/internet-research/SKILL.MD for research workflow and guidelines. -->
<!-- Citations are stored in whitepaper/runai-gpuaas-airgapping-references.md -->

## General Understanding

### Q1: What is RunAI and how does it enable GPU-as-a-Service (GPUaaS) for enterprise ML workloads?

**Core Platform Overview:**
Run:ai is a Kubernetes-native AI workload orchestration platform designed to optimize GPU utilization and streamline deployment of ML/DL workloads [1]. It provides dynamic resource management, intelligent scheduling, and seamless scalability across hybrid and multicloud environments [1]. NVIDIA acquired Run:ai to help customers make more efficient use of AI computing resources [2].

**Architecture Components:**
- **Run:ai Cluster**: Provides scheduling and workload management, extending Kubernetes native capabilities. Installed as a Kubernetes Operator in its own namespace (`runai`) [1][3].
- **Run:ai Control Plane**: Provides resource management, workload submission, and cluster monitoring. Can be deployed as SaaS or installed locally (self-hosted) [1][3].

**Key Capabilities:**
- **Dynamic GPU Allocation**: Enables workloads to specify and consume GPU memory and compute resources using Kubernetes Request/Limit notations [1].
- **GPU Fractionalization**: Allows multiple workloads to share GPUs efficiently through "logical GPUs" with isolated virtual memory spaces [4]. Users can request fractional GPU memory (percentage or exact size), and Run:ai dynamically partitions and enforces allocations [4].
- **Multi-GPU Fractions**: Workloads can request consistent fractions across multiple GPUs (e.g., 8×40GB out of 80GB on H100s) [4].
- **Fair-share Scheduling**: Manages resource allocation among teams/projects based on business priorities [1].
- **Centralized Cluster Management**: Manage all clusters from single platform across on-premises, cloud, and hybrid environments [3].

**KAI Scheduler (Open Source):**
NVIDIA open-sourced the Run:ai scheduler as "KAI Scheduler" under Apache 2.0 license [5]. It addresses:
- Managing fluctuating GPU demands
- Reduced wait times for compute access
- Resource guarantees for GPU allocation
- Seamless connection of AI tools and frameworks [5]

**Value Proposition:**
- Maximizes GPU utilization through AI-specific scheduler and fractionalization [1]
- Handles diverse workloads (notebooks, distributed training, inference) [1]
- Simplifies infrastructure management with centralized control plane [1]
- Enables fair and governed resource access via quotas [1]
- Supports hybrid and multicloud environments [1]

---

### Q2: What are the core challenges and requirements for deploying GPU orchestration platforms in airgapped (disconnected) environments?

**Definition and Context:**
An air-gapped system is physically or logically isolated from unsecured networks such as the public internet [6]. Common in defense/intelligence, healthcare, industrial control systems, finance, and R&D environments [6]. Security and compliance take precedence over agility [6].

**Core Challenges:**

1. **Transfer Process** [7]:
   - How data enters the air-gapped system is a key architectural consideration
   - Options range from burning disks (2-3 day manual process) to FTP servers
   - Speed of transfer affects entire operation lifecycle
   - Must accommodate gigabytes of images and regular Kubernetes updates

2. **Image Repository** [7]:
   - Core component from which Kubernetes pulls containers
   - Enables backup, disaster recovery, and security scanning
   - Technologies: Harbor, Nexus, Artifactory
   - Should be designed for highest availability

3. **Helm and Code Repository** [7]:
   - Helm charts stored in Git enable rebuilding containers
   - Code repository enables development within air-gapped system
   - Accelerates DevOps without external dependencies

4. **Network Boundaries and Ingress** [7]:
   - Must define how data enters/exits the system
   - Requires firewall configuration and blacklisting
   - Need to mimic load balancing of cloud-native environments

5. **Data Processing and GPU Management** [7]:
   - Processing power must accommodate workload without auto-scaling
   - Must properly load balance CPU/GPU utilization
   - For AI applications, infrastructure must handle large models

6. **Documentation Dilemma** [7]:
   - Most Kubernetes documentation is online
   - Must download/print documentation before deployment

**AI-Specific Challenges:**

1. **Model Packaging for Offline Environments** [6]:
   - Must download and package every component before deployment
   - Use `requirements.txt` and `environment.yml` for reproducible environments
   - Watch for telemetry calls or external license checks

2. **Infrastructure Inside Air Gap** [6]:
   - **Compute**: GPUs (A100, RTX), CPU fallback, cooling/power planning
   - **Storage**: High-speed SSDs/NVMe, object storage (MinIO)
   - **Networking**: Internal segmentation via VLANs/VRFs, isolation zones

3. **Containerization Without Internet** [6]:
   - Pre-build all Docker images in connected environment
   - Use `docker save/load` or private registries (Harbor)
   - Disable external telemetry and license pings
   - Consider lightweight K8s distros (k3s, MicroK8s)

4. **Monitoring and Observability** [6]:
   - Deploy Prometheus, Grafana, ELK stack locally
   - AI-specific: Evidently AI, WhyLabs for drift detection

**NVIDIA GPU Operator in Disconnected Environments:**
NVIDIA provides documentation for deploying GPU Operators in disconnected/airgapped OpenShift environments [8]. Requires mirroring images to local registry.

---

### Q3: What are the current industry standards and best practices for GPUaaS governance and automation in enterprise environments?

**GPUaaS Definition:**
GPU as a Service (GPUaaS) is a cloud computing model where GPU capabilities are accessed over the internet instead of buying physical hardware [9]. Converts capital expenses to operational expenses.

**Why Organizations Choose GPUaaS** [9]:
- **Capital efficiency**: Convert upfront hardware costs to consumption-based spending
- **Dynamic scalability**: Scale GPU resources in real-time to match demand
- **Technology currency**: Access newer GPU generations without upgrade complexity
- **Faster time to market**: Launch without procurement delays
- **Global reach**: Deploy across distributed locations

**GPU Cluster Best Practices** [10]:

1. **Define Workload Requirements**:
   - Estimate GPU hours per task, memory needs, interconnect bandwidth
   - Determines baseline cluster capacity

2. **Hardware Topology**:
   - Node composition: 2-8 GPUs, multi-core CPU, 256GB+ RAM, RDMA-capable NIC, NVMe SSDs
   - Head node for management, worker nodes via InfiniBand/100-400Gbps Ethernet
   - Storage nodes with distributed storage (Ceph, Lustre, BeeGFS)

3. **Networking**:
   - Intra-node: NVLink or PCIe Gen5
   - Inter-node: InfiniBand HDR/NDR (200-400 Gbps) or RoCE v2
   - Non-blocking topologies (fat-tree, dragonfly)

4. **Storage**:
   - Local NVMe SSDs for caching
   - Shared file systems (Lustre, CephFS, BeeGFS)
   - GPUDirect Storage (GDS) for direct GPU-storage transfer

5. **Orchestration and Job Management** [10]:
   - **Slurm**: Most widely used HPC workload manager for batch jobs
   - **Kubernetes**: Cloud-native, containerized AI pipelines with NVIDIA GPU Operator
   - **Ray**: Optimized for distributed AI, GenAI pipelines, LLM training
   - Hybrid: Slurm + Kubernetes for HPC scheduling + container orchestration

6. **Monitoring** [10]:
   - Prometheus + Grafana for utilization dashboards
   - NVIDIA DCGM for GPU thermals, power, health
   - Evidently AI/WhyLabs for model drift

7. **Security and Governance** [10]:
   - Role-based access control (RBAC)
   - Encrypt data at rest and in transit (TLS, IPsec)
   - Regular patching of CUDA, drivers, OS
   - Automate with Ansible, Terraform, or Helm

**MLOps GPU Resource Management** [11]:
- NVIDIA GPU Operator streamlines GPU provisioning, manages runtimes, device plugin discovery
- Multi-Instance GPU (MIG) allows division of single A100 into multiple instances
- Kubernetes offers out-of-box MIG support for tailored GPU instance allocations

---

### Q4: How do organizations currently handle self-hosted LLM model loading and lifecycle management in on-premises GPU environments?

**Why Self-Host LLMs** [12]:
1. **Security, Privacy, Compliance**: Data stays within infrastructure; avoids training data exposure to third parties
2. **Customization**: Scale alongside use case; avoid rate limits; train and customize models
3. **Avoid Vendor Lock-in**: Flexibility to migrate between solutions

**On-Prem LLM Deployment Patterns** [13]:
- Teams own full stack: GPUs, networking, scaling, monitoring
- Common in private data centers or air-gapped environments
- Typically uses open-source models

**Why Teams Choose On-Prem** [13]:
- **Data security and compliance**: All inference runs inside infrastructure
- **Predictable costs**: After initial hardware investment, lower ongoing costs
- **Performance control**: Tune for latency, throughput, scaling goals
- **Fewer external dependencies**: Enforce custom security measures

**Challenges of On-Prem LLM Deployments** [13]:
- **High upfront cost**: GPUs, networking, storage require large capital investment
- **Operational complexity**: Responsible for autoscaling, upgrades, monitoring, GPU procurement
- **Slower iteration**: Hard to keep up with frontier models and frameworks
- **GPU availability**: May not be available during usage peaks
- **Talent requirements**: Need specialized DevOps, InferenceOps, MLOps skills

**LLMOps Lifecycle** [14]:
1. **Data collection and preparation**: Source, clean, annotate data; comply with privacy laws
2. **Model training or fine-tuning**: Choose architecture or pretrained model; tune hyperparameters
3. **Model testing and validation**: Evaluate on unseen data; bias and security assessment
4. **Deployment**: Prepare hardware/software environments; set up monitoring; create APIs
5. **Optimization and maintenance**: Monitor for model drift; use quantization/pruning; version control

**Key LLMOps Benefits** [14]:
- Flexibility and scalability
- Automation via CI/CD pipelines
- Collaboration through standardized tools
- Continuous performance improvement
- Regular security and ethics reviews

**Self-Hosted LLM Solutions** [12]:
- **OpenLLM with Yatai**: Open-source LLM serving
- **Ray Serve**: Distributed ML framework for scaling
- **Hugging Face TGI**: Text Generation Inference
- **vLLM**: High-throughput inference with PagedAttention
- **Ollama**: Simple local LLM deployment

**Run:ai Self-Hosted Installation for Air-Gapped Environments** [15][16]:
- Requires `runai-air-gapped-<VERSION>.tar.gz` package from Run:ai support
- Must have private Docker registry within organization
- Extract package, upload images to local registry using `setup.sh`
- Creates `custom-env.yaml` for control-plane installation
- Requires local Certificate Authority configuration
- Run:ai Professional Services provides installer with `--air-gapped` flag [16]

---

### Summary

**Key Themes:**

1. **Run:ai as GPUaaS Enabler**: Run:ai provides enterprise-grade GPU orchestration through Kubernetes-native architecture, enabling GPU fractionalization, fair-share scheduling, and centralized management. The platform can be deployed self-hosted for air-gapped environments.

2. **Air-Gapped Deployment Complexity**: Deploying GPU orchestration in disconnected environments requires careful planning around image repositories, transfer processes, network boundaries, and offline documentation. AI workloads add complexity with model packaging, dependency management, and monitoring without internet access.

3. **Industry Standards Converging**: Best practices center on Kubernetes-based orchestration (Run:ai, GPU Operator), combined with HPC schedulers (Slurm) for hybrid workloads. Governance requires RBAC, encryption, and automated configuration management.

4. **LLMOps as Emerging Discipline**: Self-hosted LLM deployment follows a lifecycle similar to MLOps but with unique challenges around model size, inference optimization, and continuous retraining. Organizations choose on-prem for security, compliance, and cost predictability.

5. **Gaps Identified**:
   - Limited automation tooling specifically for air-gapped LLM model loading
   - Manual processes for transferring large model weights into disconnected environments
   - Lack of standardized "on-demand" LLM serving patterns for air-gapped GPUaaS
   - Documentation and tooling fragmentation across Run:ai, NVIDIA GPU Operator, and LLM serving frameworks

---

## Deeper Dive

*Detailed notes for each subtopic are in separate files. This section provides cross-cutting synthesis.*

### Subtopic 1: Run:ai Air-Gapped Deployment Workflows
See: `runai-airgapped-deployment-notes.md`

**Key Findings:**
- Run:ai provides official air-gapped packages (`runai-air-gapped-<VERSION>.tar.gz`) with setup scripts
- Professional Services installer offers `--air-gapped` flag for comprehensive automation
- Requires private Docker registry, local CA, and careful network configuration
- Tools like Hauler and Skopeo simplify container image mirroring
- CMU SEI research shows registry mirror configuration enables transparent image sourcing [19]

### Subtopic 2: LLM Model Loading and Serving in Air-Gapped GPUaaS
See: `llm-serving-airgapped-notes.md`

**Key Findings:**
- NVIDIA NIM provides two air-gap methods: create-model-store and download-to-cache [20]
- vLLM on Kubernetes offers best balance of performance and flexibility [22]
- Cold-start optimization requires persistent model caches and pre-loading strategies
- Run:ai + Knative enables serverless-style auto-scaling in air-gapped environments [16]
- Gap: No unified "model registry" pattern for air-gapped LLM weight management

### Subtopic 3: Governance and Automation for Air-Gapped GPUaaS
See: `governance-automation-notes.md`

**Key Findings:**
- Run:ai RBAC provides fine-grained control over users, projects, departments, quotas [24][25]
- AI Initiatives framework maps organizational structure to resource allocation
- Flux CD well-suited for air-gapped GitOps (local Git mirror, no public endpoint) [26]
- Terraform + Ansible combination reduces GPU cluster deployment time by 85% [27]
- Gap: No unified tooling combining GPU scheduling, model management, and GitOps

---

### Summary

**Synthesis Across Subtopics:**

The research reveals a maturing but fragmented ecosystem for air-gapped GPUaaS with Run:ai. Three key themes emerge:

1. **Run:ai Air-Gap Readiness**: Run:ai has invested significantly in air-gapped deployment support, with official packages, installer automation, and comprehensive documentation. The platform's Kubernetes-native architecture aligns well with air-gapped patterns (private registries, local Helm charts). However, the process still requires significant manual preparation and expertise.

2. **LLM Serving Gap**: While container orchestration for air-gapped environments is well-established, LLM model weight management lags behind. NVIDIA NIM and Ollama provide model export/import capabilities, but there's no equivalent to a "container registry" for model weights. Organizations must manage model artifacts separately from container images, creating operational complexity for "on-demand" LLM serving.

3. **Governance Maturity**: Run:ai's RBAC and AI Initiatives frameworks provide enterprise-grade governance for GPU resources. Combined with GitOps tools like Flux CD and IaC tools like Terraform/Ansible, organizations can achieve reproducible, auditable infrastructure. The challenge is integrating these tools into a cohesive workflow for air-gapped environments.

**Identified Gaps for User's Purpose:**

Given the user's goal of establishing governance and automation for "on-demand" LLM serving in air-gapped Run:ai:

1. **Model Lifecycle Automation**: No standard tooling for automating LLM model transfer, versioning, and deployment in air-gapped environments
2. **On-Demand Cold Start**: Achieving true "on-demand" requires solving model loading latency; current solutions require always-on deployments or accept cold-start delays
3. **Unified Workflow**: Organizations must integrate Run:ai, model serving (vLLM/TGI), GitOps, and IaC tools manually
4. **Documentation Fragmentation**: Best practices are scattered across Run:ai docs, NVIDIA docs, and community resources

