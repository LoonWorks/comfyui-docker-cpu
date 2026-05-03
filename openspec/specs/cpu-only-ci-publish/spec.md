## Requirements

### Requirement: CPU-only CI publishes single Docker image on tag push
The workflow SHALL build and push a single Docker image to `ghcr.io/${{ github.repository }}` whenever a Git tag matching `v[0-9]+.[0-9]+.[0-9]+` is pushed. No CUDA, cuDNN, or GPU-related build arguments SHALL be passed.

#### Scenario: Tag push triggers build
- **WHEN** a Git tag matching `v[0-9]+.[0-9]+.[0-9]+` is pushed to the repository
- **THEN** the `build-and-publish` job MUST run and push the built image to GHCR

#### Scenario: No CUDA/cuDNN build args
- **WHEN** the Docker build step runs
- **THEN** only `COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`, and `PYTORCH_VERSION` SHALL be passed as build arguments; `CUDA_VERSION` and `CUDNN_VERSION` MUST NOT be present

#### Scenario: Version extraction from Dockerfile
- **WHEN** the workflow extracts dependency versions
- **THEN** it SHALL read `COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`, and `PYTORCH_VERSION` from the `ARG` defaults in `source/Dockerfile` and fail loudly if any value is empty

### Requirement: Image is tagged with semver and dependency-version tags
The workflow SHALL apply the following tags to the published image using `docker/metadata-action`:
- `sha-<short>` — short commit SHA
- `latest` — always set (single image variant)
- `{{major}}` — only when major version is not `0`
- `{{major}}.{{minor}}`
- `{{version}}`
- `{{version}}-comfyui-<COMFYUI_VERSION>`
- `{{version}}-comfyui-<COMFYUI_VERSION>-pytorch-<PYTORCH_VERSION>`

No CUDA or cuDNN suffix tags SHALL be generated.

#### Scenario: Semver tags are applied
- **WHEN** a tag `v1.2.3` is pushed
- **THEN** the image MUST be tagged with at minimum `1.2.3`, `1.2`, `1`, `latest`, and a `sha-` prefix tag

#### Scenario: v0.x tags skip major-only tag
- **WHEN** a tag `v0.5.0` is pushed
- **THEN** the image MUST NOT receive a `0` tag (major-only)

### Requirement: Build provenance attestation is generated
The workflow SHALL generate and push a build-provenance attestation using `actions/attest-build-provenance@v3`.

#### Scenario: Attestation created on successful push
- **WHEN** the Docker image is successfully built and pushed
- **THEN** an attestation referencing `subject-digest` from the build step output MUST be pushed to the registry

### Requirement: Workflow job uses correct permissions
The `build-and-publish` job SHALL declare the following `permissions`:
- `contents: read`
- `packages: write`
- `attestations: write`
- `id-token: write`

#### Scenario: Permissions set correctly
- **WHEN** the workflow job definition is reviewed
- **THEN** all four permissions listed above MUST be present
