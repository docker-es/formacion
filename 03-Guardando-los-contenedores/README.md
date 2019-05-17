---
title: Usando Docker  los volumenes y las imagenes
description: Aprendemos a usar docker.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# volumenes

Volúmen - Es la manera sencilla y predefinida para almacenar todos los ficheros (salvo unas pocas excepciones) de un contenedor, usará el espacio de nuestro equipo real y en “/var/lib/docker/volumes” creará una carpeta para cada contenedor.

Recordemos que en Linux/Unix “todo es un fichero”.

Podemos crear un volúmen con un nombre especial (que pueda facilitar su gestión) o dejar que el propio Docker le asigne un “hash”.

Ahora creamos un volúmen llamado “mis_datos”.

	$ docker volume create mis_datos

Veremos las propiedades de ese volúmen y donde está almacenado en nuestro equipo real “/var/lib/docker/volumes/mis_datos/_data”.

	$ # docker volume inspect my-vol

También podremos eliminar ese volúmen con la opción “rm”.

  $ docker volume rm mis_datos

Hasta que no borremos los contenedores que usen ese volúmen, no podremos borrarlo.

Ejemplo de creación de un nuevo contenedor usando ese volúmen, asociándolo a la carpeta “/var/lib/mysql” del contenedor.

  $ docker run -d -it --name ubu1 -v mis_datos:/var/lib/mysql ubuntu:17.10

Podemos crear un volúmen que sea usado por varios servidores webs mediante el parámetro “service”, solo se pueden montar volúmenes usando la sintaxis “–mount” en lugar del “-v”.

  $ docker service create -d --replicas=4 --name web-service --mount source=mis_datos,target=/app ubuntu:17.10

Es posible definir un volúmen de “solo lectura” agregando el modificador “:ro”. Volviendo a nuestro ejemplo anterior, ahora el volúmen montado “mis_datos” será de lectura solamente.

  $ docker run -d -it --name ubu1 -v mis_datos:/var/lib/mysql:ro ubuntu:17.10

## Volúmen bind / conectados 

- Es una manera de asociar una carpeta de nuestro equipo real y mapearla como una carpeta dentro de un contenedor.

Este sistema nos permite ver esa carpeta desde el contenedor y también desde nuestro equipo real.

Usar esos ficheros, copiarlos y además en caso de tener una solución de almacenamiento distribuido, poder tener múltiples copias.

Un caso de uso, es tener un portal web que queremos que se active en 20 contenedores en distintas partes del mundo, en nuestro equipo real creamos una carpeta y colocamos dentro todos los ficheros necesarios para ese web.

Replicamos la carpeta en otros servidores reales y se podrá usar por otros contenedores. (soluciones como hdfs o hadoop ayudan aún mas)

Como los ficheros que son creados en esa carpeta por el contenedor son visibles, podemos usarlos en un servicio en nuestro equipo real o en otros contenedores que lo monten.

Nuestro ejemplo será crear otro contenedor, llamado “ubu2”, donde una carpeta local de nuestro equipo real “/var/lib/mysql2” (2) se mapeará como “/var/lib/mysql” en el contenedor.

  $ docker run -d -it --name ubu2 -v /var/lib/mysql2:/var/lib/mysql ubuntu:17.10

También es posible usar una refencia relativa a la carpeta donde estémos parados al correr la creación del contenedor. (pwd)

  $ docker run -d -it --name ubu2 -v "$(pwd)"/datos:/var/lib/mysql ubuntu:17.10

El comando inspect del contenedor nos dará mas información sobre la unidad montada. En este caso la carpeta “/tmp/datos” se montará como “/var/lib/mysql” en el contenedor. Indica que el acceso es R/W (lectura y escritura).

  $ docker inspect ubu2

TMPFS (temporal file system) es una manera de montar carpetas temporales en un contenedor.

Usan la RAM del equipo y su contenido desaparecerá al parar el contenedor.

En caso de tener poca RAM, los ficheros se parasán al SWAP del equipo real.

