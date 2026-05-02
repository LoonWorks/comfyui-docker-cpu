## 1. Dockerfile CPU-only foundation

- [x] 1.1 Remove GPU-oriented build arguments and any CUDA/cuDNN/NVIDIA-specific image, package, and environment configuration from `source/Dockerfile`
- [x] 1.2 Keep the Python 3.13 slim base and install only required system packages for ComfyUI runtime and build reproducibility
- [x] 1.3 Configure deterministic build steps (package manager cleanup, pinned checkout flow) to keep the image minimal and production-safe

## 2. Python environment and dependency installation

- [x] 2.1 Create and configure a dedicated virtual environment (for example `/opt/venv`) and ensure PATH/default interpreter use it
- [x] 2.2 Install CPU-only PyTorch and related packages from the CPU wheel index, then install ComfyUI and ComfyUI Manager requirements inside the venv
- [x] 2.3 Ensure ComfyUI and ComfyUI Manager are cloned to existing locations and checked out at configured version tags

## 3. Runtime behavior compatibility

- [x] 3.1 Preserve existing working directory, exposed port, entrypoint wiring, and startup argument pass-through behavior
- [x] 3.2 Preserve non-root user creation and ownership flow driven by `USER_ID` and `GROUP_ID`
- [x] 3.3 Preserve manager symlink and custom node requirements installation behavior expected by `entrypoint.sh`

## 4. Validation and operability checks

- [x] 4.1 Build image successfully using `docker build -t comfyui-cpu source`
- [x] 4.2 Run container without NVIDIA runtime/GPU flags and verify ComfyUI serves on port 8188 with existing host volume mappings
- [x] 4.3 Verify runtime works in WSL2 no-GPU environments and document any constraints discovered during validation
