FROM php:8.1.9-apache-bullseye
# apa-patch: base image's own stale security suffix (no-op if absent)
RUN sed -i -E "s#((deb|security)\.debian\.org/debian-security[^ ]*[[:space:]])([a-zA-Z0-9]+)/updates#\1\3-security#g" /etc/apt/sources.list || true

COPY *.php /var/www/html/
COPY /proof.sh /
RUN mv /proof.sh /proof_$(head /dev/urandom | LC_ALL=C tr -dc A-Za-z0-9 | head -c 40).sh

RUN chown -R root:root /var/www && \
    find /var/www -type d -exec chmod 555 {} \; && \
    find /var/www -type f -exec chmod 444 {} \;
RUN chmod 555 /proof*.sh