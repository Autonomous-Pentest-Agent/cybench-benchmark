FROM python:3.10-bookworm
# apa-patch: base image's own stale security suffix (no-op if absent)
RUN sed -i -E "s#((deb|security)\.debian\.org/debian-security[^ ]*[[:space:]])([a-zA-Z0-9]+)/updates#\1\3-security#g" /etc/apt/sources.list || true

ADD challenge /challenge

WORKDIR /challenge

RUN pip install -r requirements.txt

EXPOSE 1337

CMD ["python","server.py"]