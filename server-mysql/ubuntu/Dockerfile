FROM cloudron/base:2.0.0@sha256:f9fea80513aa7c92fe2e7bf3978b54c8ac5222f47a9a32a7f8833edf0eb5a4f4

LABEL org.opencontainers.image.title="Zabbix server (MySQL)" \
      org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.description="Zabbix server with MySQL database support" \
      org.opencontainers.image.licenses="GPL v2.0"

STOPSIGNAL SIGTERM

RUN set -eux && \
    # Not needed for cloudron
    #addgroup --system --gid 1995 --quiet zabbix && \
    #adduser --quiet \
    #        --system --disabled-login \
    #        --ingroup zabbix --ingroup root \
    #        --uid 1997 \
    #        --home /var/lib/zabbix/ \
    #    zabbix && \
    #usermod -G zabbix,dialout zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /var/lib/zabbix/export && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    mkdir -p /usr/lib/zabbix/alertscripts && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /usr/share/doc/zabbix-server-mysql && \
    apt-get -y update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
            # Cloudron changes - Not needed
            #tini \
            tzdata \
            ca-certificates \
            iputils-ping \
            traceroute \
            fping \
            libcurl4 \
            libevent-2.1 \
            # Cloudron changes - Use right version for Ubuntu 18.04 LTS
            libmysqlclient20 \
            libopenipmi0 \
            libpcre3 \
            # Cloudron changes - Use right version for Ubuntu 18.04 LTS
            libsnmp30 \
            libssh-4 \
            libssl1.1 \
            libxml2 \
            mysql-client \
            snmp-mibs-downloader \
            unixodbc && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

ARG MAJOR_VERSION=5.0
ARG ZBX_VERSION=${MAJOR_VERSION}.4
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm MIBDIRS=/var/lib/snmp/mibs/ietf:/var/lib/snmp/mibs/iana:/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.version="${ZBX_VERSION}" \
      org.opencontainers.image.source="${ZBX_SOURCES}"

RUN set -eux && \
    apt-get -y update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libevent-dev \
            libldap2-dev \
            libmysqlclient-dev \
            libopenipmi-dev \
            libpcre3-dev \
            libsnmp-dev \
            libssh-dev \
            libxml2-dev \
            make \
            pkg-config \
            git \
            unixodbc-dev && \
    cd /tmp/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    cd /tmp/zabbix-${ZBX_VERSION} && \
    zabbix_revision=`git rev-parse --short HEAD` && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    ./bootstrap.sh && \
    export CFLAGS="-fPIC -pie -Wl,-z,relro -Wl,-z,now" && \
    ./configure \
            --datadir=/usr/lib \
            --libdir=/usr/lib/zabbix \
            --prefix=/usr \
            --sysconfdir=/etc/zabbix \
            --enable-agent \
            --enable-server \
            --with-mysql \
            --with-ldap \
            --with-libcurl \
            --with-libxml2 \
            --with-net-snmp \
            --with-openipmi \
            --with-openssl \
            --with-ssh \
            --with-unixodbc \
            --enable-ipv6 \
            --silent && \
    make -j"$(nproc)" -s dbschema && \
    make -j"$(nproc)" -s && \
    cp src/zabbix_server/zabbix_server /usr/sbin/zabbix_server && \
    cp src/zabbix_get/zabbix_get /usr/bin/zabbix_get && \
    cp src/zabbix_sender/zabbix_sender /usr/bin/zabbix_sender && \
    cp conf/zabbix_server.conf /etc/zabbix/zabbix_server.conf && \
    cat database/mysql/schema.sql > database/mysql/create.sql && \
    cat database/mysql/images.sql >> database/mysql/create.sql && \
    cat database/mysql/data.sql >> database/mysql/create.sql && \
    gzip database/mysql/create.sql && \
    cp database/mysql/create.sql.gz /usr/share/doc/zabbix-server-mysql/ && \
    cd /tmp/ && \
    rm -rf /tmp/zabbix-${ZBX_VERSION}/ && \
    # Cloudron changes 
    chown --quiet -R cloudron:cloudron /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
    DEBIAN_FRONTEND=noninteractive apt-get -y purge \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libevent-dev \
            libldap2-dev \
            libmysqlclient-dev \
            libopenipmi-dev \
            libpcre3-dev \
            libsnmp-dev \
            libssh-dev \
            libxml2-dev \
            make \
            pkg-config \
            git \
            unixodbc-dev && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

# Cloudron changes - Comment out all we don't need
# EXPOSE 10051/TCP

WORKDIR /app/code

# VOLUME ["/var/lib/zabbix/snmptraps", "/var/lib/zabbix/export"]

# COPY ["docker-entrypoint.sh", "/usr/bin/"]

# ENTRYPOINT ["/usr/bin/tini", "--", "/usr/bin/docker-entrypoint.sh"]

# USER 1997

# CMD ["/usr/sbin/zabbix_server", "--foreground", "-c", "/etc/zabbix/zabbix_server.conf"]
COPY ["docker-entrypoint.sh", "sample.env", "/app/code/"]

# Cloudron - make all allowed volumes avaible in /app/data/ - see https://hub.docker.com/r/zabbix/zabbix-server-mysql > "Allowed volumes for the Zabbix server container"
RUN mv /usr/lib/zabbix/alertscripts /usr/lib/zabbix/alertscripts_default \
    && ln -s /app/data/zabbix/alertscripts /usr/lib/zabbix/alertscripts \
    #
    && mv /usr/lib/zabbix/externalscripts /usr/lib/zabbix/externalscripts_default \
    && ln -s /app/data/zabbix/externalscripts /usr/lib/zabbix/externalscripts \
    #
    && mv /var/lib/zabbix/modules /var/lib/zabbix/modules_default \
    && ln -s /app/data/zabbix/modules /var/lib/zabbix/modules \
    #
    && mv /var/lib/zabbix/enc /var/lib/zabbix/enc_default \
    && ln -s /app/data/zabbix/enc /var/lib/zabbix/enc \
    #
    && mv /var/lib/zabbix/ssh_keys /var/lib/zabbix/ssh_keys_default \
    && ln -s /app/data/zabbix/ssh_keys /var/lib/zabbix/ssh_keys \
    #
    && mv /var/lib/zabbix/ssl/certs /var/lib/zabbix/ssl/certs_default \
    && ln -s /app/data/zabbix/ssl/certs /var/lib/zabbix/ssl/certs \
    #
    && mv /var/lib/zabbix/ssl/keys /var/lib/zabbix/ssl/keys_default \
    && ln -s /app/data/zabbix/ssl/keys /var/lib/zabbix/ssl/keys \
    #
    && mv /var/lib/zabbix/ssl/ssl_ca /var/lib/zabbix/ssl/ssl_ca_default \
    && ln -s /app/data/zabbix/ssl/ssl_ca /var/lib/zabbix/ssl/ssl_ca \
    #
    && mv /var/lib/zabbix/snmptraps /var/lib/zabbix/snmptraps_default \
    && ln -s /app/data/zabbix/snmptraps /var/lib/zabbix/snmptraps \
    #
    && mv /var/lib/zabbix/mibs /var/lib/zabbix/mibs_default \
    && ln -s /app/data/zabbix/mibs /var/lib/zabbix/mibs \
    #
    && mv /var/lib/zabbix/export /var/lib/zabbix/export_default \
    && ln -s /app/data/zabbix/export /var/lib/zabbix/export \
    #
    && mv /etc/zabbix /etc/zabbix_default \
    && ln -s /app/data/zabbix/config /etc/zabbix

CMD ["/app/code/docker-entrypoint.sh"]