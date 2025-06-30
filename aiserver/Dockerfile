#
# Dockerfile for DeGirum AI server with all runtimes
#
# Copyright DeGirum Corp 2024
#

# Base image: for ARM64 should be ubuntu:22.04, for x86 should be empty
# If the BASE_IMAGE variable is not set or is empty, the default value openvino/ubuntu22_runtime:2024.6.0 will be used
ARG BASE_IMAGE
# HailoRT deb file name
ARG HAILORT_DEB
# DXRT deb file name
ARG LIBDXRT_DEB
# ONNX Runtime version, default to 1.19.0
ARG ONNX_RUNTIME_VERSION=1.19.0

FROM ${BASE_IMAGE:-openvino/ubuntu22_runtime:2024.6.0}
# PySDK wheel file name: either not defined or Python 3.10 PySDK wheel
ARG DG_CUSTOM_WHEEL
ARG ONNX_RUNTIME_VERSION

# After every FROM statement, all the ARGs get collected and are no longer available
# Docker ARG variables need to be redefined in each stage of the Dockerfile if they are used after a FROM instruction.
ARG BASE_IMAGE

# HailoRT deb file name, default to dummy.deb
ARG HAILORT_DEB=dummy.deb

# DXRT deb file name, default to dummy.deb
ARG LIBDXRT_DEB=dummy.deb

EXPOSE 8778-8779
WORKDIR /degirum
USER root

# Common preparation steps
RUN apt update && apt install -y --no-install-recommends \
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
RUN echo "Installing ONNX Runtime version ${ONNX_RUNTIME_VERSION}" >> /buildLog.txt && \
    if [ -z "${BASE_IMAGE}" ] ; then \
        arch_suff=x64 ; \
        echo "Detected architecture: x86_64" >> /buildLog.txt; \
    else \
        arch_suff=aarch64 ; \
        echo "Detected architecture: ARM64" >> /buildLog.txt; \
    fi && \
    curl -L -O https://github.com/microsoft/onnxruntime/releases/download/v${ONNX_RUNTIME_VERSION}/onnxruntime-linux-$arch_suff-${ONNX_RUNTIME_VERSION}.tgz && \
    tar -xzf onnxruntime-linux-$arch_suff-${ONNX_RUNTIME_VERSION}.tgz && \
    mv onnxruntime-linux-$arch_suff-${ONNX_RUNTIME_VERSION} /usr/local && \
    rm onnxruntime-linux-$arch_suff-${ONNX_RUNTIME_VERSION}.tgz && \
    echo "ONNX Runtime installation completed" >> /buildLog.txt

# Install Coral Edge TPU drivers and runtime
RUN echo "Installing Coral Edge TPU drivers and runtime" >> /buildLog.txt && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor | tee /etc/apt/keyrings/coral-edgetpu.gpg >/dev/null && \
    echo "deb [signed-by=/etc/apt/keyrings/coral-edgetpu.gpg] https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list && \
    apt update && \
    apt install -y libedgetpu-dev && \
    echo "Coral Edge TPU drivers and runtime installation completed" >> /buildLog.txt

# Install RKNN Runtime. Only on aarch64
RUN echo "Installing RKNN Runtime if applicable" >> /buildLog.txt && \
    if [ -z "${BASE_IMAGE}" ] ; then \
        echo "BASE_IMAGE is not set, skipping RKNN Runtime installation." >> /buildLog.txt; \
    else \
        echo "BASE_IMAGE is set, proceeding with RKNN Runtime installation." >> /buildLog.txt && \
        apt install -y git && \
        git clone https://github.com/Pelochus/ezrknn-toolkit2 && \
        cd ezrknn-toolkit2 && \
        git checkout 99db9e5b950ccc7e0d1ee20a16b92f7d8b6e60e6 && \
        cp ./rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/ && \
        cd .. && \
        apt remove -y git && \
        rm -rf ezrknn-toolkit2 && \
        echo "RKNN Runtime installation completed" >> /buildLog.txt; \
    fi

# Install Akida Runtime
RUN pip install --upgrade pip && \
    pip install akida==2.11.0

# Add Akida runtime to the LD_LIBRARY_PATH
RUN export AKIDA_PATH=$(python3 -c "import akida, os; print(os.path.dirname(akida.__file__))") && \
    echo $AKIDA_PATH > /etc/ld.so.conf.d/akida.conf && \
    ldconfig && \
    export LD_LIBRARY_PATH=$AKIDA_PATH:$LD_LIBRARY_PATH && \
    echo "Akida Runtime added to LD_LIBRARY_PATH: $LD_LIBRARY_PATH" >> /buildLog.txt

