FROM ubuntu:18.04

# apt install
## for docker image
RUN apt update && apt install -y \
    git python2.7 sudo dialog systemd
## for stip-rs & stip-sns
RUN apt install -y \
    python-pip apache2 libapache2-mod-wsgi libmysqlclient-dev mysql-client-5.7 python3-dev libpq-dev \
 && apt clean \
 && rm -rf /var/lib/apt/lists/*
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927 && echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
RUN apt update && apt install -y \ 
    mongodb-org \
 && apt clean \
 && rm -rf /var/lib/apt/lists/*

# env
ENV VERSION=v0.1.0 \
    INSTALL_DIR=/opt/s-tip \
    COMMON_DIR=/stip-common
ENV SCRIPTS_DIR ${COMMON_DIR}/install_scripts

# pip install
WORKDIR /
RUN git clone https://github.com/s-tip/stip-common.git
RUN pip install -r ${SCRIPTS_DIR}/requirements_rs.txt -r ${SCRIPTS_DIR}/requirements_gv.txt -r ${SCRIPTS_DIR}/requirements_sns.txt

# stip-common setup
RUN mkdir -p ${INSTALL_DIR}/common
RUN ln -s ${COMMON_DIR}/src ${INSTALL_DIR}/common/src && ln -s ${COMMON_DIR}/img ${INSTALL_DIR}/common/img

# stip-rs setup
WORKDIR /stip-common
RUN git clone https://github.com/oasis-open/cti-pattern-matcher.git
WORKDIR /stip-common/cti-pattern-matcher/
RUN python setup.py install
WORKDIR /stip-common
RUN git clone https://github.com/s-tip/stip-rs.git
RUN mkdir -p ${INSTALL_DIR}/rs/bin ${INSTALL_DIR}/rs/community ${INSTALL_DIR}/rs/data ${INSTALL_DIR}/rs/staticfiles
RUN ln -s ${COMMON_DIR}/stip-rs/src ${INSTALL_DIR}/rs/src
RUN cp -p ${COMMON_DIR}/stip-rs/bin/* ${INSTALL_DIR}/rs/bin/ && cp -p ${COMMON_DIR}/stip-rs/data/* ${INSTALL_DIR}/rs/data/&& cp -p ${SCRIPTS_DIR}/env_rs ${INSTALL_DIR}/rs/.env
RUN echo $VERSION > ${INSTALL_DIR}/rs/version
RUN cp -p ${SCRIPTS_DIR}/apache/ports.conf /etc/apache2/ && cp -p ${SCRIPTS_DIR}/apache/stip-rs-ssl.conf /etc/apache2/sites-available
RUN sh -c "echo ServerName `hostname` > /etc/apache2/conf-available/fqdn.conf"
RUN a2enconf fqdn
RUN a2enmod ssl
RUN a2ensite stip-rs-ssl

# stip-gv setup
WORKDIR /stip-common
RUN git clone https://github.com/s-tip/stip-gv.git
RUN mkdir -p ${INSTALL_DIR}/gv/bin ${INSTALL_DIR}/gv/conf ${INSTALL_DIR}/gv/staticfiles ${INSTALL_DIR}/gv/stix
RUN ln -s ${COMMON_DIR}/stip-gv/src ${INSTALL_DIR}/gv/src
RUN cp -p ${COMMON_DIR}/stip-gv/bin/* ${INSTALL_DIR}/gv/bin/ && cp -p ${COMMON_DIR}/stip-gv/conf/* ${INSTALL_DIR}/gv/conf && cp -p ${SCRIPTS_DIR}/env_gv ${INSTALL_DIR}/gv/.env
RUN echo ${VERSION} > ${INSTALL_DIR}/gv/version
RUN cp -p ${SCRIPTS_DIR}/apache/stip-gv-ssl.conf /etc/apache2/sites-available
RUN a2ensite stip-gv-ssl

# stip-sns setup
WORKDIR /stip-common
RUN git clone https://github.com/s-tip/stip-sns.git
RUN mkdir -p ${INSTALL_DIR}/sns/bin ${INSTALL_DIR}/sns/media/cache ${INSTALL_DIR}/sns/staticfiles
RUN cp -pr ${COMMON_DIR}/stip-sns/bin ${INSTALL_DIR}/sns && cp -pr ${COMMON_DIR}/stip-sns/media ${INSTALL_DIR}/sns && cp -pr ${SCRIPTS_DIR}/env_sns ${INSTALL_DIR}/sns/.env && cp -p ${SCRIPTS_DIR}/apache/stip-sns-ssl.conf /etc/apache2/sites-available
RUN ln -s ${COMMON_DIR}/stip-sns/src ${INSTALL_DIR}/sns/src
RUN echo ${VERSION} > ${INSTALL_DIR}/sns/version
RUN a2ensite stip-sns-ssl

# modification for docker image
## mysql/mongo hostname
RUN echo "MYSQL_HOST=mysql" >> ${INSTALL_DIR}/rs/.env && echo "MYSQL_HOST=mysql" >> ${INSTALL_DIR}/gv/.env && echo "MYSQL_HOST=mysql" >> ${INSTALL_DIR}/sns/.env
RUN sed -i".org" 's/"host": "localhost"/"host": "mongo"/g' ${COMMON_DIR}/stip-rs/src/ctirs/fixtures/mongo.json
RUN sed -i".org" 's/"circl_mongo_host": "localhost"/"circl_mongo_host": "mongo"/g' ${COMMON_DIR}/stip-rs/src/ctirs/models/sns/config/fixture/sns_config.json
RUN sed -i".org" -e "s/DEFAULT_CIRCL_MONGO_HOST = 'localhost'/DEFAULT_CIRCL_MONGO_HOST = 'mongo'/g" -e "s/DEFAULT_ATTCK_MONGO_HOST = 'localhost'/DEFAULT_ATTCK_MONGO_HOST = 'mongo'/g" ${COMMON_DIR}/stip-rs/src/ctirs/models/sns/config/models.py

# create stip user
RUN adduser --disabled-password --gecos "" stip && adduser stip stip
RUN gpasswd -a stip sudo && gpasswd -a stip adm
RUN chown -R stip:stip ${INSTALL_DIR}
