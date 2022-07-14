FROM python:3.9-slim

# custom whl installation support
ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

# install prerequisites
RUN apt-get update \
    && apt-get install -y curl gpg libusb-dev

# install edgetpu runtime
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
    && echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list \
    && apt-get update \
    && apt-get install -y libedgetpu-dev \
    && rm -rf /var/lib/apt/lists/*

# install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
RUN pip install --upgrade pip
RUN if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum --extra-index-url https://DeGirum.github.io/simple ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ;\
    fi

# crete model zoo folder
RUN mkdir -p /zoo

# run degirum AI server
CMD [ "python", "-m", "degirum.server", "--zoo", "/zoo" ]
EXPOSE 8778
