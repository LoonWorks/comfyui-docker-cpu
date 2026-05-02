## Context

The current image history and project documentation are GPU-centric and still reference CUDA-enabled execution paths. The Dockerfile in `source/` must be rewritten as a CPU-only build while preserving established runtime conventions: ComfyUI and ComfyUI Manager repository checkout, startup directory preparation, manager symlink behavior, optional non-root execution using `USER_ID`/`GROUP_ID`, and listening on port 8188. The runtime must work in WSL2 environments without NVIDIA runtime dependencies.

## Goals / Non-Goals

**Goals:**
- Produce a deterministic CPU-only Docker build on Python 3.13.
- Remove all CUDA/cuDNN/GPU-specific dependencies and configuration from the image build.
- Install CPU-only PyTorch and related packages in an isolated virtual environment used consistently at runtime.
- Keep all existing container runtime behavior that users rely on (entrypoint behavior, paths, and port exposure).
- Keep the image minimal and suitable for production operation.

**Non-Goals:**
- Optimizing inference performance beyond CPU-only correctness and compatibility.
- Changing ComfyUI feature flags, API behavior, or default startup CLI options.
- Redesigning entrypoint logic or host volume mappings.
- Introducing GPU/accelerator fallbacks in the same image variant.

## Decisions

1. Base image remains `python:3.13-slim` with no CUDA runtime layers.
- Rationale: this minimizes image size/surface area and guarantees no implicit NVIDIA dependency.
- Alternatives considered:
  - `pytorch/pytorch` CUDA/runtime tags: rejected because they conflict with CPU-only goal.
  - Debian/Ubuntu base with custom Python 3.13 install: rejected due to extra complexity and lower reproducibility.

2. Use a dedicated virtual environment (e.g., `/opt/venv`) and make it the default interpreter path.
- Rationale: isolates Python dependencies from system packages and gives a clear executable contract for build and runtime.
- Alternatives considered:
  - System-level `pip` installs: rejected due to higher risk of package drift and harder dependency control.

3. Install CPU-only PyTorch explicitly from the CPU wheel index.
- Rationale: prevents accidental GPU wheel resolution and ensures compatibility on hosts without CUDA.
- Alternatives considered:
  - Relying on default PyPI resolver behavior: rejected because package resolution can drift and may not enforce CPU-only intent.

4. Preserve ComfyUI and ComfyUI Manager pinning and install flow.
- Rationale: existing tags (`COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`) provide reproducible source checkout and are already part of the project contract.
- Alternatives considered:
  - Floating branches or latest commits: rejected due to reduced reproducibility.

5. Maintain entrypoint contract and runtime behavior while updating Python path to the venv interpreter.
- Rationale: preserves user-visible behavior (same startup flags, same directory setup, same non-root handling) while making Python execution consistent with the new build design.
- Alternatives considered:
  - Replacing entrypoint semantics with a direct CMD call: rejected because it would break manager symlink and custom-node requirements behavior.

## Risks / Trade-offs

- [Risk] CPU-only PyTorch availability may vary by version constraints from ComfyUI requirements. -> Mitigation: pin PyTorch package variants compatible with Python 3.13 and verify at image build time.
- [Risk] Custom node requirements may install GPU-targeted extras at startup. -> Mitigation: keep startup install behavior but rely on user-selected custom nodes; document that GPU-only custom nodes are out of scope for CPU image.
- [Risk] Entrypoint currently references a fixed Python path from prior image lineage. -> Mitigation: update runtime invocation to use venv interpreter path and verify both root and non-root code paths.
- [Trade-off] CPU-only image improves portability and simplicity but may reduce model performance compared to GPU variants.

## Migration Plan

1. Replace Dockerfile GPU-oriented package assumptions with CPU-only Python 3.13 build steps.
2. Build image with `docker build -t comfyui-cpu source` and validate successful dependency resolution.
3. Start container in a WSL2 environment without NVIDIA runtime and verify UI is accessible on port 8188.
4. Validate non-root startup path (`USER_ID`/`GROUP_ID`) and file ownership behavior in mounted volumes.
5. Rollback path: revert to previous Dockerfile or prior published image tag if regressions are found.

## Open Questions

- Should this CPU-only variant become the default image path, or should a separate tag/branch strategy be introduced in a follow-up change?
- Should README examples in this change remove GPU runtime flags, or should documentation updates be tracked in a separate change?
