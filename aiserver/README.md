# About This Repo

This is the Git repo of the official Docker image for DeGirum AI Server.

The image contains [DeGirum PySDK](https://docs.degirum.com/content/pysdk/) installation with the following AI runtimes installed:
- DeGirum N2X runtime for DeGirum Orca devices
- Intel&reg; OpenVINO&trade; runtime for CPU, GPU, and NPU devices
- Google TensorFlow Lite runtime for CPU and Coral Edge TPU devices
- ONNX runtime for CPU
- Rockchip RKNN Runtime for RKNPU devices

Once the docker container is started, DeGirum AI server is launched and accepts client connections on port **8778**
using DeGirum proprietary protocol, and on port **8779** using HTTP/WebSockets protocol.

The image does not contain any AI models. The model zoo directory must be supplied as the 
[bind mount](https://docs.docker.com/storage/bind-mounts/) to `/zoo` container-local directory.

## How to Build and Use Official AI Server Docker Image

Pre-requisites: [Docker Desktop](https://www.docker.com/get-started/) or 
[Docker Engine](https://docs.docker.com/engine/install/) is installed on the **docker host**: the computer 
where you want to build or run the docker container.

DeGirum provides pre-built [Docker container image on DockerHub](https://hub.docker.com/r/degirum/aiserver), 
so you can run it right away.

To **run** the Docker container (either the one you built yourself (see below) or the one from DockerHub), 
execute the following command:

    docker run -d -p 8778-8779:8778-8779 -v /my/model/zoo/dir:/zoo --privileged degirum/aiserver:latest

Here `/my/model/zoo/dir` is the local path on the docker host computer to the model zoo directory to be served by AI server. 
This parameter can be omitted. In this case, no models will be served from the local zoo, but models from cloud zoos 
can still be served.

The `--privileged` parameter is required so the PySDK server can have write access to `/sys/bus/pci` directories.

You connect to the AI server by providing the IP address or the host name of the docker host computer when connecting 
the the model zoo:

    import degirum as dg
    zoo = dg.connect(<docker host or IP> [, <cloud zoo URL>, <cloud access token>])

If you want to **build** the Docker container image yourself, execute the following commands:
    
    git clone https://github.com/DeGirum/docker-degirum
    cd docker-degirum/aiserver
    docker build . -t degirum/aiserver:latest

## How to Build and Use AI Server Docker Image with Embedded Zoo

If you want to include the model zoo directory directly into the docker image, create the following `Dockerfile` in some empty directory on your docker host computer:

    FROM degirum/aiserver:latest
    COPY /my/model/zoo/dir /zoo/

Here `/my/model/zoo/dir` is the local path on the docker host computer to the model zoo directory you want to embed into the Docker image.

Then build the docker image the by executing the following command having that directory which contains the `Dockerfile` as the current working directory:
    
    docker build . -t your-custom-tag
    
Here `your-custom-tag` is the tag to assign to the image to be created.
    
To start the docker container built this way, execute the following command (you omit bind mount parameter):

    docker run -d -p 8778-8779:8778-8779 --privileged your-custom-tag

    
#### Building with HailoRT Runtime Support

To build the Docker image with HailoRT runtime support, you need to specify the `HAILORT_DEB` build argument. Place the HailoRT `.deb` file (e.g., `hailort.deb`) next to the Dockerfile and run the following command:

```bash
docker build . -t degirum/aiserver:latest --build-arg HAILORT_DEB=hailort.deb
```

Here, hailort.deb is the path to the HailoRT .deb file that you want installed.

#### Running with Hailo Accelerator Access

If you want the container to have access to the Hailo accelerator, start the container with additional arguments with the following command:

```bash
docker run -d \
  -p 8778-8779:8778-8779 \
  -v ~/HailoModels/:/zoo \
  -v /dev:/dev \
  -v /lib/firmware:/lib/firmware \
  -v /lib/udev/rules.d:/lib/udev/rules.d \
  -v /lib/modules:/lib/modules \
  --device=/dev/hailo0:/dev/hailo0 \
  --privileged \
  degirum/aiserver:latest
