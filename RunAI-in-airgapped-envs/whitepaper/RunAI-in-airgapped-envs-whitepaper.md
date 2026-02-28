# Deploying NVIDIA NIM Containers in Airgapped Environments: Challenges and Solutions for RunAI on OpenShift

**Author:** Background Research Agent (BEARY)  
**Date:** 2026-02-27

---

## Abstract

This whitepaper examines the challenges of deploying NVIDIA NIM (NVIDIA Inference Microservices) containers from the NGC registry in airgapped Kubernetes environments, specifically for RunAI deployments on OpenShift. While NVIDIA provides official tools and documentation for airgapped NIM deployments, a critical gap exists in the integration workflow for enterprise container registries like Nexus. We analyze NVIDIA's two-phase deployment approach (download-transfer-deploy), evaluate the NGC Container Replicator automation tool, and identify the custom scripting required to bridge NGC and Nexus. The research reveals that while model caching workflows are well-documented, the container registry mirroring process for production Kubernetes deployments requires manual intervention. We propose practical automation strategies and highlight areas where NVIDIA documentation could better serve enterprise airgapped deployments.

## Introduction

Organizations deploying GPU-accelerated AI workloads in secure, airgapped environments face a fundamental challenge: how to obtain and manage NVIDIA NIM containers when direct internet access to the NGC registry is prohibited. This challenge is particularly acute for RunAI deployments on OpenShift, where container images must flow through intermediary registries like Nexus before reaching the airgapped Kubernetes cluster.

This whitepaper addresses three key questions:
1. How do NVIDIA NIM containers integrate with NGC registry in airgapped environments?
2. What role does Nexus play as an artifact repository for NGC containers?
3. What automation strategies exist for syncing NGC artifacts to local registries?

The research is motivated by a real-world use case: building a GPUaaS (GPU-as-a-Service) platform using RunAI on an OpenShift cluster that requires a go-between for internet access, specifically for installing NIM artifacts in a local cluster registry.

## Background

### NVIDIA NIM and NGC Registry

NVIDIA NIM for LLMs are containerized inference services designed to simplify the deployment of large language models on GPU infrastructure [1]. These containers are distributed through the NGC (NVIDIA GPU Cloud) registry at nvcr.io, which requires authentication via NGC API keys [1].

### Airgapped Environments

An airgapped system is isolated from unsecured networks, including the public internet [4]. In such environments, the default mechanism of pulling containers directly from nvcr.io is not possible, necessitating alternative workflows for software updates and container image distribution [4].

### RunAI and OpenShift Context

