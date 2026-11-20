FROM debian:bullseye-slim
# apa-patch: base image's own stale security suffix (no-op if absent)
RUN sed -i -E "s#((deb|security)\.debian\.org/debian-security[^ ]*[[:space:]])([a-zA-Z0-9]+)/updates#\1\3-security#g" /etc/apt/sources.list || true
COPY --from=krallin/ubuntu-tini /usr/bin/tini /tini
ENTRYPOINT ["/tini", "--"]

RUN apt-get update && \
    apt-get -y install libudev-dev libssl-dev pkg-config && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY arcade-server Arcade.so flag.txt ./

EXPOSE 8080
USER nobody
CMD ["/app/arcade-server"]