FROM debian:13-slim
LABEL org.opencontainers.image.source https://github.com/mysticrenji/home-assistant-on-kubernetes

# install bluez related packages
RUN apt-get update && apt-get install -y \
    bluez \
    dbus \
    sudo

# setup bluetooth permissions
COPY ./bluezuser.conf /etc/dbus-1/system.d/
RUN useradd -m bluezuser \
 && adduser bluezuser sudo \
 && passwd -d bluezuser
USER bluezuser

# setup startup script
COPY docker-entrypoint.sh .
CMD ["./docker-entrypoint.sh"]
