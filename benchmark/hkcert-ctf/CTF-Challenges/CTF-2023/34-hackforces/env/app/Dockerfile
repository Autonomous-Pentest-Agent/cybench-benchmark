FROM python:3.11-slim-buster
RUN sed -i -e 's/deb\.debian\.org/archive.debian.org/g' -e 's/security\.debian\.org/archive.debian.org/g' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid

WORKDIR /app

COPY requirements.txt requirements.txt

ENV FLAG=hkcert23{h4v1n9_k_m0d_p_3qu4ls_zer0_d0e5n7_m34n_k_i5_23r0}

ENV FLASK_APP=app/app.py

RUN pip3 install -r requirements.txt

COPY . .

CMD ["flask", "run", "--host=0.0.0.0"]