Crearemos otro contenedor donde una determinada carpeta de un servidor web será “temporal”.

 $ docker run -d -it --name ubu3 --tmpfs /var/html/tempo ubuntu:17.10

Salvo que indiquemos una limitación de espacio usando el modificador “tmpfs-size=999bytes”, el espacio que pueden ocupar los ficheros es ilimitado (o limitado por el espacio disponible de RAM)

Este tipo de almacenamiento puede ser usado para almacenar ficheros de sesiones web, temporales o contenido que nos interese que se borre en cada rearranque del contenedor.






# Storing and Distributing Images

### Docker hub

Para acceder al repositoro de  imagenes docker [Enlace](https://hub.docker.com/)

		+ Exploracioin
		+ Organizacion
		+ Creando
		+ Perfiles y configuracion
		+ Home
## Automatización en GIT

Crear un Dockerfile en repositorio de git



## Pushing your own image

Necesitamos hacer login, usa el comando de login:
```
	$ docker login
```
Despues indroduccimos la password para estar activados al 100%

Ahora que usuario  está autorizado para interactuar con Docker Hub, si tenemos una imagen creada necesitamos hacer __push__, Primero debemos cear una imagen
```
	$ docker build --tag usuariologin/scratch-example:latest .
```
Si todo es coreccto ya podemos  realizar push
```
	$ docker image push usuariologin/scratch-example:latest
```
Entre lo importante, el nombre de "usuariologin" antes del __"/"__  despues el nombre y __":"__ La [versión](https://semver.org/lang/es/).


El nombre para poder usar [hub](https://hub.docker.com/) cuando se crea una imagen el "tag" puede ser distinto pero es necasario alinear los nombres de las imagens con los nombre de usuario en el repositorio de images de docker.

## Deploying tu propio repositorio de Docker

Para montar un repositorio de contenedores de Docker usaremos Docker, Docker Registry está distribuido, lo que hace que su implementación sea tan fácil como ejecutar los siguientes comandos:
```
	$ docker image pull registry:2
	$ docker container run -d -p 5000:5000 --name registry registry:2
```
Estos comandos te darán la instalación más básica de Docker Registry. Como se trabaja,   podemos hacer __pull__  y bajar una  una imagen de Alpine. Para empezar, necesitamos una imagen, así que vamos a agarrar la imagen de Alpine:
```
	$ docker image pull alpine
```
Ahora que tenemos una copia de la imagen de Alpine Linux, debemos enviarla a nuestro Registro local de Docker, que está disponible en ___localhost: 5000__ Para hacer esto, necesitamos etiquetar la imagen de Alpine Linux con la URL de nuestro registro local de Docker y también un nombre de imagen diferente:
```
	$ docker image tag alpine localhost:5000/localalpine
```
Ahora que tenemos nuestra imagen etiquetada, podemos enviarla a nuestro Docker Registry alojado localmente ejecutando el siguiente comando:
```
	$ docker image push localhost:5000/localalpine
```
Confirmamos la imagen con un: $ docker image ls

Ahora, hay muchas opciones y consideraciones cuando se trata de lanzar un Registro Docker. Como puedes imaginar, los más importantes están alrededor del almacenamiento. Dado que el único propósito de un registro es almacenar y distribuir imágenes, es importante que use algún nivel de almacenamiento persistente del sistema operativo. Docker Registry actualmente soporta lo siguiente:

+ Filesystem /var/lib/registry
+ Azure (Microsoft)
+ GCS (Google cloud storage)
+ S3 (Amazon)
+ Swift (OpenStack)

La información oficial esta documenta en la [web](https://docs.docker.com/registry/configuration/)

## Otros registradoes

+ [Redhat](https://access.redhat.com/containers/)
+ [Quay](https://quay.io/)

Ahora podrás subir tu imagen. Por ejemplo, el mío se puede extraer ejecutando el siguiente comando:
```
	$ docker image pull quay.io/marioezquerro/dockerfile-example
```
+ AWS
+ [Microbadger](https://microbadger.com/)

# Docker copia de seguridad de un contenedor y sus volumenes

Imaginemos que tenemos un contenedor en marcha, el cual tiene un volumen donde almacena la información sensible que deseamos sea persistente y no se pierda. Por ahora todo funciona a las mil maravillas, pero como Murphy siempre aparece sin avisar, estaría bien hacer una copia de seguridad por si acaso.

Bien, pues manos a la obra.

Supongamos que el escenario es el siguiente. El contenedor que está en ejecución se llama my-app y que el volumen en el que almacena los datos es vol-my-app.


## Crear copia de seguridad

La mejor manera de realizar una copia del contenedor es crear una imagen del mismo y así poder exportarla. Para ello hacemos uso de la opción commit y creamos una imagen basada en nuestro contenedor.

	$ docker commit my-app my-app_backup

Si ejecutamos docker images, aparte de las imágenes que ya tenemos, aparecerá otra con el nombre my-app_backup. Esta es la imagen de nuestro contenedor que podremos exportar a un archivo **.tar ** con docker save:

$ docker save -o my-app_backup.tar  my-app

Ahora ya tenemos un archivo my-app_backup.tar que contiene una imagen completa del contenedor.

Para evitar el síndrome de Diógenes digital, borramos la imagen del backup que ya tenemos en el archivo .tar.

	$ docker image rm my-app_backup

La copia de seguridad del volumen de datos es un poco más complicado. Desconozco si los desarrolladores de Docker tienen pensado programar algo parecido a docker save pero orientado a volumenes. De mientras, la manera más limpia de hacer una copia es mediante la ejecución de un contenedor.

Me explico. La idea es ejecutar un contenedor temporalmente en el que montaremos el volumen vol-my-app y por otro lado montaremos una carpeta local, en la que almacenaremos un .tar con el contenido del volumen. Dicho así suena raro, pero tiene su sentido.

Lo mejor es parar el contenedor y así evitar que los datos no se corrompan en el proceso de copiado:

	$ docker container stop my-app

Ahora ejecutamos un nuevo contenedor junto al comando para realizar el tar con el contenido del volumen:

	$ docker run --rm -it -v vol-my-app:/volume -v /tmp:/backup debian:stable-slim \
	tar -cf /backup/vol-my-app.tar -C /volume ./

El contenedor internamente creará una .tar con el contenido del volumen, y lo dejara en /backup, que realmente es /tmp fuera del contenedor. Por lo que la copia de seguridad estará en /tmp/vol-my-app.tar.

Volvemos a arrancar el contenedor original para que todo siga ejecutándose con normalidad:

	$ docker container start my-app

## Restaurar las copia de seguridad

Lo primero seria restaurar el contenedor:

	$ docker load < my-app_backup.tar

Ejecutando docker images veremos que se ha restaurado una imagen con el nombre my-app_backup. Como nuestro contenedor depende de un volumen de datos, deberemos recuperar esos datos antes de poner en marcha un contenedor de dicha imagen.

Para recuperar el volumen el proceso es muy parecido al que se utilizó para crear la copia de seguridad. En este caso en vez de crear un .tar lo que haremos en extraer el contenido de la copia, vol-my-app.tar, y meterlo en el volumen:

Creamos un volumen con el mismo nombre que el original:

	$ docker volume create vol-my-app

Insuflamos el contenido de .tar al nuevo volumen:

	$ docker run --rm -it -v vol-my-app:/volume -v /tmp:/backup debian:stable-slim \
	sh -c "rm -rf /volume/* /volume/..?* /volume/.[!.]* ; tar -C /volume/ -xf /backup/vol-my-app.tar”

Ahora que ya tenemos todo restaurado, solo hace falta arrancar un contenedor que se base en la imagen recuperada junto al volumen:

	$ docker run -d --restart unless-stopped --name **my-app** -v vol-my-app:/var/lib/my-app -p 8888:8888/tcp my-app_backup

Ejecutando **docker stats** podremos ver que nuestro contenedor ha vuelto a la vida después de morir súbitamente.