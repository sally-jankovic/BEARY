# LLM Model Loading and Serving in Air-Gapped GPUaaS — Subtopic Notes

<!-- Subtopic notes for LLM serving in air-gapped environments -->
<!-- Citations are stored in whitepaper/runai-gpuaas-airgapping-references.md -->

## Q1: What are the best practices for loading large LLM model weights into air-gapped GPU environments?

**NVIDIA NIM Air-Gap Deployment** [20]:
NVIDIA NIM for LLMs supports serving models in air-gapped systems with no internet connection and no connection to NGC registry or Hugging Face Hub.

**Two Deployment Options**:

1. **Create Model Store Method**:
   - Use `create-model-store` command within NIM container
   - Requires HF_TOKEN on connected system to create model repository
   - Creates local model store that can be transferred to air-gapped environment
   ```bash
   docker run ... $IMG_NAME create-model-store --model-repo $NIM_MODEL_NAME --model-store /model-repo
   ```
   - In air-gapped environment, run without HF_TOKEN:
   ```bash
   docker run ... -e NIM_MODEL_NAME=/model-repo ... $IMG_NAME
   ```

2. **Download-to-Cache Method**:
   - Use `download-to-cache` command to download model profiles
   - Transfer cache directory to air-gapped system
   - Run NIM without NGC_API_KEY in air-gapped environment
   ```bash
   # On connected system
   download-to-cache -p <profile_hash>
   
   # Transfer cache
   cp -r "$LOCAL_NIM_CACHE"/* "$AIR_GAP_NIM_CACHE"
   
   # Run in air-gapped environment (no API keys needed)
   docker run ... -v "$AIR_GAP_NIM_CACHE:/opt/nim/.cache" ... $IMG_NAME
   ```

**Ollama Air-Gap Export** [21]:
```bash
# On connected machine
ollama pull qwen2.5-coder:7b
ollama export qwen2.5-coder:7b > model.tar

# Transfer via USB/secure media to air-gapped machine
ollama import model.tar
```

**Model Weight Transfer Best Practices**:
- Pre-download all model weights on connected system
- Package with all dependencies (tokenizers, configs)
- Verify checksums after transfer (SHA256)
- Use safetensors format when available (faster loading, security)
- Consider quantized models (AWQ, GPTQ) to reduce transfer size

---

## Q2: How can organizations implement on-demand LLM serving with minimal cold-start times in air-gapped environments?

**vLLM Kubernetes Deployment** [22]:
vLLM provides scalable LLM serving on Kubernetes with multiple deployment options:
- Native Kubernetes manifests
- Helm charts
- KServe integration
- KubeRay for distributed serving
- Knative for serverless scaling

**vLLM GPU Deployment Pattern** [22]:
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: vllm
        image: vllm/vllm-openai:latest
        args: ["vllm serve <model> --trust-remote-code"]
        resources:
          limits:
            nvidia.com/gpu: "1"
        volumeMounts:
        - mountPath: /root/.cache/huggingface
          name: cache-volume  # PVC for model cache
```

**Cold-Start Optimization Strategies**:

1. **Persistent Model Cache**: Use PersistentVolumeClaims to store model weights, avoiding re-download on pod restart

2. **Model Pre-loading**: Keep models loaded in memory using:
   - Always-on deployments for critical models
   - Warm standby replicas
   - Model weight streaming from fast storage (NVMe)

3. **GPU Memory Optimization**:
   - Use quantized models (INT8, INT4) for faster loading
   - Implement KV cache offloading
   - Use prefix caching for common prompts

4. **Inference Server Options** [23]:
   - **vLLM**: PagedAttention for best memory efficiency, 73% cost reduction reported
   - **TGI (Text Generation Inference)**: Exceptional on specific models (36% improvement on MPT-30B)
   - **Triton**: Versatile across diverse workloads
   - **Ollama**: Lightweight, simple deployment

**Run:ai + Knative Integration** [16]:
Run:ai installer supports optional Knative installation with `--knative` flag, enabling:
- Auto-scaling inference workloads
- Scale-to-zero for cost optimization
- Rapid scale-up on demand

**Air-Gapped Considerations**:
- Pre-pull all container images to private registry
- Store model weights on high-speed shared storage (Lustre, BeeGFS)
- Use GPUDirect Storage for direct GPU-storage transfer
- Implement local model registry/catalog for discovery

---

## Summary

Loading LLM models into air-gapped environments requires careful preparation on connected systems. NVIDIA NIM provides two methods: create-model-store and download-to-cache, both enabling fully offline operation. Ollama offers simple export/import for smaller models.

For on-demand serving, vLLM on Kubernetes provides the best balance of performance and flexibility, with PagedAttention delivering significant memory efficiency gains. Cold-start times can be minimized through persistent model caches, pre-loading strategies, and quantized models. Run:ai's Knative integration enables serverless-style auto-scaling even in air-gapped environments.

The key gap remains the lack of a unified "model registry" pattern for air-gapped environments that would enable true on-demand model loading similar to container image registries. Organizations must currently manage model weights separately from container images, adding operational complexity.

