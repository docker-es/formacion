---
title: Usando Docker  auto inicio con systemd
description: Aprendemos iniciar docker con systemd
author: Mario Ezquerro
tags: Docker, systemd
date_published: 2019-05-10
---

# Introducción Docker-compose auto arranque

Aunque podemos activar el inicio automatico con la opcion:

 $ docker create -it --restart=always --name contenedor imagen/contenedor

No nos permitira una gestion efectiva, tenemos que usar otra formula.


Podemos crear en  “/etc/systemd/system/docker-compose.service” añadimos el fichero **docker-compose.service**
con la siguiente contendio

#----------------------------------------------------------------------------------------------
```
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always
 
WorkingDirectory=/etc/docker/compose/%i
 
# Remove old containers, images and volumes
ExecStartPre=/usr/bin/docker-compose down -v
ExecStartPre=/usr/bin/docker-compose rm -fv
ExecStartPre=-/bin/bash -c 'docker volume ls -qf "name=%i_" | xargs docker volume rm'
ExecStartPre=-/bin/bash -c 'docker network ls -qf "name=%i_" | xargs docker network rm'
ExecStartPre=-/bin/bash -c 'docker ps -aqf "name=%i_*" | xargs docker rm'
 
# Compose up
ExecStart=/usr/bin/docker-compose up
 
# Compose down, remove containers and volumes
ExecStop=/usr/bin/docker-compose down -v
 
[Install]
WantedBy=multi-user.target
```
#----------------------------------------------------------------------------------------------

Y necesitamos que system ctl recarge los ficheros mediante el comando
```
	$ systemctl daemon-reload 
```
a partir de aqui

En  “/etc/docker/compose” meter crear los enlaces a los distintos  docker-compose con su directorio o un "ln -s /xxxxxx"
Por ejemplo:
```
ln -s /var/contenedores/netdata/ /etc/docker/compose/
```
