## ADDED Requirements

### Requirement: CPU-only Python 3.13 base image
The container build SHALL use a Python 3.13 CPU-only base image and SHALL NOT include CUDA, cuDNN, or NVIDIA runtime dependencies in the Dockerfile.

#### Scenario: Build image contains no GPU runtime assumptions
- **WHEN** the Dockerfile is evaluated for base image and system packages
- **THEN** no CUDA/cuDNN/NVIDIA image tags, packages, or environment variables are present

### Requirement: CPU-only PyTorch installation
The build SHALL install CPU-only PyTorch and required related packages in an isolated virtual environment, and ComfyUI runtime SHALL use that environment's Python executable.

#### Scenario: PyTorch install source enforces CPU wheels
- **WHEN** Python dependencies are installed during image build
- **THEN** PyTorch packages are resolved from a CPU-only package source and available from the configured virtual environment

### Requirement: ComfyUI and manager installation parity
The build SHALL clone and checkout configured ComfyUI and ComfyUI Manager versions, install their Python requirements, and retain startup compatibility with the existing manager symlink workflow.

#### Scenario: Startup keeps manager symlink behavior
- **WHEN** the container starts
- **THEN** ComfyUI Manager is linked under `custom_nodes` and custom node requirements installation still executes

### Requirement: Runtime behavior compatibility
The container runtime SHALL preserve existing entrypoint behavior, non-root user creation flow, exposed port metadata, and host volume path expectations used by current deployments.

#### Scenario: Existing deployment mappings continue to work
- **WHEN** a container is started with the same host volume mounts and published port as before
- **THEN** ComfyUI starts successfully on port 8188 and uses the mounted model/custom-node/output directories

### Requirement: WSL2 no-GPU operability
The image SHALL run correctly in WSL2 environments without requiring NVIDIA runtime flags or GPU device access.

#### Scenario: Container runs without NVIDIA runtime
- **WHEN** the container is launched in WSL2 on a machine without NVIDIA runtime configuration
- **THEN** ComfyUI process starts and remains healthy without GPU-related startup failures
