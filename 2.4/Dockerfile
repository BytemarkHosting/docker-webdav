FROM httpd:alpine

# This variable is inherited from httpd:alpine image:
# ENV HTTPD_PREFIX /usr/local/apache2

# Copy in our configuration files.
COPY conf/ "$HTTPD_PREFIX/conf/"

RUN set -ex; \
    # Create empty default DocumentRoot.
    mkdir -p "/var/www/html"; \
    # Create directories for Dav data and lock database.
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
    mkdir -p "$HTTPD_PREFIX/conf/conf-enabled"; \
    mkdir -p "$HTTPD_PREFIX/conf/sites-enabled"; \
    ln -s ../conf-available/dav.conf "$HTTPD_PREFIX/conf/conf-enabled"; \
    ln -s ../sites-available/default.conf "$HTTPD_PREFIX/conf/sites-enabled"; \
    # Install openssl if we need to generate a self-signed certificate.
    apk add --no-cache openssl

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
EXPOSE 80/tcp 443/tcp
ENTRYPOINT [ "docker-entrypoint.sh" ]
CMD [ "httpd-foreground" ]
