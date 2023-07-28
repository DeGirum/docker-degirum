# Use ubuntu 2204, install onnxruntime 1.15.1, move to required location, install degirum and launch server
FROM ubuntu:22.04 as build-stage

# Install required dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates python3-dev python3-pip python3-numpy \
    g++ make gcc curl \
    && curl -L -O https://github.com/microsoft/onnxruntime/releases/download/v1.15.1/onnxruntime-linux-x64-1.15.1.tgz \
    && tar -xzf onnxruntime-linux-x64-1.15.1.tgz \
    && mv onnxruntime-linux-x64-1.15.1 /usr/local \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get purge -y --auto-remove g++ make gcc curl

ARG DG_CUSTOM_WHEEL

# Set up degirum AI server
WORKDIR /degirum

# copy additional files required for degirum AI server
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./

RUN apt-get update \
    && apt-get install -y libusb-1.0-0-dev libpython3-dev\
    && rm -rf /var/lib/apt/lists/* \
    && pip install --upgrade pip \
    && if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo

# Expose the degirum AI server port
EXPOSE 8778

# Run the degirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo" ]
