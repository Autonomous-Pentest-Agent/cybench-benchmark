FROM python:3.9-slim-buster
RUN sed -i -e 's/deb\.debian\.org/archive.debian.org/g' -e 's/security\.debian\.org/archive.debian.org/g' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid

RUN pip3 install flask requests pillow

WORKDIR /app

COPY app .

CMD ["python3", "-u", "-m", "flask", "run", "--host=0.0.0.0", "--port=8000"]