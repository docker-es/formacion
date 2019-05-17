---
title: Usando Docker
description: Aprendemos a usar docker con ejemplos.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Ejemplos de Wordpress

[Ejemolos01](https://github.com/brunocascio/docker-espanol#usando-supervisor-y-en-un-%C3%BAnico-contenedor)
[Ejemplos02](https://neliosoftware.com/es/blog/como-usar-docker-para-desarrollar-en-wordpress/)






# Instalar Open

Cree una nueva red para la aplicación y la base de datos:

    $ sudo docker network create opencart-network

Cree un contenedor con la base de datos MariaDB en la nueva red:

    $ sudo docker run -d --rm --name=mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes -e MARIADB_USER=bn_opencart -e MARIADB_DATABASE=bitnami_opencart \
    --net=opencart-network bitnami/mariadb

Cree un contenedor con OpenCart en la misma red (sustituya AAA.BBB.CCC.DDD por la dirección ip de la máquina virtual):

    $ sudo docker run -d --rm --name=opencart \
    -e ALLOW_EMPTY_PASSWORD=yes -e OPENCART_DATABASE_NAME=bitnami_opencart -e OPENCART_HOST=localhost \
    -p 80:80 -p 443:443 --net=opencart-network bitnami/opencart

 http://<ip>/admin, donde <ip> es la dirección ip de la máquina virtual.

El usuario administrador predeterminado de OpenCart es **user** con contraseña **bitnami1**:

## Instalar phpMyAdmin

Para crear un contenedor phpMyAdmin:

    $ sudo docker run -d --name=phpmyadmin -p 8801:80 --net=opencart-network bitnami/phpmyadmin


El nombre de usuario de MariaDB creado para OpenCart y su contraseña se encuentran en el fichero de configuración de OpenCart (el usuario es siempre bn_opencart, pero la contraseña se genera al azar cada vez que se crea un contenedor.

El MariaDB instalado desde la imagen de Bitnami tiene usuario root sin contraseña.

## Instalar dos aplicaciones

- Cree la red:

    $ sudo docker network create wps

- Cree el contenedor de MariaDB:

    $ sudo docker run -d --name=wps-mariadb -e ALLOW_EMPTY_PASSWORD=yes --net=wps bitnami/mariadb

- Cree el contenedor de phpMyAdmin:

    $ sudo docker run -d --name=wps-pma -p 8801:80 -e DATABASE_HOST=wps-mariadb --net=wps bitnami/phpmyadmin

Entre en phpMyAdmin y cree los usuarios wp1 y wp2
- Cree el primer contenedor de WordPress:
```
    sudo docker run -d --name=wp1 -p 8802:80 -p 4432:443 \
    -e MARIADB_HOST=wps-mariadb -e WORDPRESS_DATABASE_NAME=wp1 -e WORDPRESS_DATABASE_USER=wp1   -WORDPRESS_DATABASE_PASSWORD=wp1 -e \
    WORDPRESS_USERNAME=barto -e WORDPRESS_PASSWORD=barto --net=wps bitnami/wordpress
```
- Cree el segundo contenedor de WordPress:
```
    sudo docker run -d --name=wp2 -p 8803:80 -p 4433:443 \
    -e MARIADB_HOST=wps-mariadb -e WORDPRESS_DATABASE_NAME=wp2 -e WORDPRESS_DATABASE_USER=wp2 -e WORDPRESS_DATABASE_PASSWORD=wp2 -e \
    WORDPRESS_USERNAME=barto -e WORDPRESS_PASSWORD=barto --net=wps bitnami/wordpress
```
# Ejemplo PHP

## Dockerfile
Viene [de](https://poesiabinaria.net/2019/01/utilizar-php-desde-contenedores-docker-tanto-forma-local-produccion/)

Vamos a basarnos en las imágenes oficiales de php, en este caso php7.2. Y vamos a instalar por defecto extensiones como xml, zip, curl, gettext o mcrypt (esta última debemos instalarla desde pecl.
En los contenedores vamos a vincular /var/www con un directorio local, que puede estar por ejemplo, en la $HOME del usuario actual, donde podremos tener nuestros desarrollos. Y, por otro lado, la configuración de PHP también la vincularemos con un directorio fuera del contenedor, por si tenemos que hacer cambios en algún momento. En teoría no deberíamos permitir esto, la configuración debería ser siempre fija… pero ya nos conocemos y siempre surge algo, lo mismo tenemos que elevar la memoria en algún momento, cambiar alguna directiva o alguna configuración extra.

Dockerfile:
```
FROM php:7.2-fpm
ARG HOSTUID
ENV BUILD_DEPS="autoconf file gcc g++ libc-dev make pkg-config re2c libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng-dev libssl-dev libc-client-dev libkrb5-dev zlib1g-dev libicu-dev libldap-dev libxml2-dev libxslt-dev libcurl4-openssl-dev libpq-dev libsqlite3-dev" \
    ETC_DIR="/usr/local/etc" \
    ETC_BACKUP_DIR="/usr/local/etc_backup"

RUN apt-get update && apt-get install -y less \
    procps \
    git \
    && pecl install redis \
    && pecl install xdebug \
    && docker-php-ext-enable redis xdebug \
    && apt-get install -y $BUILD_DEPS \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install -j$(nproc) imap \
    && docker-php-ext-install -j$(nproc) bcmath \
    && docker-php-ext-install -j$(nproc) calendar \
    && docker-php-ext-install -j$(nproc) exif \
    && docker-php-ext-install -j$(nproc) fileinfo \
    && docker-php-ext-install -j$(nproc) ftp \
    && docker-php-ext-install -j$(nproc) gettext \
    && docker-php-ext-install -j$(nproc) hash \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-install -j$(nproc) json \
    && docker-php-ext-install -j$(nproc) ldap \
    && docker-php-ext-install -j$(nproc) sysvshm \
    && docker-php-ext-install -j$(nproc) sysvsem \
    && docker-php-ext-install -j$(nproc) xml \
    && docker-php-ext-install -j$(nproc) zip \
    && docker-php-ext-install -j$(nproc) xsl \
    && docker-php-ext-install -j$(nproc) phar \
    && docker-php-ext-install -j$(nproc) ctype \
    && docker-php-ext-install -j$(nproc) curl \
    && docker-php-ext-install -j$(nproc) dom \
    && docker-php-ext-install -j$(nproc) soap \
    && docker-php-ext-install -j$(nproc) mbstring \
    && docker-php-ext-install -j$(nproc) posix \
    && docker-php-ext-install -j$(nproc) pdo_pgsql \
    && docker-php-ext-install -j$(nproc) pdo_sqlite \
    && docker-php-ext-install -j$(nproc) pdo_mysql \
    && yes | pecl install "channel://pecl.php.net/mcrypt-1.0.1" \
    && { \
    echo 'extension=mcrypt.so'; \
    } > $PHP_INI_DIR/conf.d/pecl-mcrypt.ini \
    && echo "Fin de instalaciones"
COPY docker-entry.sh /usr/local/binx

RUN mv $ETC_DIR $ETC_BACKUP_DIR \
    && chmod +x /usr/local/bin/docker-entry.sh \
    && rm /etc/localtime

RUN useradd -s /bin/bash -d /var/www -u $HOSTUID user

ENTRYPOINT ["/usr/local/bin/docker-entry.sh"]
CMD ["php-fpm"]
```

También tendremos un archivo **docker-entry.sh**:
```
#!/bin/bash
echo "Iniciando contenedor"

VERSION="7.2"
CONFFILE=/etc/php/$VERSION/fpm/php-fpm.conf
DOCKERIP=$(hostname --ip-address)

if [ $(ls $ETC_DIR | wc -l) -eq 0 ]; then
        echo "Copiando configuración por defecto"
        cp -r "$ETC_BACKUP_DIR"/* "$ETC_DIR"
fi

/usr/local/bin/docker-php-entrypoint $@
```
Para construir la máquina podemos utilizar esto:

    $ docker build -t myphp7.2-fpm --build-arg HOSTUID=”$(id -u)” --cpuset-cpus=”0-7″ .

Utilizo cpuset-cpus para delimitar los núcleos que vamos a utilizar para compilar los módulos. Esto puede tardar un poco y, si tenemos varios núcleos, puede interesarnos utilizar uno o dos, y mientras se construye PHP, utilizar el ordenador para navegar por Internet o algo así. Yo suelo crear un archivo build.sh con esa misma línea de antes.

Ahora, tendremos unos argumentos a la hora de lanzar el contenedor (run.sh)
```
#!/bin/bash
readonly SCRIPTPATH="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"
readonly WWWPATH="$HOME/www"

GATEWAY=""
if [ -n "$(which dnsmasq)" ]; then
    if [ -z "$(pidof dnsmasq)" ]; then
        sudo dnsmasq --bind-interfaces
    fi
    GATEWAY="--dns $(ip addr show docker0 | grep -Po 'inet \K[\d.]+')"
fi

pushd $SCRIPTPATH > /dev/null
docker run --rm --name myphp7.2-fpm -v /etc/localtime:/etc/localtime:ro -v $WWWPATH:/var/www:rw -v $(pwd)/conf:/usr/local/etc/:rw $GATEWAY --user www-data --cpuset-cpus="7" -d myphp7.2-fpm
```
Este archivo podremos reescribirlo dependiendo de nuestra configuración local. En mi ordenador, utilizo dnsmasq como dns en la máquina host, de forma que si modifico mi /etc/hosts, pueda acceder a dichos nombres desde mi contenedor PHP. Además, es conveniente editar en este archivo la variable WWWPATH donde estableceremos la ruta base desde la que tendremos todos nuestros archivos PHP, a partir de la que serviremos con FPM los archivos.

## Configurando un servidor web
Este PHP con FPM debemos configurarlo en un servidor web, para ello utilizaremos proxy_fcgi dejando el VirtualHost más o menos así (no he puesto configuración de SSL porque estoy en local, aunque también podríamos configurarla):
```
<VirtualHost *:80>
    ServerName prueba_de_mi_web.local

    ServerAdmin webmaster@localhost
    Define webpath /prueba_de_mi_web.com
    DocumentRoot /home/gaspy/www/${webpath}
    <Directory /home/gaspy/www/${webpath}/>
            Options +FollowSymLinks
        AllowOverride all
    </Directory>

    <IfModule proxy_fcgi_module>
         ProxyPassMatch "^/(.*\.ph(p[3457]?|t|tml))$" "fcgi://myphp7.2-fpm.docker.local:9000/var/www/${webpath}/$1"
         DirectoryIndex index.html index.php
    </IfModule>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Mi $HOME, en mi ordenador es /home/user/ y en www almaceno todo el código de aplicaciones web. En mi /etc/hosts he hecho una entrada que apunta a la IP del contenedor. Para ello puedo utilizar este script que actualiza /etc/hosts con los dockers que hay en ejecución en este momento.



# Entorno de Ventanas dentro de Docker

En el modo de ejecución más  simple, las aplicaciones visuales creadas dentro de un contenedor Docker,  no pueden ser mostradas en el sistema no virtualizado. Para lograr que esto funcione se requiere compartir el sistema de ventanas ya sea mediante el formato X11 nativo o mediante algún sistema de manejo remoto como VNC. El siguiente ejemplo utiliza la primera opción dada su simplicidad.

En primer lugar, se define un archivo Dockerfile que instale el servidor X11 y algunas herramientas para poder crear un programa GTK+ dentro del contenedor:

```
FROM ubuntu
RUN apt-get update
RUN apt-get install -qqy x11-apps
RUN apt-get install -qqy build-essential
RUN apt-get install -qqy cmake libgtk2.0-dev pkg-config
ENV DISPLAY :0
```

Luego, creamos el contenedor ejecutando:

sudo chmod +x build.sh
./build.sh
Este script simplemente ejecuta ‘sudo docker build -t sandbox .’, creando una imagen nueva llamada sandbox. Luego, para iniciar el contenedor es necesario definir ciertas variables de entorno con el comando e y un mapeo de directorios para montar nuestro código fuente dentro del servidor. El siguiente script realiza los pasos:

sudo chmod +x run.sh
./run.sh
Por último, ya dentro del contenedor, compilamos un pequeño programa GTK+ y lanzamos su ventana. La ventana será vista en nuestro sistema Linux gracias al mapeo del servidor X:
