## 1. Remove matrix job and infrastructure

- [x] 1.1 Delete the entire `generate-version-combinations` job from `.github/workflows/build-and-publish.yml`
- [x] 1.2 Remove the `needs: generate-version-combinations` field from the `build-and-publish` job
- [x] 1.3 Remove the `strategy.matrix` block from the `build-and-publish` job

## 2. Add version extraction step

- [x] 2.1 Add a `Extract Versions from Dockerfile` step before the checkout or after checkout that uses `grep -oP` to extract `COMFYUI_VERSION`, `COMFYUI_MANAGER_VERSION`, and `PYTORCH_VERSION` from `source/Dockerfile` and writes them to `$GITHUB_OUTPUT`
- [x] 2.2 Add a guard assertion that fails the step if any extracted version is empty

## 3. Update build-push step

- [x] 3.1 Remove `CUDA_VERSION` and `CUDNN_VERSION` from the `build-args` in the `Build and Push the Docker Image` step
- [x] 3.2 Replace `${{ matrix.comfyui_version }}`, `${{ matrix.comfyui_manager_version }}`, `${{ matrix.pytorch_version }}` references with the step output variables from the extraction step

## 4. Simplify metadata tags

- [x] 4.1 Remove all tag patterns containing `cuda` or `cudnn` from the `Create Tags & Labels` step
- [x] 4.2 Remove the `flavor: latest=${{ matrix.is_default }}` condition and set `latest=true` unconditionally
- [x] 4.3 Remove all `enable=${{ matrix.is_default }}` conditions from remaining tag patterns
- [x] 4.4 Verify remaining tags: `sha`, major (non-v0), major.minor, semver, `comfyui-<V>` variants, `pytorch-<P>` variants

## 5. Update labels and action versions

- [x] 5.1 Update `org.opencontainers.image.authors` label from `David Neumann <david.neumann@lecode.de>` to `LoonWorks`
- [x] 5.2 Correct `actions/checkout@v6` to `actions/checkout@v4`

## 6. Validate and push

- [x] 6.1 Validate the YAML syntax of the updated workflow file (e.g. `python -c "import yaml; yaml.safe_load(open('.github/workflows/build-and-publish.yml'))"`)
- [x] 6.2 Commit the updated workflow to the `main` branch and push to `LoonWorks/comfyui-docker-cpu`
- [x] 6.3 Create and push a test tag to verify the workflow triggers and publishes the image successfully
- [x] 6.3 Create and push a test tag to verify the workflow triggers and publishes the image successfully
