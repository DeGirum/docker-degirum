#
# Dockerfile for DeGirum AI server with Qualcomm runtime
#
# Copyright DeGirum Corp 2026
#

FROM ubuntu:22.04

ARG DG_CUSTOM_WHEEL

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
    libglib2.0-0 \
    git \
    uuid-runtime \
    unzip \
    libatomic1

# Install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
RUN if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo

# Install Qualcomm runtime using degirum install-runtime command and required system packages
RUN echo "Starting Qualcomm installation" >> /buildLog.txt && \
    degirum install-runtime qualcomm && \
    echo "Qualcomm installation completed" >> /buildLog.txt

# Install Qualcomm DSP binaries
RUN HDB_DIR="/tmp/hdb-$(uuidgen)" && \
    git clone --filter=blob:none --sparse https://github.com/linux-msm/hexagon-dsp-binaries.git "${HDB_DIR}" && \
    cd "${HDB_DIR}" && git checkout f9e594abf41eb1ecfc468d7c970317b5f95328f9 && \
    git sparse-checkout set qcs6490/radxa/dragon-q6a/CDSP.HT.2.5.c4-00004-KODIAK-1 && \
    mkdir -p /usr/lib/dsp/cdsp && \
    cp -a qcs6490/radxa/dragon-q6a/CDSP.HT.2.5.c4-00004-KODIAK-1/. /usr/lib/dsp/cdsp/ && \
    rm -rf "${HDB_DIR}"

# Install Qualcomm DSP runtime library
RUN set -eux; \
    KVER="$(curl -fsSL https://github.com/radxa-pkg/radxa-archive-keyring/releases/latest/download/VERSION)"; \
    curl -fsSL -o /tmp/keyring.deb \
      "https://github.com/radxa-pkg/radxa-archive-keyring/releases/latest/download/radxa-archive-keyring_${KVER}_all.deb"; \
    dpkg -i /tmp/keyring.deb; \
    echo "deb [signed-by=/usr/share/keyrings/radxa-archive-keyring.gpg] https://radxa-repo.github.io/qcs6490-noble qcs6490-noble main" \
      > /etc/apt/sources.list.d/radxa-qcs6490.list; \
    apt update; \
    apt install -y --no-install-recommends libcdsprpc1=1.0.7-3; \
    rm -rf /tmp/keyring.deb && \
    ln -s libcdsprpc.so.1 /usr/lib/aarch64-linux-gnu/libcdsprpc.so && ldconfig

# Cleanup / remove unnecessary files
RUN rm -f /degirum/Dockerfile /degirum/README.md /degirum/*.deb && \
    rm -rf /var/lib/apt/lists/* && \
    pip cache purge

# run degirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo", "--protocol", "both" ]
