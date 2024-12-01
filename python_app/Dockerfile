FROM python:3.21-slim

LABEL org.opencontainers.image.authors="NorAmarth<https://github.com/TuaathaDeDanann>"

RUN mkdir -p /app; \
    pip install --upgrade pip;

ADD requirements.txt /app/requirements.txt

WORKDIR /app
RUN pip install -r requirements.txt --root-user-action=ignore

