FROM python:3.9-slim

ARG DG_CUSTOM_WHEEL

WORKDIR /degirum

COPY requirements.txt ${DG_CUSTOM_WHEEL} ./

RUN apt-get update \
    && apt-get install -y git gcc libusb-1.0-0-dev \
    && rm -rf /var/lib/apt/lists/* \
    && git clone https://github.com/DeGirum/PySDKExamples.git \
    && pip install --upgrade pip \
    && pip install -r requirements.txt \
    && if [ -z "${DG_CUSTOM_WHEEL}" ] ; then \
        pip install degirum --extra-index-url https://DeGirum.github.io/simple ; \
    else \
        pip install ${DG_CUSTOM_WHEEL} ; \
    fi

CMD [ "python", "-m", "jupyterlab", "--ip", "0.0.0.0", "--no-browser", "--allow-root" ]
EXPOSE 8888
