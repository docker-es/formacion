---
title: Usando Docker compose.
description: Aprendemos Docker-compose y a usar las regras
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Introducción Docker-compose

Por ejemplo, si quisiera  implementara la misma aplicación, tendría que usar los siguientes comandos:

```
$ docker image pull redis:alpine
$ docker image pull russmckendrick/moby-counter
$ docker network create moby-counter
$ docker container run -d --name redis --network moby-counter redis:alpine$ docker container run -d --name moby-counter --network moby-counter -p8080:80 russmckendrick/moby-counter
```

Cada vez que necesitamos usar, aztualizar, o instalar los contenedores en otro entorno deveriamos guardar, y gestionar todas las instrucciones, ademas tenemos que tener encuenta que comandos debemos repetir, de ver si hace falta crear o no las redes de nuevo (por ejemplo).

Docker-compose  permite usar un archivo YAML para definir cómo le gustaría que se estructurara su aplicación de **múltiples** contenedores. Se tomaría el archivo YAML y se automatizaría el lanzamiento de los contenedores tal como se definió. La ventaja de esto fue que, debido a que era un archivo YAML, es muy fácil para los desarrolladores comenzar a enviar los archivos junto con sus archivos Docker dentro de sus bases de código.

Como ya se mencionó, Docker Compose usa un archivo YAML, normalmente denominado docker-compose.yml, para definir cómo debería verse su aplicación de múltiples contenedores.

TheDocker Compose la representación de la aplicación de dos contenedores que lanzamos en el Capítulo 4, Administración de Contenedores, y el Capítulo 5, Docker Machinees de la siguiente manera:

```
    version: "3"

    services: 
     redis:
      image: redis:alpine 
      volumes:
       - redis_data:/data
      restart: always

     mobycounter:
      depends_on:
       - redis
      image: russmckendrick/moby-counter
      ports:
       - "8080:80"
      restart: always
    
     volumes:
      redis_data:
```

Incluso sin trabajar a través de cada una de las líneas en el archivo, debe ser muy sencillo seguir adelante con lo que está sucediendo en. Para iniciar nuestra aplicación, simplemente cambie a la carpeta que contiene su archivo docker-compose.yml y ejecute lo siguiente: 

```
    $ docker-compose up
```

Como puede ver, desde las primeras líneas, Docker Compose hizo lo siguiente: 
- Creó una red llamada mobycounter_default usando el network1.driver predeterminado. En ninguna parte le pedimos a Docker Compose que hiciera esto.
- Creé un volumen llamado mobycounter_redis_data, nuevamente usando el controlador default2.driver. Esta vez le pedimos a Docker Compose que nos creara esta parte de la aplicación de múltiples contenedores.
- Lanzamos dos contenedores, uno llamado mobycounter_redis_1 y el segundo3.mobycounter_mobycounter_

La siguiente sección es donde se definen nuestros contenedores; Esta sección es la sección de servicios. Toma el siguiente formato:
```
    services:
    --> container name:
    ----> container options
    --> container name:
    ----> container options
```

La aplicación de votación nuestro archivo Docker Compose es un ejemplo bastante simple. Echemos un vistazo a un archivo Docker Compose más complejo y veamos cómo podemos introducir contenedores de construcción y varias redes. Encontrará una carpeta en el directorio capítulo 06 llamada aplicación de votación de ejemplo. Esta es una bifurcación de la aplicación de votación del Docker samplerepository oficial, que se puede encontrar en el [repositorio](https://github.com/dockersamples/)





# Docker Compose commands

- Detener el proyecto
```bash
docker-compose stop
# docker-compose stop <servicio>
```
- Iniciar el proyecto
``` bash
docker-compose start
# docker-compose start <servicio>
```

- Correr proyecto
``` bash
docker-compose up -d
# Nombrar el proyecto diferente a la carpeta:
# docker-compose -p mi-proy up -d
```
- Eliminar proyecto
``` bash
docker-compose down
# Eliminar los volumenes también:
# docker-compose --volumes
# Eliminar con un nombre de proyecto especifico:
# docker-compose -p mi-proy down
```
- Logs
``` bash
docker-compose logs
## docker-compose logs <servicio>
```

- Algunos comandos útiles de Docker
``` bash
## Ver contenedores
docker ps
## Ver imágenes
docker images
## Ingresar a un contenedores
 docker exec -it <nombre_contenedor> bash
## Ver logs de un contenedor
docker logs <nombre_contenedor> -f
## Eliminar imágenes huérfanas
docker rmi $(docker images -f "dangling=true" -q)

## Eliminar todas las imágenes de docker
docker rmi $(docker images -q)

## Eliminar todos los contenedores de docker
docker rm $(docker ps -a -q)
```
