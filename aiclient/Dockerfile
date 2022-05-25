FROM python:3.9-slim
WORKDIR /degirum
RUN apt-get update && apt-get install -y \
  git
RUN git clone https://github.com/DeGirum/PySDKExamples.git
COPY requirements.txt ./
RUN pip install install -r requirements.txt --extra-index-url https://DeGirum.github.io/simple
CMD [ "python", "-m", "jupyterlab", "--ip", "0.0.0.0", "--no-browser", "--allow-root" ]
EXPOSE 8888
