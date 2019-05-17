---
title: Usando Docker  Swarm
description: Aprendemos a usar docker swarm
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Docker Swarm

Docker 1.12 introdujo el modo Docker Swarm. Esto introdujo toda la funcionalidad que estaba disponible en el Docker Swarm autónomo en el motor central de Docker junto con características adicionales. Como estamos cubriendo la versión 1.13 de Docker y la anterior en este libro, usaremos el modo de enjambre de Docker, que para el resto del capítulo nos referiremos a enjambre de Docker.

Instalación de Docker Swarm Como ya está ejecutando una versión de Docker con soporte incorporado para Docker Swarm, no hay nada que deba hacer para instalar Docker Swarm; puede verificar que DockerSwarm está disponible en su instalación ejecutando el siguiente comando:

$ docker swarm --help 

Debe ver algo que se parece a la siguiente salida de Terminal cuando ejecuta el comando: 

[Imagen de salida de terminal docker swarm]()


la sobrecarga de la CPU de programación y comunicación dentro de Swarm es realmente pequeña. Gracias a eso, los administradores pueden ser (y son, por defecto) nodos de trabajo al mismo tiempo. Si va a trabajar en grupos muy grandes (más de 1000 nodos), los administradores requieren muchos más recursos, pero es insignificante con instalaciones de tamaño pequeño a mediano. Aquí puede leer sobre Swarm3k, un experimento de 4700 nodos Docker Swarm cluster.

Ejemplo usando docker swarm
```
    # let's create new service 
    docker service create \
      --image nginx \
      --replicas 2 \
      nginx 

    # ... update service ...
    docker service update \
      --image nginx:alpine \
      nginx 

    # ... and remove
    docker service rm nginx

    # but usually it's better to scale down
    docker service scale nginx=0

    # you can also scale up
    docker service scale nginx=5

    # show all services
    docker service ls

    # show containers of service with status
    docker service ps nginx

    # detailed info
    docker service inspect nginx
```

Es fácil realizar una implementación de 0 tiempos de inactividad.
También es perfecto para la implementación continua.
```
# lets build new version and push to the registry
    docker build -t hub.docker.com/image .
    docker push hub.docker.com/image
    
    # and now just update (on a master node)
    docker service update --image hub.docker.com/image servicio
```


# Roles en Docker Swarm

¿Qué funciones están involucradas con Docker Swarm? Echemos un vistazo a las dos funciones que un host puede asumir cuando se ejecuta dentro de un clúster Docker Swarm:

- **Swarm manager:** El manager del swarm es el host que es el punto de administración central para todos los hosts del swarm. El administrador swarm es donde se emite todos sus comandos para controlar esos nodos. Puede alternar entre los nodos, unirse a los nodos, eliminar los nodos y manipular esos hosts. Cada clúster puede ejecutar varios manager swarm. Para la producción, se recomienda que utilice un mínimo de cinco manager de Swarm: esto significaría que nuestro clúster puede tomar como máximo dos fallos de los nodos del administrador de Swarm antes de que comience a tener algún error. Los manager swarm utilizan el algoritmo de votos para mantener un estado consistente en todos los nodos de gestores.

- **Swarm worker:** son aquellos que ejecutan los contenedores Docker. Los worker son manejados por el swarm manager

# Swarm Master
Como he comentado anteriormente vamos a usar el sistema de descubrimiento por defecto, por lo que necesitamos un token, lo conseguimos ejecutando el siguiente comando:
```
sid=$(docker run swarm create) -> docker swarm init --advertise-addr <IP-GESTION>
echo $sid
```
Guardamos el token en la variable sid. Procedemos a crear el master y los nodos.
```
$ docker swarm join --token <ID-TOKEN> <IP_MASTER>:2377
# Si no nos acordamos del comando, ejecutamos en el master:
$ docker swarm join-token worker
```
Ahora vamos a ver el conjunto de maquinas que hemos creado anteriormente con 
```
# Para ver el estado del nodo
$ docker info

# Par ver los nodos desplegados
$ docker node ls
```

