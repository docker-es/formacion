---
title: Usando Docker Namespaces
description: Aprendemos a usar docker.
author: Mario Ezquerro
tags: Docker, namespaces
date_published: 2019-05-10
---

# Usando Linux namespaces en el mapeo de volúmenes Docker.

No hace mucho, publico un artículo sobre el uso de sockets Unix con la ventana acoplable. Estas tomas se encontraban en volúmenes acoplables para que pudieran compartirse entre varios contenedores. La idea clave fue cambiar el UID y el GID del usuario que posee el socket en el contenedor para que coincidan con los del usuario que creó la imagen. El problema principal con este enfoque es que requiere que se compile en un contenedor con el usuario que lo ejecutará. Esto hace que la solución no sea portátil.

Con suerte, el kernel de Linux nos permite usar una alternativa para asignar la identificación de usuario dentro del contenedor a una identificación de usuario predecible fuera: espacios de nombres de identificación de usuario. Según wikipedia: Los espacios de nombres son una característica del kernel de Linux que aísla y virtualiza los recursos del sistema de una colección de procesos. Los ejemplos de recursos que pueden virtualizarse incluyen ID de proceso, nombres de host, ID de usuario, acceso a la red, comunicación entre procesos y sistemas de archivos. Los espacios de nombres son un aspecto fundamental de los contenedores en Linux.

Por ejemplo, gracias al espacio de nombres PID, un proceso que se ejecuta dentro de un contenedor puede "pensar" que tiene el PID 1 dentro de un contenedor, mientras que en realidad tiene otro. Lo mismo ocurre con el espacio de nombres de usuario: un usuario puede "pensar" que tiene el 0 uid (raíz) mientras que tiene el ID de 1000 usuarios (algún usuario estándar). Esto nos permitirá estar seguros de los archivos en una ventana acoplable que:

    Todos los archivos que pertenezcan al usuario root en el contenedor pertenecerán a un usuario del sistema que no sea root en el host.
    Todos los archivos que pertenecen a otros usuarios en el contenedor se asignarán a un uid predecible (más sobre esto último).

# Configurando Docker

Vamos a configurar docker para hacer todo eso.

Primero, debemos iniciar el demonio docker con la configuracion con el flag **--userns-remap USER**  asegurarnos de que el archivo de configuración del demonio docker (/etc/docker/daemon.json) contenga algo como:
```
{
   "userns-remap": "USER"
}
```

En ambos casos, el USUARIO debe ser un usuario válido del sistema (es decir, presente en */etc/passwd*).
No olvides reiniciar el demonio si tienes que editar el archivo.

# Configure the subordinate uid/gid

*subuid* y *subgid* se utilizan para especificar los identificadores de usuario/grupo que un usuario ordinario puede usar para configurar la asignación de identificadores en un espacio de nombres de usuario. Están escritas como: **username:id:count**. Por ejemplo, con **jenselme:100000:65536** significa que el usuario jenselme puede usar 65536 ID de usuario a partir de 100000.

Docker lo utilizará para reasignar correctamente el uid en el contenedor al host. Por ejemplo, con **jenselme:100000:65536**, un archivo con un uid de 33 en el contenedor, será un archivo con un uid de 100032 en el host. Y tendrás acceso a ese archivo.

Ahora que hemos visto la teoría, configurémosla adecuadamente. Primero, edite /etc/subuid y agregue (reemplace jenselme por su propio nombre de usuario):
```
jenselme:1000:1
jenselme:100000:65536
```
Debes poder entender la segunda línea. El primero está ahí para un propósito ligeramente diferente: asegúrate de que todos los archivos creados por root pertenezcan al usuario con uid 1000. Ese soy yo en mi máquina, por supuesto, debes usar tu uid (puedes obtenerlo con **id -u USUARIO** ). De lo contrario, pertenecerán a uid 100000.

Ahora, edite /etc/subgid y agregue (reemplace jenselme por su propio nombre de usuario):
```
jenselme:982:1
jenselme:100000:65536
```
La segunda línea es el nombre en ambos casos. No usé **jenselme:1000:1**, pero **jenselme:982:1**. En mi máquina, 982 es la interfaz gráfica de usuario del grupo docker (puede obtenerlo con el **getent group docker**). Esto significa que todos los archivos creados por root, me pertenecerán al grupo docker. Este "truco" puede ser útil si, por algún motivo, necesita compartir archivos con el demonio docker. Por ejemplo, software como traefic puede necesitar leer / escribir en el socket de la ventana acoplable. Por defecto, para este socket tenemos:

# Pruebas

Ahora que estamos listos, comencemos el demonio docker (o lo reiniciamos).
```
    # Create a user called "dockremap"
    $ sudo adduser dockremapper

    # Setup subuid and subgid
    $ sudo sh -c 'echo dockremap:400000:65536 > /etc/subuid'
    $ sudo sh -c 'echo dockremap:400000:65536 > /etc/subgid'

    $ vim /etc/docker/daemon.json 

    {
     "userns-remap": "dockremap"

    }

    And restart Docker:

    $ sudo /etc/init.d/docker restart
    
    O $ dockerd --userns-remap="dockremap:dockremap"

```

