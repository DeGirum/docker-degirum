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
    SITE_PACKAGES=$(python3 -c "import sysconfig; print(sysconfig.get_path('purelib'))") && \
    AXELERA_VERSION=$(find "$SITE_PACKAGES" -type f -name 'libaxelera_runtime_agent--*.dgp' \
        | sed -n 's/.*libaxelera_runtime_agent--\([0-9.]*\)\.dgp$/\1/p') && \
# Axelera installation
    if [ -n "$AXELERA_VERSION" ]; then \
        echo "Found AXELERA_VERSION=$AXELERA_VERSION, proceeding with installation" >> /buildLog.txt; \
        GPG_URL="https://software.axelera.ai/artifactory/api/security/keypair/axelera/public" && \
        GPG_KEY_PATH="/etc/apt/keyrings/axelera.gpg" && \
        REPO_LIST_PATH="/etc/apt/sources.list.d/axelera.list" && \
        REPO_SOURCE="https://software.axelera.ai/artifactory/axelera-apt-source/ stable main" && \
        mkdir -p "$(dirname "$GPG_KEY_PATH")" && \
        curl -fsSL "$GPG_URL" | gpg --dearmor | tee "$GPG_KEY_PATH" > /dev/null && \
        chmod 644 "$GPG_KEY_PATH" && \
        echo "deb [signed-by=$GPG_KEY_PATH] $REPO_SOURCE" | tee "$REPO_LIST_PATH" > /dev/null && \
        apt update && \
        # Metis driver installation skipped in docker image
        # Pciutils and kmod are required if metis driver is not installed
        apt install -y \
            axelera-runtime-$AXELERA_VERSION \
            axelera-device-$AXELERA_VERSION \
            axelera-riscv-gnu-newlib-toolchain-409b951ba662-7 \
            pciutils \
            kmod && \
        # Write runtime library path to ldconfig's configuration directory instead of LD_LIBRARY_PATH
        echo "/opt/axelera/runtime-$AXELERA_VERSION-1/lib" > /etc/ld.so.conf.d/axelera.conf && \
        ldconfig && \
        echo "Axelera installation completed" >> /buildLog.txt; \
    else \
        echo "Axelera version not found, skipping installation" >> /buildLog.txt; \
    fi
    

# Cleanup / remove unnecessary files
RUN rm -f /degirum/Dockerfile /degirum/README.md /degirum/*.deb && \
    rm -rf /var/lib/apt/lists/* && \
    pip cache purge

# run degirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo", "--protocol", "both" ]
