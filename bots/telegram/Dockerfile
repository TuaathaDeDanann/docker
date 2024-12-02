FROM python:3.12-slim

LABEL org.opencontainers.image.authors="NorAmarth<https://github.com/TuaathaDeDanann>"

ARG RABBITMQ_URL
ARG TELEGRAM_TOKEN
ARG DASHBOARD_API_URL
ARG REDIS_URL

ENV RABBITMQ_URL=$RABBITMQ_URL
ENV TELEGRAM_TOKEN=$TELEGRAM_TOKEN
ENV DASHBOARD_API_URL=$DASHBOARD_API_URL
ENV REDIS_URL=$REDIS_URL

RUN mkdir -p /app; \
    pip install --upgrade pip;

ADD requirements.txt /app/requirements.txt

WORKDIR /app
RUN pip install -r requirements.txt --root-user-action=ignore

STOPSIGNAL SIGQUIT

CMD ["python", "bot.py"]

