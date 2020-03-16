---
title: Usando Docker introduccion
description: Aprendemos a usar docker.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Actualizar /etc/hosts con todos los contenedores docker que hay en ejecución

Si tenemos varios contenedores docker arrancados en nuestro ordenador. Muchas veces, nos interesará conectar con servicios corriendo dentro de cada uno de ellos. Algunos estarán lanzados simplemente con docker, otros con docker-compose, cada uno trabajando en un sistema distinto, y necesitamos una forma más o menos sencilla de acceder a cada uno de ellos.

Con un pequeño script podemos recorrer todos los contenedores, pedir la dirección IP de cada uno de ellos y añadirlas al nuestro archivo /etc/hosts de forma que este archivo se actualice automáticamente cada vez que lanzamos el comando.

El script

Por ejemplo lo llamamos docker_update_hosts.sh, y suele estar en /usr/local/bin. En realidad, es un enlace el que está ahí:
```
#!/bin/bash

function panic()
{
    echo "$@" >&2
    exit 1
}

if [ "`whoami`" != "root" ]
then
    panic "This program must be ran as root"
fi

# Clear /etc/hosts file
HEADER="# Added automatically by docker_update_hosts"
sed -i '/docker\.local$/d' /etc/hosts
sed -i "/$HEADER\$/d" /etc/hosts
# Remove empty lines at the end of file
sed -i  -e :a -e '/^\n*$/{$d;N;ba' -e '}' /etc/hosts

echo -e "\n$HEADER" >> /etc/hosts

IFS=$'\n' && ALLHOSTS=($( docker ps --format '{{.ID}} {{.Names}}'))

for line in ${ALLHOSTS[*]}; do
    IFS=" " read  -r ID NAME <<< "$line"
    IP="$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' $ID)"
    if [ -n "$IP" ]; then
        echo -e "$IP\t$NAME.docker.local" >> /etc/hosts
    fi
done
```

El script debe ser ejecutado como root, porque tiene que tener permiso para escribir en /etc/hosts. Para ello, en muchas distribuciones lo podremos ejecutar así:

    $ sudo docker_update_hosts

#El Fichero /etc/hosts

El script creará varios hosts llamados [contenedor].docker.local donde contenedor es el nombre de cada uno de nuestros contenedores. Al final, nos podremos juntar con algo como:

    172.17.0.8 myphp5.6-fpm.docker.local
    172.17.0.7 mongodb-testing.docker.local
    172.17.0.6 wp-plugin-test.docker.local
    172.17.0.5 myphp7.2-fpm.docker.local
    172.17.0.4 mariadb-proyectos.docker.local
    172.17.0.3 mariadb-testing.docker.local
    172.17.0.2 redis.docker.local



### Obtener la direccion IP privada de un contenedor

    $ docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" CONTENEDOR

### Para ver las ultimas, por ejemplo, 20 lineas de los logs del contenedor, ejecutamos:

```markdown
$ docker logs --follow --tail=20 CONTENEDOR
```
