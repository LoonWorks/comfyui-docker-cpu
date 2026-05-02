## Why

The current container setup is still oriented around GPU-enabled workflows and does not provide an explicit, production-safe CPU-only build path for environments without NVIDIA runtime support. A dedicated CPU-only Docker build using Python 3.13 is needed to ensure reproducible builds and reliable execution in WSL2 and other non-GPU hosts.

## What Changes

- Rewrite `source/Dockerfile` as a CPU-only image based on Python 3.13, removing CUDA/cuDNN/GPU-specific build arguments, packages, and configuration.
- Install CPU-only PyTorch and related Python packages in a dedicated virtual environment and ensure ComfyUI plus ComfyUI Manager are cloned, pinned, and installed correctly.
- Preserve existing runtime behavior including entrypoint semantics, non-root user support via `USER_ID`/`GROUP_ID`, directory and symlink setup performed at startup, and existing port/volume compatibility.
- Keep the image minimal and reproducible with deterministic install steps that are safe for production use.

## Capabilities

### New Capabilities
- `cpu-only-comfyui-image`: Build and run ComfyUI as a fully CPU-only Docker image on Python 3.13 while preserving existing container runtime behavior and host volume compatibility.

### Modified Capabilities
- None.

## Impact

- Affected code: `source/Dockerfile` (primary) with compatibility constraints from `source/entrypoint.sh`.
- Runtime/dependencies: Python environment management (venv), CPU-only PyTorch package source, and removal of GPU-runtime assumptions.
- Deployment targets: WSL2/Linux hosts without NVIDIA runtime; same published port and mounted volume paths as current usage.
