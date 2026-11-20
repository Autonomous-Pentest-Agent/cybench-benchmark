FROM python:3.9-slim-buster
RUN sed -i -e 's/deb\.debian\.org/archive.debian.org/g' -e 's/security\.debian\.org/archive.debian.org/g' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid
WORKDIR /app
RUN pip3 install requests aiohttp inflect edge-tts cchardet aiodns
COPY . .
CMD ["python3", "-m", "aiohttp.web", "-H", "0.0.0.0", "-P", "8000", "server:init"]
# docker build --tag project-sekai-ctf-mind-calculator .
# docker run -d --publish 22528:8000 --name project-sekai-ctf-mind-calculator project-sekai-ctf-mind-calculator