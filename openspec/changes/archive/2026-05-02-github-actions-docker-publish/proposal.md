## Why

The existing GitHub Actions workflow was built for a GPU/CUDA matrix build strategy and references the upstream `lecode-official` repository. This fork is CPU-only with a single image variant, so a new simplified workflow is needed to build and publish the CPU-only Docker image to `ghcr.io/loonworks/comfyui-docker-cpu` on every versioned release.

## What Changes

- Replace `.github/workflows/build-and-publish.yml` with a CPU-only single-build workflow that eliminates the CUDA/cuDNN version matrix.
- Extract only `COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`, and `PYTORCH_VERSION` from the Dockerfile for build arguments.
- Publish to `ghcr.io/${{ github.repository }}` (resolves to `ghcr.io/loonworks/comfyui-docker-cpu`).
- Tag images with semver tags and a `sha-` prefix tag, without CUDA/cuDNN suffix variants.
- Keep attestation step for supply-chain security.

## Capabilities

### New Capabilities
- `cpu-only-ci-publish`: Build and publish a single CPU-only Docker image to GHCR on versioned tag push, using simplified semver tagging without a CUDA/cuDNN matrix.

### Modified Capabilities
- None.

## Impact

- Affected file: `.github/workflows/build-and-publish.yml` (full replacement).
- No runtime or source code changes.
- Publish target: `ghcr.io/loonworks/comfyui-docker-cpu` via `github.repository` context.
- Trigger remains: push of tags matching `v[0-9]+.[0-9]+.[0-9]+`.
