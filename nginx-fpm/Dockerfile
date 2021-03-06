FROM alpine:3.12

# Install packages
RUN apk --no-cache add php7 php7-fpm php7-mysqli php7-json php7-openssl php7-curl php7-fileinfo php7-pdo php7-zip php7-pdo_mysql \
    php7-zlib php7-xml php7-phar php7-intl php7-dom php7-xmlreader php7-ctype php7-session \
    php7-mbstring php7-gd php7-pdo_sqlite nginx supervisor gettext nano

# Configure nginx
COPY config/nginx.conf /tmp/nginx.template
ENV NGINX_SERVERNAME="LUYAPROD"
RUN envsubst '${NGINX_SERVERNAME}' < /tmp/nginx.template > /etc/nginx/nginx.conf

# Configure PHP-FPM
COPY config/fpm-pool.conf /etc/php7/php-fpm.d/www.conf
COPY config/php.ini /etc/php7/conf.d/zzz_custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Setup document root
RUN mkdir -p /var/www/html
RUN mkdir -p /var/www/html/public_html/assets
RUN chmod 777 /var/www/html/public_html/assets
RUN mkdir -p /var/www/html/runtime
RUN chmod 777 /var/www/html/runtime


# Make the document root a volume
VOLUME /var/www/html

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html

COPY config/env.php /var/www/html/configs/env.php

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
