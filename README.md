# ComfyUI Docker

This is a Docker image for [ComfyUI](https://www.comfy.org/), which makes it extremely easy to run ComfyUI on Linux and Windows WSL2. The image also includes the [ComfyUI Manager](https://github.com/Comfy-Org/ComfyUI-Manager) extension.

## Getting Started

To get started, you have to install [Docker](https://www.docker.com/). This can be either Docker Engine, which can be installed by following the [Docker Engine Installation Manual](https://docs.docker.com/engine/install/) or Docker Desktop, which can be installed by [downloading the installer](https://www.docker.com/products/docker-desktop/) for your operating system. If you want to use Docker Compose to run the ComfyUI Docker container, then Docker Compose must also be installed. Docker Desktop comes with Docker Compose pre-installed. If you are using Docker Engine, you may already have Docker Compose installed. You can check this by running `docker compose version` in your terminal. If Docker Compose is not installed, you can follow the [Docker Compose Installation Manual](https://docs.docker.com/compose/install/) to install it.

This is a CPU-only image. No GPU or NVIDIA runtime is required.

## Installation & Running

### Installing & Running Using Docker Run

The ComfyUI Docker image is available from the [GitHub Container Registry](https://ghcr.io/loonworks/comfyui-docker-cpu). Installing ComfyUI is as simple as pulling the image and starting a container, which can be achieved using the following command:

```shell
docker run \
    --name comfyui \
    --detach \
    --restart unless-stopped \
    --env USER_ID="$(id -u)" \
    --env GROUP_ID="$(id -g)" \
    --volume "<path/to/models/folder>:/opt/comfyui/models:rw" \
    --volume "<path/to/custom/nodes/folder>:/opt/comfyui/custom_nodes:rw" \
    --volume "<path/to/output/folder>:/opt/comfyui/output:rw" \
    --publish 8188:8188 \
    ghcr.io/loonworks/comfyui-docker-cpu:latest
```

For a full list of the available image tags, please refer to the [image tags](#available-image-tags) section. Please note, that the `<path/to/models/folder>`, `<path/to/custom/nodes/folder>` and `<path/to/output/folder>` must be replaced with paths to directories on the host system where the models, custom nodes and generated outputs (images, workflows, etc.) will be stored, e.g., `$HOME/.comfyui/models`, `$HOME/.comfyui/custom-nodes` and `$HOME/.comfyui/output`, which can be created like so: `mkdir -p $HOME/.comfyui/{models,custom-nodes,output}`.

> [!WARNING]
> If you are coming from a version prior to v0.6.0, please note that the output directory mapping was added in version v0.6.0. If the Docker container for the previous version is still available, you can migrate your existing outputs by copying them from the old container to the host system. Assuming your previous container was named `comfyui`, you can use the following command:
>
> ```shell
> docker cp comfyui:/opt/comfyui/output/. <path/to/output/folder>
> ```

The `--detach` flag causes the container to run in the background and `--restart unless-stopped` configures the Docker Engine to automatically restart the container if it stopped itself, experienced an error, or the computer was shutdown, unless you explicitly stopped the container using `docker stop`. This means that ComfyUI will be automatically started in the background when you boot your computer. The two `--env` arguments inject the user ID and group ID of the current host user into the container. During startup, a user with the same user ID and group ID will be created, and ComfyUI will be run using this user. This ensures that files written to the volumes (e.g., models, custom nodes installed with the ComfyUI Manager, and outputs) will be owned by the host system's user. Normally, the user inside the container is `root`, which means that the files that are written from the container to the host system are also owned by `root`. If you have run ComfyUI Docker without setting the environment variables, then you may have to change the owner of the files in the models and custom nodes directories: `sudo chown -r "$(id -un):$(id -gn)" <path/to/models/folder> <path/to/custom/nodes/folder> <path/to/output/folder>`.

During startup, ComfyUI Manager is mounted into the custom nodes directory at `/opt/comfyui/custom_nodes/comfyui-manager`, which matches the upstream recommended installation path. If your custom nodes volume still contains a legacy `ComfyUI-Manager` folder from older versions, remove it to avoid duplicate Manager installs.

After the container has started, you can navigate to [localhost:8188](http://localhost:8188) to access ComfyUI.

If you want to pass additional command line arguments to ComfyUI, you can do so by specifying them as the command when starting the container. For example, if you want to allow external web apps to connect to ComfyUI in the container, you can add the `--enable-cors-header` argument like so:

```shell
docker run \
    --name comfyui \
    --detach \
    --restart unless-stopped \
    --env USER_ID="$(id -u)" \
    --env GROUP_ID="$(id -g)" \
    --volume "<path/to/models/folder>:/opt/comfyui/models:rw" \
    --volume "<path/to/custom/nodes/folder>:/opt/comfyui/custom_nodes:rw" \
    --volume "<path/to/output/folder>:/opt/comfyui/output:rw" \
    --publish 8188:8188 \
    ghcr.io/loonworks/comfyui-docker-cpu:latest \
    --enable-cors-header <origin>
```

You can stop ComfyUI Docker using the following command:

```shell
docker stop comfyui
```

This will keep the container on your system, so that you can start it again later using:

```shell
docker start comfyui
```

To completely remove the container from your system, you can use the following command:

```shell
docker rm comfyui
```

> [!WARNING]
> While the custom nodes themselves are installed outside of the container, their requirements are installed inside of the container. Also, in versions prior to v0.6.0, the outputs of ComfyUI were stored inside of the container. This means that removing the container will remove the installed requirements and all outputs you have not saved manually. When the container is started again, the requirements will be automatically installed, but this may, depending on the number of custom nodes and their requirements, take some time.

### Installing & Running Using Docker Compose

Instead of using `docker run`, you can use the provided [`compose.yml`](compose.yml) in this repository to make running, managing and updating ComfyUI Docker easier.

Models, custom nodes, and outputs are stored on the host system, by default in the `$HOME/.comfyui/models`, `$HOME/.comfyui/custom-nodes`, and `$HOME/.comfyui/output` directories, respectively. If you are running ComfyUI Docker for the first time, you have to create the required host directories for models, custom nodes, and outputs. You can do this by running:

```shell
mkdir -p $HOME/.comfyui/{models,custom-nodes,output}
```

You can change the paths to these directories using the `MODELS_PATH`, `CUSTOM_NODES_PATH`, and `OUTPUT_PATH` environment variables, respectively. To permanently set these environment variables, you can create a `.env` file alongside the [`compose.yml`](compose.yml) file with the following content:

```shell
MODELS_PATH=<path/to/models/folder>
CUSTOM_NODES_PATH=<path/to/custom/nodes/folder>
OUTPUT_PATH=<path/to/output/folder>
```

The Dockerfile uses the ComfyUI Docker image with the `latest` tag. A different tag can be specified using the `IMAGE_TAG` environment variable. Again, the image tag can be set permanently in a `.env` file. For a full list of the available image tags, please refer to the [image tags](#available-image-tags) section. Assuming you have already created a `.env` file, you can run the following command to use the `0.6.1` tag by appending the `IMAGE_TAG` variable to the existing `.env` file:

```shell
echo "IMAGE_TAG=0.6.1" >> .env
```

Normally, the user inside a Docker container is `root`, which means that the files that are written from the container to the host system are also owned by `root`. To avoid this, ComfyUI Docker creates a new user inside the container. By default, running the Docker Compose file will create a user and group with the IDs `1000` and `1000`, respectively. Most Linux systems have the first user created with these IDs. If your user has different IDs, you can change them using the `USER_ID` and `GROUP_ID` environment variables. To permanently set these environment variables, you may again use a `.env` file alongside the `compose.yml` file to specify the correct IDs. You can find out your user and group IDs by running `id -u` and `id -g` in your terminal. Assuming you have already created a `.env` file, you can run the following commands to append the user and group IDs to the existing `.env` file:

```shell
echo "USER_ID=$(id -u)" >> .env
echo "GROUP_ID=$(id -g)" >> .env
```

After this, you can start ComfyUI Docker using Docker Compose:

```shell
docker compose up --detach
```

If the ComfyUI Docker image is not available locally, it will be pulled automatically. This will start ComfyUI in the background. The --detach flag causes the container to run in the background. You can then navigate to [localhost:8188](http://localhost:8188) to access ComfyUI.

To view the logs of the running container, you can use the following command:

```shell
docker compose logs --follow
```

The `--follow` flag will continuously stream the logs to your terminal. Omitting this flag will show you the existing logs and then exit. To stop ComfyUI, you can use the following command:

```shell
docker compose down
```

This will stop the container and remove it from your system. If you need to run ComfyUI with custom arguments using Docker Compose, you can do so by adding a `command` section to the ComfyUI service in the `compose.yml` file. For example, to allow external web apps to connect to ComfyUI in the container, you can add the `--enable-cors-header` argument:

```yaml
command: ["--enable-cors-header", "*"]
```

To apply the changes, you have to recreate the container:

```shell
docker compose up --detach --force-recreate
```

## Available Image Tags

The ComfyUI Docker image is available with different tags. The available tags are:

- `latest`: The latest stable version of ComfyUI Docker (CPU-only). This will use the latest versions of ComfyUI, ComfyUI Manager, and PyTorch (CPU) available at the time of the image build.
- `0.6`, `0.6.1`: These tags will always use the specific version of ComfyUI Docker, and the latest versions of ComfyUI, ComfyUI Manager and PyTorch (CPU) available at the time of the image build.
- `0.6-comfyui-0.8.2`, `0.6.1-comfyui-0.8.2`: These tags will always use the specific versions of ComfyUI Docker and ComfyUI, and the latest versions of ComfyUI Manager and PyTorch (CPU) available at the time of the image build.
- `0.6-comfyui-0.8.2-comfyui-manager-4.0.5`, `0.6.0-comfyui-0.8.2-comfyui-manager-4.0.5`: These tags will always use the specific versions of ComfyUI Docker, ComfyUI, and ComfyUI Manager, and the latest version of PyTorch (CPU) available at the time of the image build.
- `0.6-comfyui-0.8.2-comfyui-manager-4.0.5-pytorch-2.9.1`, `0.6.1-comfyui-0.8.2-comfyui-manager-4.0.5-pytorch-2.9.1`: These tags will always use the specific versions of ComfyUI Docker, ComfyUI, ComfyUI Manager, and PyTorch.
- `sha-<short-commit-sha>`: These tags point to ComfyUI Docker that were built from a specific commit. They will always use the versions of ComfyUI Docker, ComfyUI, ComfyUI Manager, and PyTorch that were the most recent at the time of the image build.

## Updating

### Updating Using Docker Run

To update ComfyUI Docker to the latest version you have to first stop the running container, then pull the new version, optionally remove dangling images, and then restart the container:

```shell
docker stop comfyui
docker rm comfyui

docker pull ghcr.io/loonworks/comfyui-docker-cpu:latest
docker image prune # Optionally remove dangling images

docker run \
    --name comfyui \
    --detach \
    --restart unless-stopped \
    --env USER_ID="$(id -u)" \
    --env GROUP_ID="$(id -g)" \
    --volume "<path/to/models/folder>:/opt/comfyui/models:rw" \
    --volume "<path/to/custom/nodes/folder>:/opt/comfyui/custom_nodes:rw" \
    --volume "<path/to/output/folder>:/opt/comfyui/output:rw" \
    --publish 8188:8188 \
    ghcr.io/loonworks/comfyui-docker-cpu:latest
```

### Updating Using Docker Compose

To update ComfyUI Docker using Docker Compose, you have to stop the service first, then pull the new images, and restart it like so:

```shell
docker compose down
docker compose pull
docker compose up --detach
```

## Building

If you want to use the bleeding edge development version of the Docker image, you can also clone the repository and build the image yourself:

```shell
git clone https://github.com/LoonWorks/comfyui-docker-cpu.git
cd comfyui-docker-cpu
docker build --tag comfyui-docker-cpu:latest source
```

Now, a container can be started like so:

```shell
docker run \
    --name comfyui \
    --detach \
    --restart unless-stopped \
    --env USER_ID="$(id -u)" \
    --env GROUP_ID="$(id -g)" \
    --volume "<path/to/models/folder>:/opt/comfyui/models:rw" \
    --volume "<path/to/custom/nodes/folder>:/opt/comfyui/custom_nodes:rw" \
    --volume "<path/to/output/folder>:/opt/comfyui/output:rw" \
    --publish 8188:8188 \
    comfyui-docker-cpu:latest
```

## License

The ComfyUI Docker image is licensed under the [MIT License](LICENSE). [ComfyUI](https://github.com/comfyanonymous/ComfyUI/blob/master/LICENSE) and the [ComfyUI Manager](https://github.com/ltdrdata/ComfyUI-Manager/blob/main/LICENSE.txt) are both licensed under the GPL 3.0 license.
