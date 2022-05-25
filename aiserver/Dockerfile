FROM python:3.9-slim
WORKDIR /degirum
RUN pip install degirum --extra-index-url https://DeGirum.github.io/simple
CMD [ "python", "-m", "degirum.server", "--zoo", "/zoo" ]
EXPOSE 8778
