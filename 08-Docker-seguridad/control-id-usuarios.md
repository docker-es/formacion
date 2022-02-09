## Control de Containers con  Namespaces
## Docker y --userns-remap

Tienes tambien la información (en:)[https://github.com/docker-es/formacion/blob/master/04-Gestion-Contenedoes/linux-user-namespaces.md]

Un namespace puede evitar que los contenedores se ejecuten como usuarios privilegiados, como el daemon de docker suele iniciarse con el usuario root, los conedores internamente se pesentan con este usuario, lo que permite una  escalada de privilegios, sobretodo en el "mapeo" de volumenes.

Otro problem que necesitamos solucionar es os archivos creados dentro de los contenedores tienden a tener una propiedad impredecible al inspeccionarlos desde el host. El propietario de los archivos en un volumen es root (uid 0) por defecto, pero tan pronto como las cuentas de usuario no root están creadas dentro en el contenedor y escriben en el sistema de archivos, y estas no existen en los host, los propietarios se vuelven más o menos aleatorios desde la perspectiva del host.

Es un problema cuando necesita acceder a los datos de volumen desde el host utilizando la misma cuenta de usuario que esta creado dentro del contenedor.

Las soluciones típicas son:

- Forzar los UID de los usuarios en el momento de la creación en Dockerfiles (plante problemas de crear imagenes universales)
- Pasar el UID del usuario del host al 'docker run' como una variable de entorno y luego ejecutar algunos 'chown' en los volúmenes en un script de punto de entrada.

Podemos habilitar control el usuario de los docker. haciendo uso de `/etc/subuid` y  `/etc/subgid` archivos como se muestra a continuación.

- crear un usuario usando el `adduser` comando

```markup
sudo adduser dockremap
```

Copy

- Configurar un subuido para el usuario `dockremap`

```markup
sudo sh -c 'echo dockremap:400000:65536 > /etc/subuid'
```

Copy

- Luego configure subgid para el usuario `dockremap`

```markup
sudo sh -c 'echo dockremap:400000:65536 > /etc/subgid'
```

Copy

- Abra la `daemon.json` archivo y rellénelo con el siguiente contenido para asociar el `userns-remap` atributo al usuario `dockremap`

```markup
vi   /etc/docker/daemon.json
```

Copy

```markup
{ 

 "userns-remap": "dockremap"

}
```

Copy

- Prensa `:wq` para guardar y cerrar `daemon.json` archivo y finalmente reinicie la ventana acoplable para habilitar los espacios de nombres en un host de la ventana acoplable

```markup
sudo  /etc/init.d/docker  restart
```

Esto permite que cuando accedemos internamete aun contenedore te presentas como "root" pero fuera de ese contenedor tienes un ID "alto".

## Siempre tenemos que tener en cuenta que las varibles de usuario son:

Para los contenedores se indica que deberán correr como el userid/groupid pasado como parámetro a través de las variables de entorno DOCKER_UID y DOCKER_GID respectivamente.

De esta forma, ambos contenedores pueden ser levantados a nombre de un usuario no privilegiado en una estación de trabajo. Tanto Apache como MySQL por ejemplo, correrán a nombre de nuestro usuario local, el mismo usuario con el que editaremos los fuentes desde un IDE. Esto simplifica mucho el trabajo para los desarrolladores.

Gracias a esta configuración no es necesario realizar ningún tipo de ajustes de permisos en la copia de trabajo del repositorio de los archivos del sitio web a servir.



## Si no usamos los Namespaces

La solución es asignar dinámicamente el uid del usuario en el tiempo de compilación para que coincida con el host.

Ejemplo `Dockerfile`:

```
FROM ubuntu
# Defines argument which can be passed during build time.
ARG UID=1000
# Create a user with given UID.
RUN useradd -d /home/ubuntu -ms /bin/bash -g root -G sudo -u $UID ubuntu
# Switch to ubuntu user by default.
USER ubuntu
# Check the current uid of the user.
RUN id
# ...
```

Luego construye como:

```
docker build --build-arg UID=$UID -t mycontainer .
```

y ejecutar como:

```
docker run mycontainer
```

------

Si tiene un contenedor existente, cree un contenedor de envoltura con lo siguiente `Dockerfile`:

```
FROM someexistingcontainer
ARG UID=1000
USER root
# This assumes you've the existing user ubuntu.
RUN usermod -u $UID ubuntu
USER ubuntu
```

Esto se puede envolver `docker-compose.yml`como:

```
version: '3.4'
services:
  myservice:
    command: id
    image: myservice
    build:
      context: .
    volumes:
    - /data:/data:rw
```

Luego compile y ejecute como:

```
docker-compose build --build-arg UID=$UID myservice; docker-compose run myservice
```
