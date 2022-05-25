# About This Repo

This is the Git repo of the official Docker image for DeGirum AI Client.

The image is based on [python:3.9-slim](https://hub.docker.com/_/python) image and contains [DeGirum PySDK](https://degirum.github.io/simple/degirum/index.html) and [JupyterLab](https://jupyter.org/install) packages.
All additional Python packages to be installed at the image creation process are listed in [requirements.txt](https://github.com/DeGirum/docker-degirum/blob/master/aiclient/requirements.txt).

Once the Docker container is started, JupyterLab accepts client connections on port **8888**.
`JUPYTER_TOKEN` environment variable is used to define JupyterLab token value.
`/degirum` directory inside the container serves as JupyterLab root directory and contains [DeGirum PySDK examples](https://github.com/DeGirum/PySDKExamples).

## How to Build and Use Official AI Client Docker Image

Pre-requisites: [Docker Desktop](https://www.docker.com/get-started/) is installed on the **docker host**: the computer where you want 
to build or run the docker container.

To **build** the Docker container image, execute the following command:

    docker build . -t degirum/aiclient:latest

To **start** the Docker container, execute the following command:

    docker run -it --rm -p 8080:8888 -v /my/work/dir:/degirum -e JUPYTER_TOKEN="token" degirum/aiclient:latest

Here you specify `/my/work/dir` and `"token"` parameters, described just below.

You can open http://docker_host_IP:8080 URL in your browser to access JupyterLab
(here `docker_host_IP` is the IP address or the host name of the docker host computer).

The `"token"` parameter serves as your login credentials to JupyterLab.

The `/my/work/dir` parameter is the docker host computer local directory, which is used as a JupyterLab root directory.
