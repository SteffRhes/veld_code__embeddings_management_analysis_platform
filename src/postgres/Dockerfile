FROM debian:bullseye-20250407-slim
ENV PG_MAJOR 17
ENV PG_VERSION 17.4

RUN apt update
RUN apt install -y --no-install-recommends \
  build-essential=12.9* \
  pkg-config=0.29.2* \
  libicu-dev=67.1* \
  bison=2:3.7.5* \
  flex=2.6.4* \
  libreadline-dev=8.1* \
  zlib1g-dev=1:1.2.11* \
  less=551-*

RUN set -eux; \
  if [ -f /etc/dpkg/dpkg.cfg.d/docker ]; then \
    grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
    sed -ri '/\/usr\/share\/locale/d' /etc/dpkg/dpkg.cfg.d/docker; \
    ! grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
  fi; \
  apt-get update; apt-get install -y --no-install-recommends locales; rm -rf /var/lib/apt/lists/*; \
  echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen; \
  locale-gen; \
  locale -a | grep 'en_US.utf8'
ENV LANG en_US.utf8

COPY ./src/ /veld/code/
WORKDIR /veld/code/
RUN ./configure && make && make check && make install
ENV PATH="${PATH}:/usr/local/pgsql/bin"
RUN postgres --version

ENV PGDATA /veld/storage/
RUN mkdir -p /veld/storage/
VOLUME /veld/storage/

RUN mkdir /docker-entrypoint-initdb.d
COPY docker-entrypoint.sh docker-ensure-initdb.sh /usr/local/bin/
RUN ln -sT docker-ensure-initdb.sh /usr/local/bin/docker-enforce-initdb.sh
ENTRYPOINT ["docker-entrypoint.sh"]

STOPSIGNAL SIGINT
EXPOSE 5432
CMD ["postgres", "-c", "config_file=/veld/input/postgresql.conf"]

