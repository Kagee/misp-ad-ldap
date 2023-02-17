FROM ubuntu:20.04


ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_PRIORITY critical

RUN apt-get update && \
      apt-get -qy upgrade && \
      apt-get -qy install \
        sudo \
        supervisor \
        git \
        vim

RUN useradd -m -s /bin/bash -N -u 1000 misp && \
    echo "MISP ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/misp

RUN apt-get -qy install \
        mariadb-client curl gcc git gpg-agent make python3 openssl \
        zip unzip virtualenv libfuzzy-dev sqlite3 moreutils \
        apache2 apache2-doc apache2-utils \
        python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev \
        python-setuptools \
        libapache2-mod-php7.4 php7.4 php7.4-cli php7.4-dev \
        php7.4-json php7.4-xml php7.4-mysql php7.4-opcache \
        php7.4-readline php7.4-mbstring php7.4-zip php7.4-curl \
        php7.4-redis php7.4-gnupg php7.4-intl php7.4-bcmath \
        php7.4-gd php7.4-ldap

ENV PHP_ETC_BASE=/etc/php/7.4
ENV PHP_INI=/etc/php/7.4/apache2/php.ini

# Do PHP config changes here

RUN cat $PHP_INI

ENV PATH_TO_MISP /var/www/MISP
ENV WWW_USER www-data
ENV SUDO_WWW "sudo -H -u ${WWW_USER} "

RUN mkdir ${PATH_TO_MISP}; \
    chown ${WWW_USER}:${WWW_USER} ${PATH_TO_MISP}

RUN ${SUDO_WWW} git clone https://github.com/MISP/MISP.git ${PATH_TO_MISP};
RUN ${SUDO_WWW} git -C ${PATH_TO_MISP} submodule update --progress --init --recursive;
RUN ${SUDO_WWW} git -C ${PATH_TO_MISP} submodule foreach --recursive git config core.filemode false;
RUN ${SUDO_WWW} git -C ${PATH_TO_MISP} config core.filemode false;

RUN ${SUDO_WWW} virtualenv -p python3 ${PATH_TO_MISP}/venv; \
    mkdir /var/www/.cache/; \
    chown ${WWW_USER}:${WWW_USER} /var/www/.cache; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install ordered-set python-dateutil six weakrefmethod; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install ${PATH_TO_MISP}/app/files/scripts/misp-stix; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install ${PATH_TO_MISP}/PyMISP;

#    # FIXME: Remove libfaup etc once the egg has the library baked-in
#    sudo apt-get install cmake libcaca-dev liblua5.3-dev -y
#    cd /tmp
#    false; while [[ $? -ne 0 ]]; do [[ ! -d "faup" ]] && ${SUDO_CMD} git clone https://github.com/stricaud/faup.git faup; done
#    false; while [[ $? -ne 0 ]]; do [[ ! -d "gtcaca" ]] && ${SUDO_CMD} git clone https://github.com/stricaud/gtcaca.git gtcaca; done
#    sudo chown -R ${MISP_USER}:${MISP_USER} faup gtcaca
#    cd gtcaca
##    ${SUDO_CMD} mkdir -p build
#    cd build
#    ${SUDO_CMD} cmake .. && ${SUDO_CMD} make
#    sudo make install
#    cd ../../faup
#    ${SUDO_CMD} mkdir -p build
#    cd build
#    ${SUDO_CMD} cmake .. && ${SUDO_CMD} make
#    sudo make install
#    sudo ldconfig

    # install pydeep
RUN ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install git+https://github.com/kbandla/pydeep.git; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install lief; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install zmq redis; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install python-magic; \
    ${SUDO_WWW} ${PATH_TO_MISP}/venv/bin/pip install plyara;

RUN mkdir -p /var/www/.composer; \
    chown ${WWW_USER}:${WWW_USER} /var/www/.composer; \
    ${SUDO_WWW} sh -c "cd ${PATH_TO_MISP}/app ;php composer.phar install --no-dev"

RUN phpenmod redis; phpenmod gnupg

RUN chown -R ${WWW_USER}:${WWW_USER} ${PATH_TO_MISP}; \
      chmod -R 750 ${PATH_TO_MISP}; \
      chmod -R g+ws ${PATH_TO_MISP}/app/tmp; \
      chmod -R g+ws ${PATH_TO_MISP}/app/files; \
      chmod -R g+ws ${PATH_TO_MISP}/app/files/scripts/tmp;

RUN cp ${PATH_TO_MISP}/INSTALL/apache.24.misp.ssl /etc/apache2/sites-available/misp-ssl.conf

  # create a self-signed certificate:
RUN openssl req -newkey rsa:4096 -days 365 -nodes -x509 \
  -subj "/C=${OPENSSL_C}/ST=${OPENSSL_ST}/L=${OPENSSL_L}/O=${OPENSSL_O}/OU=${OPENSSL_OU}/CN=${OPENSSL_CN}/emailAddress=${OPENSSL_EMAILADDRESS}" \
  -keyout /etc/ssl/private/misp.local.key -out /etc/ssl/private/misp.local.crt

  # Enable modules, settings, and default of SSL in Apache
RUN a2dismod status; \
      a2enmod ssl rewrite headers; \
      a2dissite 000-default; \
      a2ensite misp-ssl

RUN cp ${PATH_TO_MISP}/INSTALL/misp.logrotate /etc/logrotate.d/misp; \
      chmod 0640 /etc/logrotate.d/misp

RUN ${SUDO_WWW} cp -a ${PATH_TO_MISP}/app/Config/bootstrap.default.php ${PATH_TO_MISP}/app/Config/bootstrap.php; \
      ${SUDO_WWW} cp -a ${PATH_TO_MISP}/app/Config/database.default.php ${PATH_TO_MISP}/app/Config/database.php; \
      ${SUDO_WWW} cp -a ${PATH_TO_MISP}/app/Config/core.default.php ${PATH_TO_MISP}/app/Config/core.php; \
      ${SUDO_WWW} cp -a ${PATH_TO_MISP}/app/Config/config.default.php ${PATH_TO_MISP}/app/Config/config.php;

RUN  echo "<?php class DATABASE_CONFIG { public \$default = array( 'datasource' => 'Database/Mysql', 'persistent' => false, 'host' => 'db', 'login' => 'misp', 'port' => 3306, 'password' => 'example', 'database' => 'misp', 'prefix' => '',  'encoding' => 'utf8' ); } ?>" | ${SUDO_WWW} tee ${PATH_TO_MISP}/app/Config/database.php

  # Important! Change the salt key in ${PATH_TO_MISP}/app/Config/config.php
  # The salt key must be a string at least 32 bytes long.
  # The admin user account will be generated on the first login, make sure that the salt is changed before you create that user
  # If you forget to do this step, and you are still dealing with a fresh installation, just alter the salt,
  # delete the user from mysql and log in again using the default admin credentials (admin@admin.test / admin)

  # and make sure the file permissions are still OK
RUN chown -R ${WWW_USER}:${WWW_USER} ${PATH_TO_MISP}/app/Config; \
      chmod -R 750 ${PATH_TO_MISP}/app/Config

USER misp
CMD "/bin/bash"
