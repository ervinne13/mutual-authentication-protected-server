FROM ubuntu:16.04

ARG COMPOSE_VOLUME_PATH

RUN apt-get -y --fix-missing update
# Install apache, PHP, and supplimentary programs, openssh-server, curl, and lynx-cur
RUN apt-get -y --force-yes install software-properties-common python-software-properties

# Setup for PHP
RUN LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php
RUN apt-get -y --fix-missing update
RUN apt-cache pkgnames | grep php7.2
RUN apt-get -y --fix-missing update
RUN apt-get -y install nginx \
	php7.2 \
	php7.2-fpm \
	php7.2-cli \
	php7.2-gd \
	php7.2-intl \
	php7.2-common \
	php7.2-mysql \
	php7.2-curl \
	curl \
	lynx-cur \
	php7.2-dom \
	zip \
	unzip \
	php7.2-xml \
	php7.2-zip \
	php7.2-mbstring \
	php7.2-json \
	php7.2-opcache \
	php7.2-dev \
	php7.2-sqlite \
	git \
	vim

RUN apt-cache search php7.2

# Back to PHP: Composer
RUN apt-get -y --fix-missing update
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php
RUN curl -s https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

RUN mkdir -p ${COMPOSE_VOLUME_PATH}

RUN echo "cgi.fix_pathinfo=0" >> /etc/php/7.2/fpm/php.ini
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# Setup Root CA
RUN mkdir -p /root/certs/root-ca
ADD ./certs/root-ca/openssl.cnf /root/certs/root-ca
RUN openssl req -config /root/certs/root-ca/openssl.cnf -newkey rsa:2048 -nodes -keyform PEM \
	-keyout /root/certs/root-ca/ca.key -x509 -days 3650 \
	-extensions certauth -outform PEM -out /root/certs/root-ca/ca.crt

# Create a request from client to generate client certificate
RUN mkdir -p /root/certs/client
ADD ./certs/client/openssl.cnf /root/certs/client
ADD ./certs/client/instructions.txt /root/certs/client
RUN openssl genrsa -out /root/certs/client/client.key 2048
RUN openssl req -config /root/certs/client/openssl.cnf -new \
	-key /root/certs/client/client.key -out /root/certs/client/client.req

# Sign the client's request to create certificate
RUN openssl x509 -req -in /root/certs/client/client.req \
	-CA /root/certs/root-ca/ca.crt -CAkey /root/certs/root-ca/ca.key \
	-extfile /root/certs/client/openssl.cnf -extensions client -days 365 -outform PEM \
	-out /root/certs/client/client.crt -CAcreateserial -CAserial /root/certs/client/serial.seq

# Setup Server Cert
RUN mkdir -p /root/certs/server
ADD ./certs/server/openssl.cnf /root/certs/server
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
	-config /root/certs/server/openssl.cnf \
	-keyout /root/certs/server/server.key -out /root/certs/server/server.crt

# Setup NGINX
RUN rm /etc/nginx/sites-enabled/default
ADD ./server-nginx.conf /etc/nginx/sites-available/
RUN ln -s /etc/nginx/sites-available/server-nginx.conf /etc/nginx/sites-enabled/server

ADD ./server-https-nginx.conf /etc/nginx/sites-available/
RUN ln -s /etc/nginx/sites-available/server-https-nginx.conf /etc/nginx/sites-enabled/server-https

COPY start.sh /

RUN chmod +x /start.sh

EXPOSE 80

CMD ["/start.sh"]
