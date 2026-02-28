# RunAI-in-airgapped-envs — Notes

<!-- Notes for the RunAI-in-airgapped-envs topic. Organized by question, not source. -->
<!-- See .windsurf/skills/internet-research/SKILL.MD for research workflow and guidelines. -->
<!-- Citations are stored in whitepaper/RunAI-in-airgapped-envs-references.md -->

## General Understanding

<!-- Notes from initial research questions. One subsection per question. -->

### Q1: What are NVIDIA NIM containers and how do they integrate with NGC registry in airgapped environments?

NVIDIA NIM (NVIDIA Inference Microservices) for LLMs are containerized inference services that support deployment in air-gapped environments with no internet connection and no connection to NGC registry or Hugging Face Hub [1].

**Two deployment approaches for airgapped environments:**

1. **Multi-LLM NIMs** - Use `create-model-store` command within the NIM container to create a repository for a single model. Requires HuggingFace token (HF_TOKEN) during model store creation on internet-connected machine, then transfer to airgapped environment without the token [1].

2. **LLM-specific NIMs** - Use `download-to-cache` utility to pre-download model profiles to cache on internet-connected machine, then transfer cache to airgapped system. After transfer, run NIM without NGC_API_KEY [1].

**Key workflow pattern:**
- Internet-connected phase: Download models/profiles using NGC API key or HF token
- Transfer phase: Copy cached models to airgapped environment  
- Airgapped phase: Run NIM containers pointing to local cache, no credentials needed [1]

**Kubernetes/OpenShift integration:**
The NIM Operator supports two airgapped options: accessing NGC through proxy with custom certificates, or using mirrored local model registries [2]. For OpenShift, certificate injection must be enabled using ConfigMaps with `config.openshift.io/inject-trusted-cabundle: "true"` label [2].

**Community validation:**
Users have confirmed the ability to download NIM docker images from nvcr.io while online, then deploy containers offline using local assets [3]. The official documentation's "Serving models from local assets" section provides the canonical workflow [3].

### Q2: What is the role of Nexus as an artifact repository for NGC containers in airgapped Kubernetes clusters?

Nexus Repository (specifically Sonatype OSS Nexus) serves as a critical intermediary for airgapped Kubernetes environments, providing local storage and distribution of container images and artifacts [5].

**Repository requirements for airgapped K8s:**
- **RAW repository** - Stores Go binaries and Helm packages (minimum 15GB) [5]
- **Docker repository** - Manages Docker images including NGC containers (minimum 15GB) [5]
- Alternative: Docker Registry can be used instead of Nexus for container management [5]

**NVIDIA's approach to airgapped systems:**
For DGX systems (which share similar airgap challenges), NVIDIA recommends creating private mirrors of public repositories [4]. The process involves:
1. Identifying sources from `/etc/apt/sources.list` and `/etc/apt.sources.list.d/` [4]
2. Creating and maintaining private repository mirrors [4]
3. Updating system sources to point to private mirrors instead of public repos [4]

**Key insight:** While NVIDIA documentation doesn't explicitly mention Nexus for NIM containers, the pattern is clear - airgapped environments require a local artifact repository (Nexus, Harbor, or Docker Registry) to cache and serve NGC container images that would normally be pulled from nvcr.io [4][5].

### Summary

NVIDIA NIM containers support airgapped deployment through a two-phase approach: download models/containers while online using NGC API keys or HF tokens, then transfer to airgapped environments where they run from local cache without credentials [1][3]. For Kubernetes deployments, the NIM Operator offers proxy-based access or mirrored local registries, with OpenShift requiring specific certificate injection configurations [2]. While NVIDIA documentation doesn't explicitly prescribe Nexus, the established pattern for airgapped K8s environments uses local artifact repositories (Nexus, Harbor, or Docker Registry) to cache and serve container images, requiring minimum 15GB storage for Docker images [4][5]. The critical gap is that NVIDIA's NIM documentation focuses on the model caching workflow but doesn't detail the container registry mirroring process needed for production airgapped K8s deployments.

