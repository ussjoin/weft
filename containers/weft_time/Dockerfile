FROM arm32v7/alpine:latest

# This line links GHCR.io with the repository.
LABEL org.opencontainers.image.source=https://github.com/ussjoin/weft/tree/main/containers/weft_time

RUN apk update

RUN apk upgrade

RUN apk add --no-cache chrony supervisor jq mosquitto-clients git build-base scons python3-dev gpsd

COPY config/chrony.conf /etc/chrony/chrony.conf
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN mkdir -p /etc/periodic/1min

COPY config/send_loc /etc/periodic/1min

RUN echo "*       *       *       *       *       run-parts /etc/periodic/1min" >> /etc/crontabs/root

# NTP, UDP
EXPOSE 123 

# GPSD, TCP
EXPOSE 2947

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

HEALTHCHECK CMD chronyc tracking || exit 1
