FROM centos:centos7

LABEL org.opencontainers.image.title="Zabbix web-interface (Apache, PostgreSQL)" \
      org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.description="Zabbix web-interface based on Apache2 web server with PostgreSQL database support" \
      org.opencontainers.image.licenses="GPL v2.0"

STOPSIGNAL SIGTERM

RUN set -eux && \
    groupadd -g 1995 --system zabbix && \
    adduser -r --shell /sbin/nologin \
            -g zabbix -G root \
            -d /var/lib/zabbix/ -u 1997 \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /etc/zabbix/web && \
    yum --quiet makecache && \
    yum -y install --setopt=tsflags=nodocs \
            dejavu-sans-fonts \
            curl \
            httpd \
            mod_ssl \
            php \
            php-bcmath \
            php-gd \
            php-ldap \
            php-mbstring \
            php-pgsql \
            php-xml \
            postgresql && \
    rm -f "/etc/httpd/conf.d/default.conf" && \
    rm -f "/etc/httpd/conf.d/ssl.conf" && \
    sed -ri \
        -e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g' \
        -e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' \
        "/etc/httpd/conf/httpd.conf" && \
    sed -i 's/Listen 80/Listen 8080/g' /etc/httpd/conf/httpd.conf && \
    yum -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /etc/udev/hwdb.bin /root/.pki 

ARG MAJOR_VERSION=master
ARG ZBX_VERSION=${MAJOR_VERSION}
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.version="${ZBX_VERSION}" \
      org.opencontainers.image.source="${ZBX_SOURCES}"

COPY ["conf/etc/", "/etc/"]

RUN set -eux && \
    yum --quiet makecache && \
    yum -y install --setopt=tsflags=nodocs \
            gettext \
            git && \
    cd /usr/share/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    mkdir /usr/share/zabbix/ && \
    cp -R /usr/share/zabbix-${ZBX_VERSION}/frontends/php/* /usr/share/zabbix/ && \
    rm -rf /usr/share/zabbix-${ZBX_VERSION}/ && \
    cd /usr/share/zabbix/ && \
    rm -f conf/zabbix.conf.php && \
    rm -rf tests && \
    ./locale/make_mo.sh && \
    ln -s "/etc/zabbix/web/zabbix.conf.php" "/usr/share/zabbix/conf/zabbix.conf.php" && \
    cat /usr/share/zabbix/include/locales.inc.php | grep display | grep true | awk '{$1=$1};1' | \
                cut -d"'" -f 2 | sort | \
                xargs -I '{}' bash -c 'echo "{}" && localedef -c -i {} -f UTF-8 {}.UTF-8 2>/dev/null' && \
    chown --quiet -R zabbix:root /etc/zabbix/  && \
    chgrp -R 0 /etc/zabbix/  && \
    chmod -R g=u /etc/zabbix/  && \
    chown --quiet -R zabbix:root /etc/httpd/ /etc/php.d/ /etc/php.ini && \
    chgrp -R 0 /etc/httpd/ /etc/php.d/ /etc/php.ini && \
    chmod -R g=u /etc/httpd/ /etc/php.d/ /etc/php.ini && \
    chown --quiet -R zabbix:root /run/httpd/ /var/lib/php/session/ && \
    chgrp -R 0 /run/httpd/ /var/lib/php/session/ && \
    chmod -R g=u /run/httpd/ /var/lib/php/session/ && \
    yum -y history undo `yum -q history | sed -n 3p |column -t | cut -d' ' -f1` && \
    yum -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /etc/udev/hwdb.bin /root/.pki


EXPOSE 8080/TCP 8443/TCP

WORKDIR /usr/share/zabbix

VOLUME ["/etc/ssl/apache2"]

COPY ["docker-entrypoint.sh", "/usr/bin/"]

USER 1997

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