---

## Deeper Dive

<!-- Notes from in-depth research questions. One subsection per question. -->

### Q1: What are the current workarounds and best practices for downloading and managing NIM containers from NGC in airgapped OpenShift/K8s environments?

**NGC Container Replicator - Official automation tool:**
NVIDIA provides an open-source NGC Container Replicator tool that creates a local clone of the NGC container registry [7]. Key features:
- Automatically converts NGC containers to Singularity images for HPC environments [7]
- Can run as cron job (daily/weekly) to keep local library synchronized with NGC [7]
- Uses `--no-exporter` option to generate Docker tar archives for "sneakernettin" to airgapped Docker registries [7]
- Requires NGC API key only for replicator, not for end users [7]
- Example: `docker run deepops/replicator --project=hpc --image=namd --singularity --no-exporter --api-key=<key>` [7]

**Model cache management:**
Users seeking offline deployment need to understand NIM cache directory structure [6]. The official configuration documentation provides details on cache paths and mounting local file system paths into containers [6].

**Current gap:** NGC Container Replicator documentation focuses on HPC/Singularity use cases. There's limited official guidance on using it specifically for Kubernetes/OpenShift Docker registries or integrating with Nexus/Harbor for NIM containers. The tool supports Docker tar export (`--no-exporter` disables it), suggesting this is the intended path for airgapped Docker registries, but the workflow isn't explicitly documented for NIM deployments [7].

### Q2: How can I automate the process of syncing NGC artifacts to a local registry through Nexus or similar intermediaries?

**NVIDIA's automation patterns from TAO Toolkit:**
The TAO Toolkit provides a reference implementation for airgapped NGC workflows with three phases [8]:
1. **Asset preparation** (internet-connected): Use `pretrained_models.py` script from tao-core to download models with NGC API key [8]
2. **Secure transfer**: Move prepared assets to airgapped environment [8]
3. **Deployment**: Deploy using Helm charts with environment variable `AIRGAPPED_MODE=true` and `LOCAL_MODEL_REGISTRY` pointing to local storage [8]

**Key automation approaches:**
- **NGC Container Replicator**: Run as cron job with Docker tar export enabled (remove `--no-exporter` flag) to generate archives for local Docker registry [7]
- **Environment-based configuration**: Use `AIRGAPPED_MODE=true` environment variable to signal offline operation [8]
- **Local registry path**: Configure `LOCAL_MODEL_REGISTRY` to point to shared storage accessible by K8s pods [8]

**Nexus-specific gap:** Neither NVIDIA documentation explicitly covers Nexus integration. The implied workflow is:
1. Use NGC Container Replicator or manual `docker pull` + `docker save` on internet-connected machine
2. Transfer tar archives to airgapped network
3. Load into Nexus Docker registry using `docker load` + `docker tag` + `docker push` to Nexus
4. Configure K8s to pull from Nexus instead of nvcr.io

This workflow requires custom scripting - NVIDIA doesn't provide turnkey Nexus automation [7][8].

### Summary

NVIDIA provides the NGC Container Replicator as an official automation tool for cloning NGC registries, supporting cron-based synchronization and Docker tar export for airgapped registries [7]. The TAO Toolkit demonstrates NVIDIA's airgapped deployment pattern: download assets with NGC API key while online, transfer to airgapped environment, then deploy with `AIRGAPPED_MODE=true` pointing to local storage [8]. However, there's a critical documentation gap for Kubernetes/OpenShift users: while the Container Replicator supports Docker tar export and TAO Toolkit shows environment-based configuration, neither explicitly documents the Nexus/Harbor integration workflow. Users must bridge this gap with custom scripting to: pull containers online, export as tars, transfer to airgapped network, then load into Nexus Docker registry and reconfigure K8s image pull policies [7][8]. The lack of turnkey Nexus automation represents a significant friction point for enterprise airgapped deployments, particularly for RunAI on OpenShift where container registry integration is critical infrastructure.
