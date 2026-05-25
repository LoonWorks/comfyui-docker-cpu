
# Defines build arguments for the versions of ComfyUI, ComfyUI Manager, and PyTorch to use
ARG COMFYUI_VERSION=0.22.2
ARG COMFYUI_MANAGER_VERSION=4.2.1
ARG PYTORCH_VERSION=2.9.1

# This image is based on the official Python 3.13 slim image for a minimal, CPU-only build
FROM python:3.13-slim

# Re-declares build arguments after FROM so they are available in RUN instructions
ARG COMFYUI_VERSION
ARG COMFYUI_MANAGER_VERSION
ARG PYTORCH_VERSION

# Installs system packages required by ComfyUI and its dependencies
RUN apt-get update --assume-yes && \
    apt-get install --assume-yes \
        git \
        sudo \
        libgl1 \
        libglib2.0-0 && \
    rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

# Creates a Python virtual environment so that all packages are isolated from the system Python installation,
# and sets it as the default interpreter for all subsequent build steps and runtime processes
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Clones the ComfyUI repository and checks out the configured release
RUN git clone https://github.com/Comfy-Org/ComfyUI.git /opt/comfyui && \
    cd /opt/comfyui && \
    git checkout "v${COMFYUI_VERSION}"

# Clones the ComfyUI Manager repository and checks out the configured release; ComfyUI Manager is an extension for ComfyUI that enables users to install
# custom nodes and download models directly from the ComfyUI interface; instead of installing it to "/opt/comfyui/custom_nodes/comfyui-manager", which
# is the directory it is meant to be installed in, it is installed to its own directory; the entrypoint will symlink the directory to the correct
# location upon startup; the reason for this is that the ComfyUI Manager must be installed in the same directory that it installs custom nodes to, but
# this directory is mounted as a volume, so that the custom nodes are not installed inside of the container and are not lost when the container is
# removed; this way, the custom nodes are installed on the host machine
RUN git clone https://github.com/Comfy-Org/ComfyUI-Manager.git /opt/comfyui-manager && \
    cd /opt/comfyui-manager && \
    git checkout ${COMFYUI_MANAGER_VERSION}

# Installs CPU-only PyTorch from the official CPU wheel index, then installs the required Python packages for
# both ComfyUI and the ComfyUI Manager; using the CPU index URL ensures no CUDA-capable wheels are ever resolved
RUN pip install \
        torch==${PYTORCH_VERSION} \
        torchvision \
        torchaudio \
        --index-url https://download.pytorch.org/whl/cpu && \
    pip install \
        --requirement /opt/comfyui/requirements.txt \
        --requirement /opt/comfyui-manager/requirements.txt

# Sets the working directory to the ComfyUI directory
WORKDIR /opt/comfyui

# Exposes the default port of ComfyUI (this is not actually exposing the port to the host machine, but it is good practice to include it as metadata,
# so that the user knows which port to publish)
EXPOSE 8188

# Adds the startup script to the container; the startup script will create all necessary directories in the models and custom nodes volumes that were
# mounted to the container and symlink the ComfyUI Manager to the correct directory; it will also create a user with the same UID and GID as the user
# that started the container, so that the files created by the container are owned by the user that started the container and not the root user
ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
