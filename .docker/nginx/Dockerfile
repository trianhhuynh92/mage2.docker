FROM pagespeed/nginx-pagespeed

LABEL maintainer="Osiozekhai Aliu"

ARG WORKDIR_SERVER
ARG TZ
ARG USER
ARG SHOP_URI
ARG SSL

COPY conf/nginx.conf /etc/nginx/nginx.conf
COPY conf/default.conf /etc/nginx/conf.d/default.conf
COPY conf/default_ssl.conf /etc/nginx/conf.d/default_ssl.conf
COPY conf/env.php /home/${USER}/env.php
COPY docker-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-entrypoint.sh

RUN apk add --update openssl tzdata curl \
    && ln -sf /usr/share/zoneinfo/Etc/$TZ  /etc/localtime \
    && echo $TZ > /etc/timezone \
    && addgroup -g 1000 --system ${USER} \
    && adduser -u 1000 --system -D -G ${USER} ${USER} \
    && mkdir ${WORKDIR_SERVER} \
    && chown -R ${USER}:${USER} ${WORKDIR_SERVER} \
    && chmod -R 755 ${WORKDIR_SERVER} \
    && mkdir -p /etc/letsencrypt/

RUN sed -i "s#__user#${USER}#g" /etc/nginx/nginx.conf \
    && sed -i "s#__working_dir#${WORKDIR_SERVER}#g" /etc/nginx/conf.d/default.conf \
    && sed -i "s#__shop_uri#${SHOP_URI}#g" /etc/nginx/conf.d/default.conf \
    && sed -i "s#__working_dir#${WORKDIR_SERVER}#g" /etc/nginx/conf.d/default_ssl.conf \
    && sed -i "s#__shop_uri#${SHOP_URI}#g" /etc/nginx/conf.d/default_ssl.conf \
    && apk del tzdata \
    && rm -rf /var/cache/apk/*

RUN if [ "$SSL" = "true" ]; then \
        mkdir -p /etc/nginx/ssl \
        && mkdir -p /var/cache/ngx_pagespeed \
        && chown -R ${USER}:${USER} /etc/nginx/ssl/ \
        && chown -R ${USER}:${USER} /var/cache/ngx_pagespeed \
        && chmod 755 -R /var/cache/ngx_pagespeed \
        && openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/privkey.pem -out /etc/nginx/ssl/fullchain.pem -subj /CN=${SHOP_URI} \
        && sed -i "s#__default#default_ssl#g" /etc/nginx/nginx.conf; \
    else \
        sed -i "s#__default#default#g" /etc/nginx/nginx.conf; \
    fi