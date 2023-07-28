# About This Repo

This is the repository of the official Docker image for the DeGirum AI Server running the ONNX Runtime. This Docker image is designed to run on systems with Ubuntu 22.04 and facilitates CPU inference via ONNX.

The image is based on the Ubuntu 22.04 image and includes the [DeGirum PySDK](https://docs.degirum.com/content/pysdk/).
Once the Docker container is initiated, the DeGirum AI server commences and begins to accept client connections on port **8778**.
The image doesn't contain any AI models. The model zoo directory must be provided as the [bind mount](https://docs.docker.com/storage/bind-mounts/).

### NOTE: This Docker image uses ONNX Runtime version 1.15.1

## How to Build and Use Official AI Server Docker Image

Pre-requisites: [Docker Desktop](https://www.docker.com/get-started/) or [Docker Engine](https://docs.docker.com/engine/install/) must be installed on the **Docker host**: the machine where you aim to build or run the Docker container.

DeGirum provides a pre-built [Docker container image on DockerHub](https://hub.docker.com/r/degirum/aiserveronnx), enabling immediate usage. To **run** the Docker container (either the one you built yourself (see below) or the one from DockerHub), run the following command:

```bash
docker run -d -p 8778:8778 -v /my/model/zoo/dir:/zoo --privileged degirum/aiserveronnx:latest
```

Here /my/model/zoo/dir is the local path on the Docker host machine to the model zoo directory to be utilized by the AI server. This parameter can be omitted. If so, no models will be served from the local zoo, but models from cloud zoos can still be served.

The --privileged parameter is necessary so that the PySDK server can have write access to /sys/bus/pci directories.

To connect to the AI server, provide the IP address or the host name of the Docker host machine when connecting to the model zoo:

    import degirum as dg
    zoo = dg.connect_model_zoo(docker_host_IP)

If you want to **build** the Docker container image yourself instead, execute the following commands:

    git clone https://github.com/DeGirum/docker-degirum
    cd docker-degirum/aiserveronnx
    docker build . -t degirum/aiserveronnx:latest


## How to Build and Use AI Server Docker Image with Embedded Zoo

If you want to include the model zoo directory directly into the Docker image, create the following `Dockerfile` in some empty directory on your Docker host computer:

    FROM degirum/aiserveronnx:latest
    COPY /my/model/zoo/dir /zoo/

Here `/my/model/zoo/dir` is the local path on the Docker host computer to the model zoo directory you want to embed into the Docker image.

Then build the Docker image the by executing the following command having that directory which contains the `Dockerfile` as the current working directory:
    
    docker build . -t your-custom-tag
    
Here `your-custom-tag` is the tag to assign to the image to be created.
    
To start the Docker container built this way, execute the following command (omitting the bind mount parameter):

    docker run -d -p 8778:8778 --privileged your-custom-tag

