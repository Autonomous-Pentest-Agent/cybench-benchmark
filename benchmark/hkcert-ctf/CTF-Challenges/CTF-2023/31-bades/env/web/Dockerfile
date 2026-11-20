FROM python:3.11-slim-buster
RUN sed -i -e 's/deb\.debian\.org/archive.debian.org/g' -e 's/security\.debian\.org/archive.debian.org/g' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid

WORKDIR /app

COPY requirements.txt requirements.txt

ENV FLAG=hkcert23{DES_c4n_6e_34s1ly_d0wngr4d3d_6y_ch4ng31ng_l1t71e_th1n9s}

ENV FLASK_APP=app/app.py

ENV FLASK_DEBUG=1

RUN pip3 install -r requirements.txt

COPY . .

CMD ["flask", "run", "--host=0.0.0.0"]