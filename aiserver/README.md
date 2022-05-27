# About This Repo

This is the Git repo of the official Docker image for DeGirum AI Server.

The image is based on [python:3.9-slim](https://hub.docker.com/_/python) image and contains [DeGirum PySDK](https://degirum.github.io/simple/degirum/index.html) installation.
Once the docker container is started, DeGirum AI server is launched and accepts client connections on port **8778**.
The image does not contain any AI models. The model zoo directory must be supplied as the [bind mount](https://docs.docker.com/storage/bind-mounts/).

## How to Build and Use Official AI Server Docker Image

Pre-requisites: [Docker Desktop](https://www.docker.com/get-started/) is installed on the **docker host**: the computer where you want 
to build or run the docker container.

DeGirum provides pre-built [Docker container image on DockerHub](https://hub.docker.com/r/degirum/aiserver), so you can run it right away.
But if you want to **build** the Docker container image yourself, execute the following commands:
    
    git clone https://github.com/DeGirum/docker-degirum
    cd docker-degirum/aiserver
    docker build . -t degirum/aiserver:latest

To **run** the Docker container (either you build it yourself or you download it from DockerHub), execute the following command:

    docker run -d -p 8778:8778 -v /my/model/zoo/dir:/zoo --privileged degirum/aiserver:latest

Here `/my/model/zoo/dir` is the local path on the docker host computer to the model zoo directory to be served by AI server.

`--privileged` parameter is required so the PySDK server can have write access to `/sys/bus/pci` directories.

You connect to the AI server by providing the IP address or the host name of the docker host computer when connecting the the model zoo:

    import degirum as dg
    zoo = dg.connect_model_zoo(docker_host_IP)

## How to Build and Use AI Server Docker Image with Embedded Zoo

If you want to include the model zoo directory directly into the docker image, create the following `Dockerfile` in some empty directory on your docker host computer:

    FROM degirum/aiserver:latest
    COPY /my/model/zoo/dir /zoo/

Here `/my/model/zoo/dir` is the local path on the docker host computer to the model zoo directory you want to embed into the Docker image.

Then build the docker image the by executing the following command having that directory which contains the `Dockerfile` as the current working directory:
    
    docker build . -t your-custom-tag
    
Here `your-custom-tag` is the tag to assign to the image to be created.
    
To start the docker container built this way, execute the following command (omiting bind mount parameter):

    docker run -d -p 8778:8778 --privileged your-custom-tag

    
