---
title: Usando Docker Namespaces
description: Aprendemos a usar docker.
author: Mario Ezquerro
tags: Docker, namespaces
date_published: 2019-05-10
---

# Docker Machine

## Introducción

Docker está presente en un alto número de ecosistemas cloud públicos y soportado por múltiples plataformas y funciona en múltiples host con sus diferente OS.

Poder gestionar con un único comando con la interacción  mínima de un usuario nos ayuda a simplificar y unificar la experiencia del usuario.
Este comando permite pasar instrucción sobre los scripts para lanzar Docker simplificando el uso de los contenedores.

## Instalación de Docker Machine

Para instalar la última versión (0.7.0) de esta herramienta ejecutamos:
```
$ curl -L https://github.com/docker/machine/releases/download/v0.7.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine
```

Y comprobamos la instalación:

$ docker-machine -version
docker-machine version 0.7.0, build a650a40

## Utilizando Docker Machine con VirtualBox

Vamos a ver distintos ejemplos de Docker Machine, utilizando distintos drivers. Primero vamos a utilizar el driver de VirtualBox que nos permitirá crear una máquina virtual con Docker Engine en un ordenador donde tengamos instalado VirtualBox. Para ello ejecutamos la siguiente instrucción:
```
$ docker-machine create -d virtualbox nodo1
```

Esta instrucción va  a crear una nueva máquina (nodo1) donde se va a instalar una distribución Linux llamada node1. Utilizando el driver de VirtualBox podemos indicar las características de la máquina que vamos a crear por medio de parámetros, podemos indicar las características hardware (--virtualbox-memory, --virtualbox-disk-size, …) Para más información de los parámetros que podemos usar puedes mirar la [documentación del driver](https://docs.docker.com/machine/drivers/virtualbox/).

Docker Machine crea una única SSH-Keyy para la máquina virtual, esto significa que tu podrás acceder usando SSH mas tarde, una vez que tu maquina esta iniciada del todo, y entonces Docker Machine realizara la conexión con la maquina virtual.


Una vez creada la máquina podemos comprobar que lo tenemos gestionados por Docker Machine, para ello:

$ docker-machine ls

Para conectarnos desde nuestro "cliente docker"  al "Docker server Engine" de la nueva máquina necesitamos declarar las variables de entornos adecuadas, para obtener las variables de entorno de esta máquina podemos ejecutar:

$ docker-machine env nodo1


A partir de ahora, y utilizando el cliente docker, estaremos trabajando con el Docker Engine de nodo1:

$ docker-machine ssh nodo1

$ docker run -d -p 80:5000 training/webapp python app.py

Y para acceder al contenedor deberíamos utilizar la ip del servidor docker, que la podemos obtener:

$ docker-machine ip nodo1

Otras opciones de docker-machine que podemos utilizar son:

- inspect: Nos devuelve información de una máquina.
- ssh, scp: Nos permite acceder por ssh y copiar ficheros a una determinada máquina.
- start, stop, restart, status: Podemos controlar una máquina.
- rm: Es la opción que borra una máquina de la base de datos de Docker Machine. Con determinados drivers también elimina la máquina.
- upgrade: Actualiza a la última versión de docker la máquina indicada.


# Use Docker Machine to provision hosts on cloud providers

## Examples
### Digital Ocean

For Digital Ocean, this command creates a Droplet (cloud host) called “docker-sandbox”.

$ docker-machine create --driver digitalocean --digitalocean-access-token xxxxx docker-sandbox
For a step-by-step guide on using Machine to create Docker hosts on Digital Ocean, see the Digital Ocean Example.

### Amazon Web Services (AWS)
For AWS EC2, this command creates an instance called “aws-sandbox”:

$ docker-machine create --driver amazonec2 --amazonec2-access-key AKI******* --amazonec2-secret-key 8T93C*******  aws-sandbox

Otros [drivers](https://github.com/docker/docker.github.io/blob/master/machine/AVAILABLE_DRIVER_PLUGINS.md)



#

Let's check the version of Docker that is installed by running this command:



$ docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso docker-rancher

$ eval $(docker-machine env docker-rancher) ->toda la gestion pasa a esta maquina

$ docker $(docker-machine config docker-rancher) version

$ docker $(docker-machine config docker-rancher) run -d -p 80:5000 training/webapp python app.py