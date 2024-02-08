FROM openvino/ubuntu22_runtime:2023.3.0

ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
USER root
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

# run degirum AI server
CMD [ "python3", "-m", "degirum.server", "--zoo", "/zoo" ]
EXPOSE 8778
