## Context

The original `build-and-publish.yml` was inherited from `lecode-official/comfyui-docker` and was designed for a GPU/CUDA matrix: it uses a `generate-version-combinations` job that queries Docker Hub for available PyTorch CUDA image tags, generates a JSON matrix, then fans out into parallel GPU builds for each CUDA/cuDNN combination.

This fork (`LoonWorks/comfyui-docker-cpu`) has rewritten the Dockerfile to be CPU-only using `python:3.13-slim` and PyTorch CPU wheels. There is no CUDA or cuDNN dependency. The matrix-based workflow is incompatible: the `generate-version-combinations` job will produce zero rows (no CUDA tags match) causing an empty-matrix failure, and the `build-and-publish` job passes `CUDA_VERSION`/`CUDNN_VERSION` build args that no longer exist in the Dockerfile.

A replacement workflow must be simple, single-stage, and publish a single image variant.

## Goals / Non-Goals

**Goals:**
- Build and push `ghcr.io/loonworks/comfyui-docker-cpu` to GHCR on each versioned release tag.
- Derive `COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`, and `PYTORCH_VERSION` from Dockerfile `ARG` defaults (extracted inline in the workflow) and pass them as build args.
- Produce a useful set of semver-based tags without CUDA/cuDNN variants.
- Retain build-provenance attestation for supply-chain security.
- Maintain the existing trigger: tag push matching `v[0-9]+.[0-9]+.[0-9]+`.

**Non-Goals:**
- Multi-arch (ARM) builds — single `linux/amd64` is sufficient for the current use case.
- Scheduled builds or branch-based push triggers.
- Changing the Dockerfile or entrypoint.

## Decisions

### Single job, no matrix
The entire `generate-version-combinations` job and its Python script are removed. There is only one image variant, so no matrix strategy is needed. This eliminates the Docker Hub API dependency, Python setup, and JSON matrix plumbing.

_Alternative considered_: Keep the matrix job but short-circuit it to emit a single row. Rejected because it adds unnecessary complexity and still fails if Docker Hub is unreachable.

### Extract versions from Dockerfile with grep
Version ARG defaults are extracted by grepping `source/Dockerfile` with a shell one-liner inside the workflow step. This avoids hardcoding versions in the workflow file, keeping the Dockerfile as the single source of truth.

```yaml
run: |
  echo "comfyui_version=$(grep -oP '(?<=ARG COMFYUI_VERSION=)\S+' source/Dockerfile)" >> $GITHUB_OUTPUT
  echo "comfyui_manager_version=$(grep -oP '(?<=ARG COMFYUI_MANAGER_VERSION=)\S+' source/Dockerfile)" >> $GITHUB_OUTPUT
  echo "pytorch_version=$(grep -oP '(?<=ARG PYTORCH_VERSION=)\S+' source/Dockerfile)" >> $GITHUB_OUTPUT
```

_Alternative considered_: Hardcode version strings in the workflow. Rejected: would require two-place updates on every version bump.

### Tag strategy — drop CUDA/cuDNN suffixes, keep PyTorch suffixes
The simplified tag set keeps:
- `sha-<short>` — traceability
- `{{major}}`, `{{major}}.{{minor}}`, `{{version}}` — standard semver (skipping `{{major}}` when it is `0`)
- `{{version}}-comfyui-<V>` — ComfyUI version pinning
- `{{version}}-comfyui-<V>-pytorch-<P>` — full dependency pinning
- `latest` — always on every tag push (single variant)

CUDA and cuDNN suffix variants are removed entirely since there is only one image.

### Use `actions/checkout@v4`
The upstream workflow referenced `actions/checkout@v6` which does not exist (latest stable is v4). Corrected.

### Author label — neutral fork attribution
`org.opencontainers.image.authors` is updated to `LoonWorks` and the email removed to avoid referencing the upstream author.

## Risks / Trade-offs

- **Grep fragility**: If the Dockerfile ARG lines change format, version extraction will silently produce empty strings. → Mitigation: validate outputs are non-empty with a shell assertion; CI will fail loudly.
- **GHCR token scope**: `packages: write` permission is already present in the original workflow; no change needed.
- **`latest` always set**: Unlike the original workflow which conditioned `latest` on `is_default`, the CPU fork has only one image so `latest` is always appropriate.

## Migration Plan

1. The `generate-version-combinations` job is removed entirely.
2. The `build-and-publish` job is rewritten in place: `needs` removed, matrix removed, `CUDA_VERSION`/`CUDNN_VERSION` build args removed, tag patterns updated.
3. No rollback complexity: the old workflow is simply replaced; reversion is a `git revert` if needed.

## Open Questions

- None.
