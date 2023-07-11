FROM python:3.9-slim

ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

COPY Dockerfile ${DG_CUSTOM_WHEEL} ./

RUN apt-get update \
    && apt-get install -y curl gpg libusb-1.0-0-dev \
    && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
    && echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list \
    && apt-get update \
    && apt-get install -y libedgetpu-dev \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --upgrade pip \
    && if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi \
    && mkdir -p /zoo

CMD [ "python", "-m", "degirum.server", "--zoo", "/zoo" ]
EXPOSE 8778