### Desplegamos un servicio

$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```
#ejemplo 2
$ docker service create --name my-web1 --publish 8081:80 --replicas 2 nginx
$ docker service scale my-web1=3
```
Comprobamos el listado de servicios que se están ejecutando

    $ docker service ls

Inspeccionamos el servicio creado con el comando docker service inspect –pretty <SERVICE-ID>
(Si ejecutamos el mismo comando sin el parámetro –pretty obtenemos la información en fomato json)

    $ docker service inspect --pretty helloworld

En este primer ejemplo hemos desplegado un servicio con 1 instancia del contenedor alpine. De una forma muy sencilla vamos a aumentar el número de instancias de las tareas que ejecutan el servicio con el comando docker service scale <SERVICE-ID>=<NUMBER-OF-TASK

    $ docker service scale helloworld=5

Ahora vamos a ver como reducir el número de instancias de una forma igual de sencilla

    $ docker service scale helloworld=3
    $ docker service ls
    $ docker service ps helloworld

Para finalizar con este ejemplo vamos a borrar el servicio con docker service rm <SERVICE-ID>

    $ docker service rm helloworld


### Actualizaciones del servicio

Vamos a crear un nuevo servicio y ver como es el proceso de actualización de las versiones de los contenedores que se ejecutan en el servicio.
```
docker service create \
  --replicas 3 \
  --name grafana \
  --update-delay 10s \
  grafana/grafana:5.0.0
```
El proceso de actualización es el siguiente:

El manager planifica la actualización de una de las tareas (o varias si se ha indicado un valor mayor que 1 para el paralelismo de las actualizaciones)

- Se para el contenedor de la tarea
- Se actualiza el contenedor
- Se inicia el contenedor actualizado con la nueva imagen

Cuando esta tarea de actualización devuelve el estado RUNNING, se planifica la actualización de la siguiente tarea tras el tiempo configurado de espera
Si alguna de las tareas devuelve el estado FAILED, se pausan las tareas de actualización. Para reiniciar una tarea pausada ejecutamos docker service update <SERVICE-ID>
Comprobamos e inspeccionamos el estado del servicio

Vamos a actualizar el servicio a la versión 5.0.1 de grafana:

    $ docker service update --image grafana/grafana:5.0.1 grafana

Durante la ejecución podemos ver el estado de las tareas:

    $ docker service ps grafana -> Repetir comando....

### Poner un nodo en modo mantenimiento

Para poner un nodo en modo mantenimiento ejecutamos: docker node update –availability drain <NODE-ID>

    $ docker node update --availability drain srv-dck-003


