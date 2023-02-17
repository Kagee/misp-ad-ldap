FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_PRIORITY critical

RUN apt-get update && \
      apt-get -qy upgrade && \
      apt-get -qy install \
        sudo \
        supervisor \
        git \
        vim \
        mariadb-client

#  sudo apt-get install curl gcc git gpg-agent make python3 openssl redis-server sudo vim zip unzip virtualenv libfuzzy-dev sqlite3 moreutils -qy
#  sudo apt-get install apache2 apache2-doc apache2-utils -qy
#  sudo apt-get install python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev python-setuptools -qy
# sudo apt install -qy \
#  libapache2-mod-php7.4 \
#  php7.4 php7.4-cli \
#  php7.4-dev \
#  php7.4-json php7.4-xml php7.4-mysql php7.4-opcache php7.4-readline php7.4-mbstring php7.4-zip php7.4-curl \
#  php7.4-redis php7.4-gnupg \
#  php7.4-intl php7.4-bcmath \
#  php7.4-gd




RUN useradd -m -s /bin/bash -N -u 1000 misp && \
    echo "MISP ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers

USER misp
CMD "/bin/bash"