Nota para los usuarios de SELinux: debe agregar Z (mayúscula) al montar los volúmenes, como esto: **-v $(pwd)/test:/test/:Z**. De lo contrario, el contexto de SELinux no será correcto y no podrá acceder a los volúmenes desde el contenedor.[tip](https://www.jujens.eu/posts/2015/May/24/docker/#volumes).

Lo primero que debes notar es que si has descargado imágenes o creado contenedores, no los verás con **docker images** o **docker ps -a**. Esto se debe a que, cuando se habilita la reasignación de usuarios, todas las imágenes y contenedores se ubican en una subcarpeta dedicada. En mi máquina, es 
/var/lib/docker/1000.982.

Ahora que sabemos que esto es lo esperado, intentemos cosas. Ejecutar en algún lugar:
```
docker run -it -v "$(pwd)/test:/test/" nginx /bin/bash
```

Esto abrirá un indicador de bash como root en el contenedor. Vaya al volumen con **cd/test** y cree un archivo: **touch rootfile**. Si ejecuta un **ls -l** dentro del contenedor, debería ver algo como:
```
root@02a5bcc3452c:/test# ls -l
total 0
-rw-r--r--. 1 root   root 0 Jun 11 16:25 rootfile
```
Vamos a revisar el uid y gid para estar seguros:
```
root@02a5bcc3452c:/test# ls -ln
total 0
-rw-r--r--. 1000 Jun 11 16:25 rootfile
```
Now run ls -l in the host:
```
root@02a5bcc3452c:/test# ls -ln
total 0
-rw-r--r--. 1 jenselme docker 0 Jun 11 18:25 rootfile
```
Let's check the uid and guid:
```
$ ls -ln
total 0
-rw-r--r--. 1   1000 982 0 Jun 11 18:25 rootfile
```

Pertenece a la raíz y al nogroup como se esperaba (en el host, el archivo pertenece a jenselme: jenselme no jenselme: docker, de ahí el nogrupo, podría ejecutar chown jenselme: docker www-data-file-from-host para arreglar el gid) . Si verifica el uid y el gid, verá que también es el esperado.

Ahora cambiemos el propietario de www-data-file-from-host a 100032: 100032 con chown 100032: 100032 www-data-file-from-host (esto debe ejecutarse como root para evitar una operación no permitida). Te dejo comprobar el dueño, uid, gid en el contenedor. También puede verificar que el usuario de www-data pueda escribir en el archivo con echo 'test'> www-data-file-from-host.

Esto se ve bien, ¿no? Sin embargo encontré una mancha oscura en esto. Si intenta editar www-data-file-from-host o www-data-file en el host, fallará con el permiso denegado. Por lo que entiendo de la subuidad y subgid, esto no es normal. Si alguien tiene una explicación para esto, por favor deje un comentario. Veo dos soluciones para eso:

- Lo básico:

    - Cree un grupo con id 100032 (como root): groupadd -g 100032 docker-www-data
    - Agregue su nombre a este grupo (como root): usermod -aG docker-www-data jenselme
    - Desconecte vuelva a conectar o use el comando: newgrp docker-www-data para tener en cuenta este cambio.
    - Dé permiso de escritura al grupo en el contenedor: chmod g=rw www-data-file
    - Escribe en el archivo.

    Nota: No puede hacer nada acerca del usuario ya que solo puede tener un ID de usuario.

# Usando namespaces (con soporte en el kernel) ###############################

https://coderwall.com/p/s_ydlq/using-user-namespaces-on-docker

$DEFAULKERNEL=$(grubby --default-kernel)

grubby --info /boot/vmlinuz-3.10.0-514.6.1.el7.x86_64

sudo grubby --args="user_namespace.enable=1" \
    --update-kernel=/boot/vmlinuz-3.10.0-XXX.XX.X.el7.x86_64

Para todos dos kernel.....  grubby --update-kernel=ALL --args=console=ttyS0,115200

añade al default argumento
grubby --update-kernel=ALL --args="user_namespace.enable=1"
grubby --info DEFAULT

grubby --update-kernel=ALL --remove-args=console="user_namespace.enable=1"



## Create a user called "dockremap"
$ sudo adduser dockremap
dockremap:x:10056:10056:Linux container User,,:/home/dockremap:/bin/false

## Setup subuid and subgid
$ sudo sh -c 'echo dockremap:500000:65536 > /etc/subuid'
$ sudo sh -c 'echo dockremap:500000:65536 > /etc/subgid'

--userns-remap=default

--userns-remap=dockremap


ps ax | grep [d]ockerd
