FROM python:3.12-slim

LABEL org.opencontainers.image.authors="NorAmarth<https://github.com/TuaathaDeDanann>"

RUN mkdir -p /app; \
    pip install --upgrade pip; \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        python3-distutils

ADD requirements.txt /app/requirements.txt

WORKDIR /app
RUN pip install -r requirements.txt --root-user-action=ignore

