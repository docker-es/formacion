---
title: Usando Docker  gestión de contenedores
description: Aprendemos a usar docker y sus comandos.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Gestion de Contendores

## Comandos Basicos

$ docker container run hello-world

following Dockerfile, it runs an executable called hello:
    FROM scratch
    COPY hello /
    CMD ["/hello"]
The hello executable prints the "Hello from Docker!" text to the Terminal and then the
process exits. As the process exits, our container also stops; this can be seen by running the
following:

$ docker container ls
$ docker container ls -a

De donde salen los [nombres](https://github.com/moby/moby/blob/master/pkg/namesgenerator/names-generator.go)

$ docker container run -d --name nginx-test -p 8080:80 nginx
$ docker container ls

$ docker container run --name nginx-foreground -p 9090:80 nginx
$ docker container ls -a #shows that you have two containers
$ docker container start nginx-test

## Attach

Para acceder a un contenedor usamos:
```
    $ docker container attach nginx-test
    You cannot attach to a stopped container, start it first
```
Pero tiene que estar en funcionanamiento :
$ docker container start nginx-test

## Exec

El comando __attach__ es útil si necesita conectarse al proceso que está ejecutando su contenedor, pero ¿qué sucede si necesita algo un poco más interactivo? Puedes usar el comando exec; esto genera un segundo proceso dentro del contenedor con el que puedes interactuar.

Por ejemplo, para ver el contenido del archivo /etc/ ebian_version, podemos ejecutar el siguiente comando:
```
    $ docker container exec nginx-test cat /etc/debian_version
```

## Limitación de recursos hardware

Mediante cgroups podemos limitar el acceso al hardware y no permitir a los contenedores más uso de hardware del que nosotros requerimos. En ello podemos limitar tanto en CPU como en RAM o red.
Limitación de CPU

Para la limitación de CPU podemos limitar de la siguiente forma:

    --cpu-shares en un limitación relativa a la cantidad de CPUs y sería un tanto %. Osea si tenemos 8 CPUs podemos limitar a 50 y usariamos 4.
    --cpu-period especifica un tiempo limite de uso de CPU y normalmente va asociado al siguiente parámetro.
    --cpu-quota indica la cuota de uso de un CPU.

Estos parámetros forman parte de la documentación de la limitación del CFS

Ejemplo:

>_ docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:14.04 /bin/bash

Si tenemos una CPU usará el 50% de la CPU cada 50ms
Limitación de RAM

En el caso de la RAM podemos hacer algo similar e indicarle al contenedor solo la RAM que necesita.

    --memory=1g limitaríamos el uso total a 1GB.
    --memory-reservation es un limite menos estricto y sería un mínimo de RAM. Esto nos sirve más para garantizar una memoria pero es necesario usar el otro límite.
    --memory-swap indica el uso que podemos usar en un contenedor de memoria SWAP.
    --memory-swappines especifica el porcentaje de memoria swap que puede usar sobre el total disponible. Como siempre el uso de swap no es nada recomendable por penalización del rendimiento.
    --kernel-memory es un limite a la memoria del kernel que puede usar el contenedor. Este tipo de límite no lo he usado mucho, pero conviene echarle un vistazo. Ayuda saber cuanta memoria tienes o más bien cuanto tienes en uso grep Slab /proc/meminfo. Es conveniente poner una media del 10% de la memoria que uses para el contenedor.

Como ejemplo crearemos un contenedor con limitantes tanto en CPU como en RAM, por lo que primero ejecutaremos:

    $ docker run -it --memory="512M" --cpu-shares 1024 --name container01 zokeber/centos /bin/sh

Con el parámetro  --cpu-shares 1024 le indicamos que use el 50% del CPU y con --memory="512M" solo 512M de RAM. Verificamos que estos recursos asignados son los correctos, con docker inspect:
```
##Verificamos la RAM:
$ docker inspect -f "{{ .HostConfig.Memory }}" container01
536870912
 
##Verificamos CPU:
docker inspect -f "{{ .HostConfig.CpuShares }}" container02
1024
```
Ahora bien, para asignarles mas RAM y que solo utilice el 25% de la CPU total, ejecutamos con docker update lo siguiente:

    $ docker update --memory="1024M" --cpu-shares 512 container01

En cambio, si queremos limitar solamente la RAM a 512M, ejecutamos:

    $ docker update --memory="512M" container01

Verificamos que el cambio a los recursos es el correcto:
```
##Verificamos la RAM:
$ docker inspect -f "{{ .HostConfig.Memory }}" container01
536870912
```

# Realizar copias de seguridad de contenedores
a estén encendidos o apagados, podemos realizar respaldos de seguridad de los contenedores. Utilizando la opción «export» empaquetará el contenido, generando un fichero con extensión «.tar» de la siguiente manera:

docker export -o fichero-resultante.tar nombre-contenedor
# o bien
docker export nombre-contenedor > fichero-resultante.tar

Veamos un ejemplo. Primero de todo listaré los contenedores que tengo en el servidor:

[root@centos7 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                     NAMES
bf4d12bda9bf        667fd02ce964        "docker-entrypoint..."   2 weeks ago         Exited (0) 2 weeks ago                             wordpress1
4ecc6becfcb6        b1fe0881b739        "docker-entrypoint..."   2 weeks ago         Up 9 seconds             0.0.0.0:33306->3306/tcp   mariadbtst10

Tal y como se observa, tengo un contenedor encendido que corresponde a un motor de base de datos MariaDB y otro apagado, que corresponde al CMS WordPress. Vamos a realizar la copia de seguridad del que tenemos encendido:
docker export mariadbtst10 > mariadbtst10.back.tar

Con el resultado:

[root@centos7 ~]# docker export mariadbtst10 > mariadbtst10.back.tar
[root@centos7 ~]# ls -ltr
-rw-r--r--  1 root root 401052160 ene  9 18:37 mariadbtst10.back.ta

## Restauración de copias de seguridad de contenedores

Hay que tener en cuenta, antes de nada, que no es posible restaurar el contenedor directamente, de forma automática. En cambio, sí podemos crear una imagen, a partir de un respaldo de un contenedor, mediante el parámetro «import» de la siguiente manera:

docker import fichero-backup.tar nombre-nueva-imagen

Veamos un ejemplo:

docker import mariadbtst10.back.tar copiamariadb

# Copias de seguridad de imágenes

También tenemos la posibilidad de realizar copias de seguridad de imágenes. El proceso se realiza al utilizar el parámetro ‘save‘, que empaquetará el contenido y generará un fichero con extensión «tar«, así:

docker save imagen > imagen.tar
#o bien
docker save -o imagen.tar imagen

## Restaurar copias de seguridad de imágenes
