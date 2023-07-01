# About This Repo

This is the repository of the official Docker image for the DeGirum AI Server. This Docker image is designed to run on NVIDIA Jetson systems and contains upport for GPU inference via Tensor RT.

The image is based on the [NVIDIA L4T TensorRT](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-tensorrt) image and contains [DeGirum PySDK](https://degirum.github.io/simple/degirum/index.html).
Once the Docker container is started, the DeGirum AI server is launched and starts accepting client connections on port **8778**.
The image does not contain any AI models. The model zoo directory must be supplied as the [bind mount](https://docs.docker.com/storage/bind-mounts/).

## How to Build and Use Official AI Server Docker Image

Pre-requisites: [Docker Desktop](https://www.docker.com/get-started/) or [Docker Engine](https://docs.docker.com/engine/install/) is installed on the **Docker host**: the computer where you want 
to build or run the Docker container.

DeGirum provides a pre-built [Docker container image on DockerHub](https://hub.docker.com/r/degirum/aiservertrt), so you can run it right away.
To **run** the Docker container (either the one you built yourself (see below) or the one from DockerHub), execute the following command:

    docker run -d -p 8778:8778 -v /my/model/zoo/dir:/zoo --runtime nvidia --privileged degirum/aiservertrt:latest

Here `/my/model/zoo/dir` is the local path on the Docker host computer to the model zoo directory to be served by the AI server. This parameter can be omitted. In this case, no models will be served from the local zoo, but models from cloud zoos can still be served.

The `--privileged` parameter is required so that the PySDK server can have write access to `/sys/bus/pci` directories.
The `--runtime nvidia` parameter is required so that the GPU or DLA hardware is accessible to the AI server.

You connect to the AI server by providing the IP address or the host name of the Docker host computer when connecting the the model zoo:

    import degirum as dg
    zoo = dg.connect_model_zoo(docker_host_IP)

If you want to **build** the Docker container image yourself instead, execute the following commands:

    git clone https://github.com/DeGirum/docker-degirum
    cd docker-degirum/aiservertrt
    docker build . -t degirum/aiservertrt:latest

## How to Build and Use AI Server Docker Image with Embedded Zoo

If you want to include the model zoo directory directly into the Docker image, create the following `Dockerfile` in some empty directory on your Docker host computer:

    FROM degirum/aiservertrt:latest
    COPY /my/model/zoo/dir /zoo/

Here `/my/model/zoo/dir` is the local path on the Docker host computer to the model zoo directory you want to embed into the Docker image.

Then build the Docker image the by executing the following command having that directory which contains the `Dockerfile` as the current working directory:
    
    docker build . -t your-custom-tag
    
Here `your-custom-tag` is the tag to assign to the image to be created.
    
To start the Docker container built this way, execute the following command (omitting the bind mount parameter):

    docker run -d -p 8778:8778 --runtime nvidia --privileged your-custom-tag

This software contains source code provided by NVIDIA Corporation.