# Creando y añadiendo al servicio
--------------------------------
When you deploy a stack like this: (Podemos usar **$ curl -sL https://git.io/vNodz > docker-compose.yml**)
```
 $ docker stack deploy --with-registry-auth --compose-file docker-compose.yml my-stack
 $ docker stack services --format 'table {{.Name}}\t{{.Mode}}\t{{.Replicas}}\t{{.Image}}\t{{.Ports}}' my-stack
 $ docker stack ps --format 'table {{.Name}}\t{{.Node}}\t{{.CurrentState}}' my-stack
 # Clear
 $ docker stack rm my-stack
```
It creates a network called my-stack_default

So to launch a service that can communicate with services that are in that stack you need to launch them like this:

$ docker service create -name statefulservice --network my-stack_default reponame/imagename

#  Promoviendo un nodo worker

Digamos que desea realizar algún mantenimiento en su nodo de master único, pero desea mantener la disponibilidad de su clúster.
No hay problema; puede promover un nodo de trabajo a un nodo de administrador. Mientras tenemos nuestro clúster local de tres nodos en funcionamiento, promovamos swarm-worker01 para que sea un nuevo administrador. Para hacer esto, ejecute lo siguiente:

    $ docker node promot swarm-worker01

Debe recibir un mensaje que confirme que su nodo se ha promocionado  inmediatamente después de ejecutar el comando:

    Node swarm-worker01 promoted to a manager in the swarm

Haga una lista de los nodos ejecutando esto: 

    $ docker node ls

# Demotando un nodo gestor

Es posible que ya haya juntado dos y dos, pero para degradar un nodo de administrador a un trabajador, simplemente debe ejecutar esto:

    $ docker node degradar swarm-manager

Recibirá una respuesta inmediata diciendo lo siguiente: Manager swarm-manager degradado en el swarm. Ahora hemos degradado nuestro nodo, y puede verificar el estado de los nodos dentro del clúster ejecutando este comando:
    
    $ docker node ls

Como su cliente local de Docker todavía apunta hacia, el nodo recién degradado, recibirá un mensaje que dice lo siguiente : 

    Respuesta de error del daemon: Este nodo no es un administrador de enjambres. Los nodos de trabajo no se pueden utilizar para ver o modificar el estado del clúster. Ejecute este comando en un nodo de Amanager o promocione el nodo actual a un administrador.

Como ya hemos aprendido, es fácil actualizar nuestra configuración de cliente local para comunicarse con otros nodos mediante Docker Machine. Para dirigir su cliente local al nodo newmanager, ejecute lo siguiente:

    $ eval $ (docker-machine env swarm-worker01)

# Draining un nodo

Para eliminar temporalmente un nodo de nuestro clúster, de modo que podamos realizar el mantenimiento, necesitamos establecer el estado del nodo en Drain. Echemos un vistazo a drenar nuestro nodo administrador anterior. Para hacer esto, necesitamos ejecutar el siguiente comando:

    $ docker node update --availability drain swarm-manager

Esto detendrá cualquier tarea nueva, como el lanzamiento o la ejecución de nuevos contenedores contra el nodo que estamos drenando. Una vez que se hayan bloqueado las nuevas tareas, todas las tareas en ejecución se migrarán desde el nodo que estamos drenando a los nodos con un estado ACTIVO.

Como puede ver en la siguiente salida de Terminal, la lista de nodos ahora muestra que swarm-manager aparece como Drain en la columna de DISPONIBILIDAD:

Debe mostrar que el nodo tiene una DISPONIBILIDAD de pausa. Para volver a agregar el nodo al clúster, simplemente cambie la DISPONIBILIDAD a activo ejecutando esto:

    $ docker node update --availability active swarm-manager


# Usando Docker-machine

(Usando como ejemplo:)[https://www.adictosaltrabajo.com/2015/12/03/docker-compose-machine-y-swarm/]

Inicamos la primera maquina

$ docker-machine create --driver virtualbox dev

Confirmamos
$ docker-machine ls

Localizamos IP
$ docker-machine <IP_MASTER>

Accedemos a la maquina
$ docker-machine ssh <IP_MASTER>

Creamos la maquina como master 

$ docker swarm init --advertise-addr <IP-GESTION>  ->Guardamos el TOKEN
$ docker exit

Creamos dos maquinas más:


$ docker-machine create --driver virtualbox dev01
$ docker-machine create --driver virtualbox dev02

Accedemos a la segunda maquian
$ docker-machine ssh <ip-dev01>
$ docker swarm join --token <ID-TOKEN> <IP_MASTER>:2377

# Si no nos acordamos del comando, ejecutamos en el master:
$ docker swarm join-token worker

Repetimos con la seguna maquina

Accedemos a la segunda maquian
$ docker-machine ssh <ip-dev02>
$ docker swarm join --token <ID-TOKEN> <IP_MASTER>:2377

Al final tenemos tres maquinas siendo la primera master.
A parti de aqui gestionamos todo desde la maquinas MASTER.
