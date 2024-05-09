#
# Dockerfile for DeGirum AI server with all runtimes
#
# Copyright DeGirum Corp 2024
#

# Base image: for ARM64 should be ubuntu:22.04, for x86 should be empty
ARG BASE_IMAGE
FROM ${BASE_IMAGE:-openvino/ubuntu22_runtime:2023.3.0}

# PySDK wheel file name: either not defined or Python 3.10 PySDK wheel
ARG DG_CUSTOM_WHEEL

EXPOSE 8778-8779
WORKDIR /degirum
USER root

# Common preparation steps
RUN apt-get update && apt-get install -y --no-install-recommends \
    libusb-1.0-0-dev \
    libpython3-dev \
    python3-dev \
    python3-pip \
    python3-numpy \
    ca-certificates \
    curl \
    gpg-agent \
    gpg \
    libgl1 \
    libglib2.0-0

# Install ONNX runtime
RUN \
    if [ -z "${BASE_IMAGE}" ] ; then \
        arch_suff=x64 ; \
    else \
        arch_suff=aarch64 ; \
    fi \
    && curl -L -O https://github.com/microsoft/onnxruntime/releases/download/v1.15.1/onnxruntime-linux-$arch_suff-1.15.1.tgz \
    && tar -xzf onnxruntime-linux-$arch_suff-1.15.1.tgz \
    && mv onnxruntime-linux-$arch_suff-1.15.1 /usr/local \
    && rm onnxruntime-linux-$arch_suff-1.15.1.tgz

# Install Coral Edge TPU drivers and runtime
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
    && echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list \
    && apt-get update \
    && apt-get install -y libedgetpu-dev

# Install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
RUN if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo \
    && rm -rf /var/lib/apt/lists/*

# Run DeGirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo", "--protocol", "both" ]
