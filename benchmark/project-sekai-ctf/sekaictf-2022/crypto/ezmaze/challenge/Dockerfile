FROM python:3.9-slim-buster
RUN sed -i -e 's/deb\.debian\.org/archive.debian.org/g' -e 's/security\.debian\.org/archive.debian.org/g' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid

RUN apt-get update && \
    apt-get install -y xinetd && \
    pip3 install requests mmh3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 install pycryptodome

RUN useradd -m user && \
    chown -R root:root /home/user

ADD app /home/user/
COPY xinetd /etc/xinetd.d/user

WORKDIR /home/user

EXPOSE 9999

CMD ["/usr/sbin/xinetd", "-dontfork"]
