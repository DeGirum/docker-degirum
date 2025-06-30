# About This Repo

This is the repository of the official Docker image for the DeGirum AI Server. This Docker image is designed to run on systems with an Axelera METIS device and contains support for Axelera's low-level runtime.

The image is based on Canonical's [Ubuntu 22.04](https://ubuntu.com/containers) image 
and contains [DeGirum PySDK](https://docs.degirum.com/content/pysdk/).

Once the docker container is started, DeGirum AI server is launched and accepts client connections on port **8778**
using DeGirum proprietary protocol, and on port **8779** using HTTP/WebSockets protocol.

The image does not contain any AI models. The model zoo directory must be supplied as the 
[bind mount](https://docs.docker.com/storage/bind-mounts/) to `/zoo` container-local directory.

## How to Build and Use Official AI Server Docker Image

### Pre-requisites: 

- [Docker Desktop](https://www.docker.com/get-started/) or [Docker Engine](https://docs.docker.com/engine/install/) 
is installed on the **Docker host**: the computer where you want to build or run the Docker container.

- Axelera's Metis driver is installed on the **Docker host**. Follow the Axelera Runtime Installation instructions in the docs.

DeGirum provides a pre-built [Docker container image on DockerHub](https://hub.docker.com/r/degirum/aiservertrt), so 
you can run it right away. To **run** the Docker container (either the one you built yourself (see below) or the one from DockerHub),
execute the following command:

    docker run -d -p 8778-8779:8778-8779 -v /my/model/zoo/dir:/zoo  --privileged degirum/aiserverax:latest

Here `/my/model/zoo/dir` is the local path on the Docker host computer to the model zoo directory to be served by the AI server.
This parameter can be omitted. In this case, no models will be served from the local zoo, but models from cloud zoos 
can still be served.

The `--privileged` parameter is required so that the PySDK server can have write access to `/sys/bus/pci` directories.

You connect to the AI server by providing the IP address or the host name of the Docker host computer when connecting the model zoo:

    import degirum as dg
    zoo = dg.connect(<docker host or IP> [, <cloud zoo URL>, <cloud access token>])

If you want to **build** the Docker container image yourself instead, execute the following commands:

    git clone https://github.com/DeGirum/docker-degirum
    cd docker-degirum/aiserverax
    docker build . -t degirum/aiserverax:latest

## How to Build and Use AI Server Docker Image with Embedded Zoo

If you want to include the model zoo directory directly into the Docker image, create the following `Dockerfile` in some empty directory on your Docker host computer:

    FROM degirum/aiserverax:latest
    COPY /my/model/zoo/dir /zoo/

Here `/my/model/zoo/dir` is the local path on the Docker host computer to the model zoo directory you want to embed into the Docker image.

Then build the Docker image by executing the following command having that directory which contains the `Dockerfile` as the current working directory:
    
    docker build . -t your-custom-tag
    
Here `your-custom-tag` is the tag to assign to the image to be created.
    
To start the Docker container built this way, execute the following command (omitting the bind mount parameter):

    docker run -d -p 8778-8779:8778-8779 --privileged your-custom-tag

This software contains runtime libraries provided by Axelera.
