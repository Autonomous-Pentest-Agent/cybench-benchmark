FROM python:3.9-slim-bullseye 
# apa-patch: base image's own stale security suffix (no-op if absent)
RUN sed -i -E "s#((deb|security)\.debian\.org/debian-security[^ ]*[[:space:]])([a-zA-Z0-9]+)/updates#\1\3-security#g" /etc/apt/sources.list || true

COPY ./challenge/requirements.txt /root

ENV HTTP_PORT=1337
ENV TCP_PORT=1338

RUN python3 -m pip install -r /root/requirements.txt && \
    useradd -m ctf && \
    apt update && \
    apt install -y curl socat git && \
    mkdir -p /startup && \
    mkdir -p /var/log/ctf


RUN true \
    && curl -L https://foundry.paradigm.xyz | bash \
    && bash -c "source /root/.bashrc && foundryup" \
    && chmod 755 -R /root \
    && true

COPY ./config/98-start-gunicorn /startup
COPY ./config/99-start-socat /startup
COPY ./config/entrypoint.sh /

COPY ./challenge/eth_sandbox /usr/lib/python/eth_sandbox

ENV PYTHONPATH /usr/lib/python

COPY ./challenge/deploy/ /home/ctf/
COPY ./challenge/contracts /tmp/contracts

RUN true \
    && chmod 777 /entrypoint.sh \
    && cd /tmp \
    && /root/.foundry/bin/forge build --out /home/ctf/compiled \
    && rm -rf /tmp/contracts \
    && true

EXPOSE 1337
EXPOSE 1338
ENTRYPOINT ["/bin/bash", "-c", "./entrypoint.sh"]
