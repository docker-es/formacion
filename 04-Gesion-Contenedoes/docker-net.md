---
title: Usando Docker Redes
description: Aprendemos a usar docker y redes.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Gestion de red en  Contendores

## Redes terminos basicos

- PAM:Gestión de direcciones IP. El problema de lagestión de direcciones IP no  es  exclusiva  de  los  contenedores.  Estos  servicios  existen  en  las  redes tradicionales (el servicio DHCP es un IPAM). El ámbito de losde contenedores, hay dos métodos principales de IPAM: Los segmentos de direcciones IP basadosen CIDR o la asignación de una IP a cada contenedor. La asignación de una IP única en un grupo de servidores de contenedores,  se  resuelve medianteIPAM.

- Overlay:Red separada superpuesta sobre los dos o tres niveles existentes. La red usualmente tendrá supropio espacio de direcciones IP independiente, y su propio    enrutamiento.

- IPSesc:un protocolo de comunicación encriptado de un punto a punto, usualmente usado en el canal de datos en la red Overlay

- vxLAN:Es una solución conjunta propuesta por VMware, Cisco, RedHat etc.,   principalmente  para  resolver  el  problema  de  que  la  red  virtual  VLAN  es demasiado pequeña (4096). Debido a que cada cliente en la nube pública tiene una VPC diferente, 4096 redes virtuales no es suficiente. vxLAN, puede soportar  hasta  16millones  de  redes  virtuales,  suficiente    para  una  nube pública.

- Bridge:La conexión de las dos redes entre el equipo de red. En el contexto de  este  trabajo  fin  de  máster  nos  referimos  a  Bridges  de  Linux,  y concretamente    Docker0.

- BGP:Protocolo de enrutamiento de red autónomo en la red de troncal.


## Redes predeterminadas

$ docker network ls

- Bridge: La  red  bridge  representa  a  la  red  docker0  que  existe  en  todas  las instalaciones  de  Docker.  A  menos  que  use  docker  run -net=opción,  el daemon  de  Docker  conectará  el  contenedor  a  esta  red  de  forma predeterminada.  Usando  el  comando ifconfig en  el  host,  puede  verse  que este Bridge es parte de la pila de red del host.

    $ docker run -d -P --net=bridge nginx:1.9.1

- None: La  red  agrega  un  contenedor  a  una  pila  de  red  específica  del contenedor sin conectarlo a ningún interfaz de red.  $ Docker network inspect none

    $ docker run -d -P --net=none  nginx:1.9.1
    $ docker ps
    $ docker inspect <ID:docker> | grep IPAddress

- Host: La  red  agrega  un  contenedor  a  la  pila  de  red  del  host.  Se  puede comprobar que la configuración de red en el contenedor es la misma que el host.

    $ docker run -d --net=host ubuntu:14.04 tail -f /dev/null
    $ ip addr | grep -A 2 eth0:
    $ docker ps
    $ docker exec -it <id:docker> ip addr 

- 
## Comando de red docker

$ docker network --help

Crear una red de prueba de red
$ sudo docker network create network-test

$ sudo docker network inspect test-network

Inicamos un  contenedor y lo conectamos a nuestra rede de pruebas

$ sudo docker run -itd --name=test1 --net=network-test busybox
