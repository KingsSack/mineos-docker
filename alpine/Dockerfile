FROM node:current-alpine3.12
MAINTAINER William Dizon <wdchromium@gmail.com>

RUN ports -u \
  && prt-get install -if python \
  && rm -rf /usr/ports/*

#enable prt-get repos
RUN mv /etc/ports/contrib.rsync.inactive /etc/ports/contrib.rsync \
  && sed -i 's/#prtdir \/usr\/ports\/contrib/prtdir \/usr\/ports\/contrib/' /etc/prt-get.conf

RUN ports -u \
  && prt-get install expat \
  git \
  librsync \
  rdiff-backup \
  screen \
  setuptools \
  meld3 \
  supervisor \
  icu \
  && rm -rf /usr/ports/*

#install java
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'

RUN apk add --no-cache tzdata musl-locales musl-locales-lang \
    && rm -rf /var/cache/apk/*

ENV JAVA_VERSION jdk-16.0.1+9

RUN set -eux; \
    apk add --no-cache --virtual .fetch-deps curl; \
    ARCH="$(apk --print-arch)"; \
    case "${ARCH}" in \
       amd64|x86_64) \
         ESUM='5f10ffc12056b735f5fe505ec6c481fff94de27e80c1d77ea9f9e2830669cc1d'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk16-binaries/releases/download/jdk-16.0.1%2B9/OpenJDK16U-jre_x64_alpine-linux_hotspot_16.0.1_9.tar.gz'; \
         ;; \
       *) \
         echo "Unsupported arch: ${ARCH}"; \
         exit 1; \
         ;; \
    esac; \
    curl -LfsSo /tmp/openjdk.tar.gz ${BINARY_URL}; \
    echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c -; \
    mkdir -p /opt/java/openjdk; \
    cd /opt/java/openjdk; \
    tar -xf /tmp/openjdk.tar.gz --strip-components=1; \
    apk del --purge .fetch-deps; \
    rm -rf /var/cache/apk/*; \
    rm -rf /tmp/openjdk.tar.gz;

ENV JAVA_HOME=/opt/java/openjdk \
    PATH="/opt/java/openjdk/bin:$PATH"

##install chpassword utility
#RUN cd /root \
#  && httpup sync http://kyber.io/crux/ports/teatime/#chpasswd chpasswd \
#  && cd /root/chpasswd \
#  && pkgmk -d -i \
#  && cd /root \
#  && rm -rf /root/chpasswd

#download mineos from github
RUN mkdir -p /usr/games/minecraft \
  && cd /usr/games/minecraft \
  && git clone --depth=1 https://github.com/hexparrot/mineos-node.git . \
  && cp mineos.conf /etc/mineos.conf \
  && chmod +x webui.js mineos_console.js service.js \
  && npm install

#configure and run supervisor
RUN cp /usr/games/minecraft/init/supervisor_conf /etc/supervisor.d/mineos.conf
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]

#entrypoint allowing for setting of mc password
COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 8443 25565-25570
VOLUME /var/games/minecraft