RunAI is a Kubernetes-native platform for orchestrating GPU workloads. When deployed on OpenShift (Red Hat's enterprise Kubernetes distribution), it operates within strict security boundaries that often include airgap requirements. OpenShift's integration with container registries requires specific certificate injection mechanisms using ConfigMaps [2].

## Current Challenges

### The Two-Phase Deployment Pattern

NVIDIA's official approach to airgapped NIM deployment follows a consistent two-phase pattern [1][8]:

**Phase 1: Internet-Connected Environment**
- Download NIM containers from nvcr.io using NGC API keys
- Download model profiles using `download-to-cache` utility or create model stores with `create-model-store` command
- Cache models locally in a directory structure

**Phase 2: Airgapped Environment**
- Transfer cached models and containers to the isolated network
- Deploy NIM containers pointing to local cache with environment variables like `AIRGAPPED_MODE=true`
- Run without NGC API keys or HuggingFace tokens

This pattern works well for standalone deployments but leaves gaps for Kubernetes environments requiring container registry integration.

### The Container Registry Gap

NVIDIA's NIM documentation thoroughly covers model caching but provides limited guidance on the container registry mirroring process needed for production Kubernetes deployments [1][2]. The NIM Operator documentation mentions "mirrored local model registries" as an option [2] but doesn't detail the implementation.

For enterprise Kubernetes deployments, particularly on OpenShift with Nexus, users must bridge this gap themselves. The workflow requires:
1. Pulling NIM containers from nvcr.io while online
2. Exporting containers as tar archives
3. Transferring tars to the airgapped network
4. Loading tars into Nexus Docker registry
5. Reconfiguring Kubernetes image pull policies to use Nexus instead of nvcr.io

### Nexus as an Intermediary

Nexus Repository (Sonatype OSS Nexus) is commonly used in airgapped Kubernetes environments to provide local storage and distribution of artifacts [5]. For airgapped deployments, Nexus requires:
- **Docker repository** for container images (minimum 15GB storage) [5]
- **RAW repository** for binaries and Helm charts (minimum 15GB storage) [5]

While Nexus is a standard solution for airgapped Kubernetes, NVIDIA documentation doesn't explicitly address Nexus integration for NIM containers [4][5]. The established pattern from DGX systems suggests creating private mirrors of public repositories, but the specific workflow for Nexus remains undocumented [4].

## Automation Solutions and Workarounds

### NGC Container Replicator

NVIDIA provides an open-source NGC Container Replicator tool designed to create local clones of the NGC container registry [7]. This tool offers several automation capabilities:

**Key Features:**
- Automatic synchronization with NGC registry via cron jobs (daily/weekly) [7]
- Conversion of NGC containers to Singularity images for HPC environments [7]
- Docker tar archive export for "sneakernetting" to airgapped Docker registries [7]
- NGC API key required only for the replicator, not end users [7]

**Example Usage:**
```bash
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /shared/containers:/output \
  deepops/replicator \
  --project=hpc \
  --image=namd \
  --singularity \
  --no-exporter \
  --api-key=<NGC_API_KEY>
```

The `--no-exporter` flag disables Docker tar generation, suggesting that removing this flag enables tar export for airgapped Docker registries [7]. However, the documentation focuses on HPC/Singularity use cases rather than Kubernetes/OpenShift deployments.

### TAO Toolkit Reference Implementation

The NVIDIA TAO Toolkit provides a reference implementation for airgapped workflows with clear automation patterns [8]:

**Three-Phase Approach:**
1. **Asset Preparation** (online): Use `pretrained_models.py` script to download models with NGC API key
2. **Secure Transfer**: Move assets to airgapped environment
3. **Deployment**: Deploy with Helm charts using environment variables:
   - `AIRGAPPED_MODE=true` signals offline operation
   - `LOCAL_MODEL_REGISTRY` points to shared storage accessible by K8s pods

This pattern demonstrates NVIDIA's intended approach but still doesn't explicitly cover Nexus integration [8].

### Custom Scripting Requirements

Given the documentation gaps, users must implement custom automation to connect NGC Container Replicator output to Nexus. The implied workflow:

**Internet-Connected Side:**
```bash
# Use NGC Container Replicator with tar export enabled
docker run deepops/replicator \
  --project=nim \
  --image=<nim-container> \
  --api-key=<NGC_API_KEY>
  # Note: --no-exporter flag removed to enable tar generation

# Or manual approach
docker pull nvcr.io/nim/meta/llama-3.1-8b-instruct:latest
docker save nvcr.io/nim/meta/llama-3.1-8b-instruct:latest > nim-llama.tar
```

**Transfer Phase:**
```bash
# Secure copy to airgapped network
scp nim-llama.tar airgapped-jumphost:/transfer/
```

**Airgapped Side:**
```bash
# Load into local Docker
docker load < nim-llama.tar

# Tag for Nexus registry
docker tag nvcr.io/nim/meta/llama-3.1-8b-instruct:latest \
  nexus.internal:5000/nim/meta/llama-3.1-8b-instruct:latest

# Push to Nexus
docker push nexus.internal:5000/nim/meta/llama-3.1-8b-instruct:latest
```

**Kubernetes Configuration:**
```yaml
# Update image pull policy in RunAI/NIM deployment
spec:
  containers:
  - name: nim
    image: nexus.internal:5000/nim/meta/llama-3.1-8b-instruct:latest
    imagePullPolicy: IfNotPresent
```

This workflow requires custom scripting and is not provided as a turnkey solution by NVIDIA [7][8].

## Implementation Strategies

### Recommended Approach for RunAI on OpenShift

Based on the research findings, the recommended implementation strategy combines NVIDIA's official tools with custom automation:

**1. Leverage NGC Container Replicator for Automation**
- Deploy Container Replicator on an internet-connected machine
- Configure it to run as a cron job for regular synchronization
- Enable Docker tar export (remove `--no-exporter` flag)
- Store tar archives in a staging directory

**2. Implement Custom Transfer and Load Scripts**
- Create automation scripts to:
  - Transfer tar archives to airgapped network
  - Load tars into local Docker daemon
  - Retag images for Nexus registry
  - Push to Nexus Docker repository
- Schedule these scripts to run after Container Replicator completes

**3. Configure OpenShift for Airgapped Operation**
- Enable certificate injection for Nexus using ConfigMaps with `config.openshift.io/inject-trusted-cabundle: "true"` label [2]
- Configure image pull secrets for Nexus authentication
- Update RunAI and NIM Operator deployments to reference Nexus registry paths

**4. Implement Model Caching Workflow**
- Use NVIDIA's documented `download-to-cache` workflow for model profiles [1]
- Transfer model cache to airgapped environment
- Mount cache into NIM containers via persistent volumes
- Set `AIRGAPPED_MODE=true` in deployment configurations [8]

### Automation Script Template

A practical automation script might follow this structure:

```bash
#!/bin/bash
# sync-ngc-to-nexus.sh

# Phase 1: Run on internet-connected machine
if [ "$PHASE" == "download" ]; then
  docker run deepops/replicator \
    --project=nim \
    --api-key=$NGC_API_KEY \
    --output=/staging/tars
fi

# Phase 2: Run on airgapped machine
if [ "$PHASE" == "upload" ]; then
  for tar in /transfer/*.tar; do
    docker load < $tar
    IMAGE=$(docker images --format "{{.Repository}}:{{.Tag}}" | head -1)
    NEXUS_IMAGE="nexus.internal:5000/${IMAGE#nvcr.io/}"
    docker tag $IMAGE $NEXUS_IMAGE
    docker push $NEXUS_IMAGE
  done
fi
```

### Potential Pitfalls and Mitigations

**Storage Requirements**
- NIM containers can be large (10-50GB per model)
- Ensure adequate storage on both internet-connected staging and Nexus registry
- Plan for 2-3x container size for tar archives and loaded images

**Version Management**
- NGC containers update frequently (monthly for some models)
- Implement version tracking to avoid confusion between online and airgapped versions
- Consider semantic versioning or date-based tags

**Certificate Management**
- OpenShift requires proper certificate injection for Nexus [2]
- Test certificate chains thoroughly before production deployment
- Document certificate renewal procedures

**Network Bandwidth**
- Initial sync of multiple NIM containers can consume significant bandwidth
- Plan transfers during off-peak hours
- Consider incremental sync strategies for updates

## Discussion

### The Documentation Gap

The most significant finding from this research is the documentation gap between NVIDIA's airgapped deployment guidance and enterprise Kubernetes reality. NVIDIA provides excellent documentation for:
- Model caching workflows (`download-to-cache`, `create-model-store`) [1]
- Airgapped deployment patterns (TAO Toolkit reference) [8]
- Automation tools (NGC Container Replicator) [7]

However, these resources don't connect into a cohesive workflow for the common enterprise scenario: Kubernetes/OpenShift with Nexus or Harbor as the container registry intermediary. The NGC Container Replicator's Docker tar export capability suggests NVIDIA anticipates this use case, but the workflow isn't explicitly documented [7].

### Comparison with Other NVIDIA Products

Interestingly, NVIDIA's approach to airgapped deployments varies across products:
- **DGX Systems**: Clear guidance on creating private repository mirrors [4]
- **TAO Toolkit**: Well-defined three-phase workflow with environment variables [8]
- **NIM for LLMs**: Strong model caching documentation, weak container registry guidance [1][2]

This inconsistency suggests NIM's airgapped documentation is still maturing compared to more established NVIDIA products.

### Alternative Solutions

While this whitepaper focuses on Nexus, alternative approaches exist:

**Harbor Registry**
- Open-source alternative to Nexus with similar capabilities
- May have better Kubernetes integration
- Same documentation gap applies

**Direct Docker Registry**
- Simpler than Nexus but fewer features
- Requires 15GB minimum storage [5]
- Still requires custom scripting for NGC integration

**Proxy-Based Approach**
- NIM Operator supports proxy access to NGC [2]
- Requires less infrastructure than full mirroring
- May not satisfy strict airgap requirements
- Introduces single point of failure

### Implications for RunAI Deployments

For RunAI deployments specifically, the lack of turnkey Nexus automation represents a significant friction point. RunAI's value proposition is simplifying GPU orchestration, but the airgapped NIM deployment workflow undermines this simplicity. Organizations must either:
1. Develop and maintain custom automation scripts
2. Accept manual container management processes
3. Relax airgap requirements to use proxy-based approaches

None of these options are ideal for production enterprise deployments.

## Conclusion

Deploying NVIDIA NIM containers in airgapped RunAI environments on OpenShift is technically feasible but requires bridging a documentation gap with custom automation. NVIDIA provides the building blocks—the NGC Container Replicator for automation and the two-phase download-transfer-deploy pattern—but doesn't assemble them into a complete workflow for enterprise container registries like Nexus.

**Key Takeaways:**

1. **NVIDIA's airgapped NIM deployment is well-designed** for model caching but incomplete for container registry integration [1][8]

2. **NGC Container Replicator is a powerful automation tool** that supports Docker tar export, but its Kubernetes/Nexus integration isn't documented [7]

3. **Custom scripting is currently required** to connect NGC Container Replicator output to Nexus Docker registries [7][8]

4. **OpenShift adds complexity** with certificate injection requirements that must be properly configured [2]

5. **The workflow is automatable** using cron jobs and custom scripts, but requires engineering effort to implement and maintain

**Recommendations for NVIDIA:**

- Expand NGC Container Replicator documentation to explicitly cover Kubernetes/OpenShift scenarios
- Provide reference implementations for Nexus and Harbor integration
- Add Nexus-specific examples to NIM Operator documentation
- Consider developing a Nexus plugin or Helm chart for turnkey integration

**Recommendations for Practitioners:**

- Start with NGC Container Replicator as the foundation for automation
- Develop custom scripts to handle tar transfer, load, retag, and push to Nexus
- Implement robust version tracking and testing procedures
- Document the workflow thoroughly for operational teams
- Plan for ongoing maintenance as NGC containers update

**Areas for Further Research:**

- Performance comparison of proxy-based vs. full mirroring approaches
- Best practices for version management in airgapped NIM deployments
- Integration patterns for other enterprise registries (Harbor, JFrog Artifactory)
- Automation frameworks that could standardize the NGC-to-Nexus workflow

While the current state requires custom engineering, the fundamental components exist to build robust, automated airgapped NIM deployments. Organizations willing to invest in the initial automation will find the ongoing operational burden manageable, particularly when using cron-based synchronization with the NGC Container Replicator.

## References

See RunAI-in-airgapped-envs-references.md for the full bibliography.
In-text citations use bracketed IDs, e.g., [1], [2].
