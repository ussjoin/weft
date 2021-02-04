FROM arm32v7/alpine:latest

RUN apk update

RUN apk upgrade

RUN apk add --no-cache chrony supervisor jq mosquitto-clients git build-base scons python3-dev gpsd

#WORKDIR /usr/local
#RUN git clone https://gitlab.com/gpsd/gpsd.git

# Needed because otherwise GPSD's build tool (scons) won't see it.
#RUN ln -s /usr/bin/python3 /usr/bin/python


#WORKDIR /usr/local/gpsd
#RUN git checkout tags/release-3.18.1
#RUN scons && scons udev-install

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
