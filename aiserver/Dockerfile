FROM python:3.9-slim

ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

COPY Dockerfile ${DG_CUSTOM_WHEEL} ./

RUN apt-get update \
    && apt-get install -y libusb-1.0-0-dev \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --upgrade pip \
    && if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum  ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo

# run degirum AI server
CMD [ "python", "-m", "degirum.server", "--zoo", "/zoo" ]
EXPOSE 8778
