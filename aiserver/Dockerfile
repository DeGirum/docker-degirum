FROM python:3.9-slim

# custom whl installation support
ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

# install prerequisites
RUN apt-get update \
    && apt-get install -y libusb-dev \
    && rm -rf /var/lib/apt/lists/*

# install PySDK
COPY Dockerfile ${DG_CUSTOM_WHEEL} ./
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