# NPU Driver installation
# Only performed on TARGETARCH=amd64
RUN echo "Starting NPU Driver installation if applicable" >> /buildLog.txt && \
    if [ -z "${BASE_IMAGE}" ] ; then \
        echo "Base Image not set, so x86_64 system, starting NPU driver installation." >> /buildLog.txt; \
        apt install -y wget udev && \
        wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-driver-compiler-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb && \
        wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-fw-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb && \
        wget https://github.com/intel/linux-npu-driver/releases/download/v1.5.1/intel-level-zero-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb && \
        apt install -y libtbb12 && \
        dpkg -i intel-driver-compiler-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt install -f -y && \
        dpkg -i intel-fw-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt install -f -y && \
        dpkg -i intel-level-zero-npu_1.5.1.20240708-9842236399_ubuntu22.04_amd64.deb || apt install -f -y && \
        if dpkg -l | grep -q level-zero; then \
            echo "level-zero already installed." >> /buildLog.txt; \
        else \
            wget https://github.com/oneapi-src/level-zero/releases/download/v1.17.6/level-zero_1.17.6+u22.04_amd64.deb && \
            wget https://github.com/oneapi-src/level-zero/releases/download/v1.17.6/level-zero-devel_1.17.6+u22.04_amd64.deb && \
            dpkg -i level-zero*.deb && \
            echo "Fetched and installed level-zero from GitHub." >> /buildLog.txt; \
        fi; \
    else \
        echo "BASE_IMAGE is set, skipping NPU Driver installation." >> /buildLog.txt; \
    fi

# MemryX RT installation
RUN apt update && apt install -y linux-headers-generic && \
    curl https://developer.memryx.com/deb/memryx.asc | tee /etc/apt/keyrings/memryx.asc >/dev/null && \
    echo 'deb [signed-by=/etc/apt/keyrings/memryx.asc] https://developer.memryx.com/deb stable main' | tee /etc/apt/sources.list.d/memryx.list >/dev/null && \
    apt update 

RUN --mount=type=tmpfs,target=/sys/bus/pci apt install -y memx-drivers=1.2.1-3.1 memx-accl=1.2.3-1


# DXRT installation
# The deb file for DXRT is passed as an optional build argument: LIBDXRT_DEB
# It defaults to dummy.deb.

# Copy the DXRT .deb file into the Docker image
COPY ${LIBDXRT_DEB} /tmp/libdxrt.deb

# DXRT installation
RUN echo "Starting DXRT installation if applicable" >> /buildLog.txt && \
    if [ "${LIBDXRT_DEB}" != "dummy.deb" ]; then \
        echo "DXRT deb specified, proceeding with installation" >> /buildLog.txt; \
        apt-get update && apt-get install -y unzip git cmake ninja-build python3-dev python3-pybind11 wget && \
        dpkg --unpack /tmp/libdxrt.deb && \
        DEBIAN_FRONTEND=noninteractive dpkg --configure -a --debug=2 || true && \
        echo "DXRT installation completed" >> /buildLog.txt; \
    else \
        echo "DXRT deb not specified, skipping installation" >> /buildLog.txt; \
    fi


# HailoRT installation
# The deb file for HailoRT is passed as an optional build argument: HAILORT_DEB
# It defaults to dummy.deb.

# Copy the HailoRT .deb file into the Docker image
COPY ${HAILORT_DEB} /tmp/hailort.deb


# HailoRT installation
RUN echo "Starting HailoRT installation if applicable" >> /buildLog.txt && \
    if [ "${HAILORT_DEB}" != "dummy.deb" ]; then \
        touch /.dockerenv && \
        echo "HailoRT deb specified, proceeding with installation" >> /buildLog.txt; \
        dpkg --unpack /tmp/hailort.deb && \
        DEBIAN_FRONTEND=noninteractive dpkg --configure -a || true && \
        echo "HailoRT installation completed" >> /buildLog.txt; \
    else \
        echo "HailoRT deb not specified, skipping installation" >> /buildLog.txt; \
    fi


# Install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
RUN if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo
    
# Cleanup / remove unnecessary files
RUN rm -f /degirum/Dockerfile /degirum/README.md /degirum/*.deb && \
    rm -rf /var/lib/apt/lists/* \
            /usr/share/libdxrt/src && \
    pip cache purge


# Run DeGirum AI server.
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo", "--protocol", "both" ]
