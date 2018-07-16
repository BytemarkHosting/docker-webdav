FROM httpd:alpine
MAINTAINER Bytemark Hosting "support@bytemark.co.uk"

# This variable is inherited from httpd:alpine image:
# ENV HTTPD_PREFIX /usr/local/apache2

RUN set -ex; \
    # Create Debian-style subdirectories.
    mkdir -p "$HTTPD_PREFIX/conf/conf-available"; \
    mkdir -p "$HTTPD_PREFIX/conf/conf-enabled"; \
    mkdir -p "$HTTPD_PREFIX/conf/sites-available"; \
    mkdir -p "$HTTPD_PREFIX/conf/sites-enabled"

# Copy in our configuration files.
COPY dav.conf "$HTTPD_PREFIX/conf/conf-available"
COPY default.conf "$HTTPD_PREFIX/conf/sites-available"
COPY default-ssl.conf "$HTTPD_PREFIX/conf/sites-available"

RUN set -ex; \
    # Create empty default DocumentRoot.
    mkdir -p "/var/www/html"; \
    # Create directories for Dav data and lock database.
    mkdir -p "/var/lib/dav"; \
    mkdir -p "/var/lib/dav/data"; \
    touch "/var/lib/dav/DavLock"; \
    chown -R www-data:www-data "/var/lib/dav"; \
    \
    # Enable DAV modules.
    for i in dav dav_fs; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "$HTTPD_PREFIX/conf/httpd.conf"; \
    done; \
    \
    # Make sure authentication modules are enabled.
    for i in authn_core authn_file authz_core authz_user auth_basic auth_digest; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "$HTTPD_PREFIX/conf/httpd.conf"; \
    done; \
    \
    # Make sure other modules are enabled.
    for i in alias headers mime setenvif; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "$HTTPD_PREFIX/conf/httpd.conf"; \
    done; \
    \
    # Run httpd as "www-data" (instead of "daemon").
    for i in User Group; do \
        sed -i -e "s|^$i .*|$i www-data|" "$HTTPD_PREFIX/conf/httpd.conf"; \
    done; \
    \
    # Include enabled configs and sites.
    printf '%s\n' "Include conf/conf-enabled/*.conf" \
        >> "$HTTPD_PREFIX/conf/httpd.conf"; \
    printf '%s\n' "Include conf/sites-enabled/*.conf" \
        >> "$HTTPD_PREFIX/conf/httpd.conf"; \
    \
    # Enable dav and default site.
    ln -s ../conf-available/dav.conf "$HTTPD_PREFIX/conf/conf-enabled"; \
    ln -s ../sites-available/default.conf "$HTTPD_PREFIX/conf/sites-enabled"

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
EXPOSE 80/tcp 443/tcp
ENTRYPOINT [ "docker-entrypoint.sh" ]
CMD [ "httpd-foreground" ]
