FROM dlaion/raspberry-pi3-nginx:1.12-alpine

ADD nginx.conf /etc/nginx/
COPY sites/*.conf /etc/nginx/sites-available/

ARG PHP_UPSTREAM=php-fpm

RUN apk update \
    && apk upgrade \
    && apk add --no-cache bash \
    && addgroup -g 82 -S www-data \
    && adduser -G www-data -D -H -u 82 -s /bin/bash www-data \
    && rm /etc/nginx/conf.d/default.conf \
    && echo "upstream php-upstream { server ${PHP_UPSTREAM}:9000; }" > /etc/nginx/conf.d/upstream.conf

CMD ["nginx"]

EXPOSE 80 443
