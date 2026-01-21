#
# Dockerfile for DeGirum AI server with Axelera runtime
#
# Copyright DeGirum Corp 2025
#

FROM ubuntu:22.04

ARG DG_CUSTOM_WHEEL

#ARG AXELERA_VERSION

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

# Install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
RUN if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo

# Extract Axelera version from PySDK installation
# This is all done in a single RUN step to preserve the value of AXELERA_VERSION
RUN echo "Starting Axelera installation" >> /buildLog.txt && \
    degirum install-runtime axelera && \
    # Metis driver installation skipped in docker image
    # Pciutils and kmod are required if metis driver is not installed
    apt install -y pciutils kmod && \
    echo "Axelera installation completed" >> /buildLog.txt
    

# Cleanup / remove unnecessary files
RUN rm -f /degirum/Dockerfile /degirum/README.md /degirum/*.deb && \
    rm -rf /var/lib/apt/lists/* && \
    pip cache purge

# run degirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo", "--protocol", "both" ]
