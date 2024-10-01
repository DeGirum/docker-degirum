#
# Dockerfile for DeGirum AI server with all runtimes
#
# Copyright DeGirum Corp 2024
#

# Base image: for ARM64 should be ubuntu:22.04, for x86 should be empty
ARG BASE_IMAGE
# If the BASE_IMAGE variable is not set or is empty, the default value openvino/ubuntu22_runtime:2024.4.0 will be used
FROM ${BASE_IMAGE:-openvino/ubuntu22_runtime:2024.4.0}
# PySDK wheel file name: either not defined or Python 3.10 PySDK wheel
ARG DG_CUSTOM_WHEEL

# After every FROM statement, all the ARGs get collected and are no longer available
# Docker ARG variables need to be redefined in each stage of the Dockerfile if they are used after a FROM instruction.
ARG BASE_IMAGE

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

# Install RKNN Runtime. Only on aarch64
RUN \
    if [ -z "${BASE_IMAGE}" ] ; then \
        echo "BASE_IMAGE is not set, skipping RKNN Runtime installation."; \
    else \
        apt update && apt install -y git && \
        git clone https://github.com/Pelochus/ezrknn-toolkit2 && \
        cd ezrknn-toolkit2 && \
        git checkout 99db9e5b950ccc7e0d1ee20a16b92f7d8b6e60e6 && \
        cp ./rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/ && \
        cd .. && \
        apt remove -y git && \
        rm -rf ezrknn-toolkit2; \
    fi

# NPU Driver installation
# Only performed on TARGETARCH=amd64
RUN if [ -z "${BASE_IMAGE}" ] ; then \
    echo "Base Image not set, so x86_64 system, starting NPU driver installation." >> /buildLog.txt; \
    apt update && apt install -y wget udev && \
    wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-driver-compiler-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb && \
    wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-fw-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb && \
    wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-level-zero-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb; \
    apt update && apt install -y libtbb12; \
    dpkg -i intel-driver-compiler-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt-get install -f; \
    dpkg -i intel-fw-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt-get install -f; \
    dpkg -i intel-level-zero-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt-get install -f; \
    if dpkg -l | grep -q level-zero; then \
        echo "level-zero already installed." >> /buildLog.txt; \
    else \
        wget https://github.com/oneapi-src/level-zero/releases/download/v1.17.6/level-zero_1.17.6+u22.04_amd64.deb && \
        wget https://github.com/oneapi-src/level-zero/releases/download/v1.17.6/level-zero-devel_1.17.6+u22.04_amd64.deb && \
        dpkg -i level-zero*.deb && \
        echo "Fetched and installed level-zero from GitHub." >> /buildLog.txt; \
    fi; \
else echo "BASE_IMAGE is not set, skipping NPU Drvier installation." >> /buildLog.txt; fi


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
