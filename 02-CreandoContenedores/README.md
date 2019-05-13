---
title: Usando Docker 
description: Aprendemos a  Crear contenedores usando docker.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Docker Building

Veremos como personalizar nuestras propias imagenes de los contenedores, aprendermos a:
- Sobre el fichero Dockerfile
- Como funciona Docker Build
- Construyendo una imagen base usando el Dockerfile
- Variables de entorno
- Juntaremos todo

 [Esquema de ejemplo VM vs Docker](index-01.png)

## Introducció a fichero Docker File 

Como es un fichero de  Dockerfile en profundidad.  Un Dockerfile es simplemente un archivo de texto plano que contiene un conjunto de comandos definidos por el usuario, que cuando se ejecutan mediante el comando *docker image build*, que lo que realiza es una "Compilacion" de esa imagen de docker.

Un fichero Dockerfile de ejemplo:


```
FROM alpine:latest
LABEL maintainer="Mario Ezquerro <mario.ezquerro@gmail.com>"
LABEL description="This example Dockerfile installs NGINX."
RUN apk add --update nginx && \
    rm -rf /var/cache/apk/* && \
    mkdir -p /tmp/nginx/

COPY files/nginx.conf /etc/nginx/nginx.conf
COPY files/default.conf /etc/nginx/conf.d/default.confBuilding 
ADD files/html.tar.gz /usr/share/nginx/

EXPOSE 80/tcp
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]

```

Alpine Linux es una distribución de Linux no comercial pequeña, desarrollada independientemente, diseñada para la seguridad, la eficiencia y la facilidad de uso. Para obtener más información sobre Alpine Linux, visite el sitio web del proyecto en su [web](https://www.alpinelinux.org/)


## Revisando el Dockerfile en profundidad

Comando por orden de uso en un fichero Dockerfile: 

    - FROM
    - LABEL
    - RUN
    - COPY and ADD
    - EXPOSE
    - ENTRYPOINT and CMD
    - Otros comandos Dockerfile

## FROM
 Selecciona la imagen base que se usa, se indica imagen:tag, en nuestro caso es __alpine:latest__
```
   FROM <imagen>
   FROM <imagen>:<tag>
```

## LABEL
 Permite añadir información extra sobre el container las etiquetas que se puede usar estan docuemtadas [aqui](http://label-schema.org/)
 Las etiques se pueden consultar usando __docker inspect__.
 ```
    $ docker image inspect <IMAGEN_ID>
 ```
 O tambien:
 ```
    $ docker image inspect -f {{.Config.Labels}} <IMAGE_ID>
 ```

## RUN
Con el comando RUN permite instalar sofware y correr scripts con el ejemplo siguiente:
```
RUN   apk add --update nginx && \
      rm -rf /var/cache/apk/* && \
      mkdir -p /tmp/nginx/ 

RUN apk add --update nginx
RUN rm -rf /var/cache/apk/*
RUN mkdir -p /tmp/nginx/
```
El resutado de los dos ejemplos no es identico  al  agregar varias etiquetas, esto crearía una capa individual para cada uno de los Comandos RUN, que en su mayor parte deberíamos intentar y evitar.

Tiene dos modos modo shell: /bin/sh -c
```
 RUN comando
```
Y el modo ejecución:
```
   RUN ["ejecutable", "parámetro1", "parámetro2"]
   RUN ["/bin/bash", "-c", "echo prueba"]
```


## COPY and ADD
Este comando copia ficheros a nuestra imgen por ejemplo:
```
   COPY files/nginx.conf /etc/nginx/nginx.conf
```
Estos comandos sobrescriben cualquier archivo que existe tendriamos dentro del docker. 
ADD tiene mas capacidad,  ademas de agregar fichros carga y descomprime ficheros .tar y coloca las carpetas y archvos en la rutoa y puede usar rutas remotas como
```
   ADD http://www.myremotesource.com/files/html.tar.gz /usr/share/nginx/
```

## EXPOSE

El comando **EXPOSE** le permite a Docker saber que cuando se ejecuta la imagen, el puerto y El protocolo definido será expuesto en runtime. Este comando no asigna el puerto al máquina host, pero en su lugar abre el puerto para permitir el acceso al servicio en el contenedor red. Por ejemplo, en nuestro Dockerfile, le estamos diciendo a Docker que abra el puerto 80 cada vez la imagen corre:
```
   EXPOSE 80 / tcp
```
## ENTRYPOINT and CMD

Usaremos Entrypoint seguido  con comandos extra para que nuestro contenedor sea ejecutable.
```
   ENTRYPOINT ["nginx"]
   CMD ["-g", "daemon off;"]
```
Esto equivale a : $ nginx -g daemon off

Al hacer un _"docker run"_ sobre se ejecutara este ultimocomando

Esta instrucción también nos permite indicar el comando que se va ejecutar al iniciar el contenedor, pero en este caso el usuario no puede indicar otro comando al iniciar el contenedor.

Si usamos esta instrucción no permitimos o no  esperamos que el usuario ejecute otro comando que el especificado. Se puede usar junto a una instrucción CMD, donde se indicará los parámetro por defecto que tendrá el comando indicado en el ENTRYPOINT. Cualquier argumento que pasemos en la línea de comandos mediante docker run **serán anexados después** de todos los elementos especificados mediante la instrucción ENTRYPOINT, y anulará cualquier **elemento especificado con CMD**.
```
   ENTRYPOINT ["http", "-v"]
   CMD ["-p", "80"]
```

- $ docker run centos:centos7: Se creará el contenedor con el servidor web escuchando en el puerto 80.
- $ docker run centos:centros7 -p 8080: Se creará el contenedor con el servidor web escuchando en el puerto 8080.

La combinación de ENTRYPOINT y CMD le permite especificar el ejecutable predeterminado para su imagen, al mismo tiempo que proporciona argumentos predeterminados a ese ejecutable que el usuario puede invalidar. Veamos un ejemplo:
```
   FROM ubuntu:trusty
   ENTRYPOINT ["/bin/ping","-c","3"]
   CMD ["localhost"]
```
Al ejecutar el comando  sin argumentos:
```
   $ docker build -t ping .
   .......
   $ docker run ping
```
Y con argumentos:
```
   $ docker run ping docker.io
```



## Otros comandos Dockerfile
+ USER

   Permite definir el nombre del usuario que se utliza ne comando como en el RUN, CMD o ENTRYPOINT, el usuario tiene que estar en el sistema con permisos adecuados

+ WORKDIR

   Define el direcctorio de trabajo de los comando

+ ONBUILD

   La instrucción ONBUILD le permite esconder un conjunto de comandos que se usarán cuando la imagen se use nuevamente como una imagen base para un contenedor. Por ejemplo, si desea pasr una imagen a los desarrolladores y todos ellos tienen un código diferente que desean probar, puede usar la instrucción ONBUILD para asegurarse de que se carga el código real. Luego, los desarrolladores simplemente agregarán su código al directorio que  les diga y, cuando ejecuten un nuevo agregarán su código a la imagen en ejecución. La instrucción ONBUILD se puede utilizar junto con las instrucciones ADD y RUN ejemplo:
      
```
      ONBUILD RUN apk update && apk upgrade && rm -rf /var/cache/apk/*
```

   Esto ejecutaría una actualización y una actualización del paquete cada vez que nuestra imagen se use como base para otra imagen.

+ ENV

   El comando ENV establece las variables de entorno dentro de la imagen cuando se construye y cuando se ejecuta. Estas variables se pueden anular cuando inicie su imagen.
```
COPY package.json $PROJECT_DIR 
RUN npm install 
COPY . $PROJECT_DIR 
ENV MEDIA_DIR=/media \ 
		 NODE_ENV=production \ 
		 APP_PORT=3000 

VOLUME $MEDIA_DIR 
EXPOSE $APP_PORT 
HEALTHCHECK CMD curl --fail http://localhost:$APP_PORT || exit 
```

## Añadir HEALTHCHECK
Podemos iniciar el contenedor docker con la opción --restart always (reiniciar siempre). Después de un fallo del contenedor, el "demonio" de Docker intentará reiniciarlo. Es muy útil si tu contenedor tiene que estar operativo todo el tiempo. Pero, ¿qué pasa si el contenedor se está ejecutando, pero no está disponible (bucle infinito, configuración no válida, etc.)? Con la instrucción HEALTHCHECK podemos decirle a Docker que compruebe periódicamente el estado de salud de nuestro contenedor. Puede ser cualquier comando, devolviendo 0 como código de salida si todo está bien, y 1 en el caso contrario.

Nuestro ejemplo:
```
FROM node:7-alpine 
LABEL maintainer "jakub.skalecki@example.com" 

ENV PROJECT_DIR=/app 
WORKDIR $PROJECT_DIR 

COPY package.json $PROJECT_DIR 
RUN npm install 
COPY . $PROJECT_DIR 
ENV MEDIA_DIR=/media \ 
		 NODE_ENV=production \ 
		 APP_PORT=3000 

VOLUME $MEDIA_DIR 
EXPOSE $APP_PORT 
HEALTHCHECK CMD curl --fail http://localhost:$APP_PORT || exit 

ENTRYPOINT ["./entrypoint.sh"] 
CMD ["start"]
```
**curl --fail** devuelve el código de salida que no es cero si la petición falla.



## Dockerfiles – best practices

Las BestPractices para escribir nuestros propios Dockerfiles:

Usar un archivo .dockerignore Básicamente ignorará los elementos que haya especificado en el archivo durante el proceso de construcción.

Recuerde tener solo un archivo Docker por carpeta para ayudarlo a organizar sus contenedores.

Usa el control de versiones para tu Dockerfile; Al igual que cualquier otro documento basado en texto, el control de versiones lo ayudará a avanzar, pero solo hacia atrás, según sea necesario.

Minimiza la cantidad de paquetes que necesitas por imagen. Uno de los objetivos más importantes que desea lograr al construir sus imágenes es mantenerlas lo más pequeñas posible. No instalar paquetes innecesarios será de gran ayuda para lograr esto

Ejecutar solo un proceso de solicitud por contenedor. Cada vez que necesite una nueva aplicación, se recomienda utilizar un nuevo contenedor para ejecutar esa aplicación. Si bien puede juntar los comandos en un solo contenedor, es mejor separarlos. Mantén las cosas simples; Complicar más su archivo Docker agregará una gran cantidad de información y también podría causarle problemas más adelante.

Con el ejemplo, Docker tiene una guía de estilo bastante detallada para publicar las imágenes oficiales que albergan en Docker Hub, [documentada](https://github.com/docker-library/official-images/)


##Comando [Build](https://docs.docker.com/engine/reference/builder/)

El comando a usar es:

```
$ docker image build --help
```

+ ## Contruir un Dockerfile
      Usar --tag o -t para la construccion y  no es necesario usar --file o -f si se esta en la misma carpeta que el fichero DockerFile, solo se necesita añadir el'.' al final
      Se puede usar el fichero .dockerignore se usa para excluir aquellos archivos o carpetas que no queremos incluir en la compilación. Por defecto, todos los archivos de la carpeta Dockerfile se subirán. También discutimos colocar el Dockerfile en una carpeta separada, y lo mismo se aplica para .dockerignore. Debe ir en la carpeta donde se colocó el Dockerfile.
      
      Mantener todos los elementos que desea usar en una imagen en la misma carpeta lo ayudará a mantener la cantidad de elementos, si usamos un comando COPY este copiara todo, si no queremos que se incluyan ficheros estos deben estar en este fichero como ejemplo:
         
   ```
         .git
         .ipynb_checkpoints/*
         /notebooks/*
         /unused/*
         Dockerfile
         .DS_Store
         .gitignore
         README.md
         env.*
         /devops/*

         # To prevent storing dev/temporary container data
         *.csv
         /tmp/*

   ```

+ ## Imagenes a medida con Dockerfile 
      Si tenemos un Docker file como:
   ```
         FROM alpine:latest
         LABEL maintainer="Russ McKendrick <russ@mckendrick.io>"
         LABEL description="This example Dockerfile installs NGINX."

         RUN apk add --update nginx && \
            rm -rf /var/cache/apk/* && \
            mkdir -p /tmp/nginx/

         COPY files/nginx.conf /etc/nginx/nginx.conf
         COPY files/default.conf /etc/nginx/conf.d/default.conf
         ADD files/html.tar.gz /usr/share/nginx/

         EXPOSE 80/tcp

         ENTRYPOINT ["nginx"]
         CMD ["-g", "daemon off;"]
   ```
      Para nuestro __docker build__ podemos hacer la construccion
         $ docker image build --file <path_to_Dockerfile> --tag <REPOSITORY>:<TAG> .

+ ## Actuliazando nuestra imagen

      Descargamos la imagen que queremos actualizar:

         $ docker image pull alpine:latest

      Arrncamos la imagen y operamos con ella:

         $ docker container run -it --name alpine-test alpine /bin/sh

      Instalamos un NGINX:
   ```   
         $ apk update
         $ apk upgrade
         $ apk add --update nginx
         $ rm -rf /var/cache/apk/*
         $ mkdir -p /tmp/nginx/
         $ exit
   ```
      Tenemos que hacer nuestro commit:

         $ docker container commit <container_name> <REPOSITORY>:<TAG>

      Podemos guarda la imagen en un fichero TAR:
         $ docker image save -o <name_of_file.tar> <REPOSITORY>:<TAG>

+ ## Imagenes a desde cero

      Para crear una imagen desde cero tenemos un recurso 'scratch'
        
         $ docker image build --tag local:fromscratch .
      
      El fichero Dockerfile puede tener:
   ```
         FROM scratch
         ADD files/alpine-minirootfs-3.6.1-x86_64.tar /
         CMD ["/bin/sh"]
   ```
      y para iniciarlo:
      
         $ docker container run -it --name alpine-test local:fromscratch /bin/sh


+ ## Variables de entorno
      Con la instrucción ENV usaremos las variables de entorno recomendado usar un signo igual entre las expresiones, la ventaja es que ademas estas valores pueden consultarse desde dentro del contenedor
         
         ENV <key> <valor>
         ENV host sql
      
      Ademas usando es signo '=' podemos usar varios por  linea:

         ENV nombre-usuario=admin password=1234

      Estas variables pueden consultarse con la orden:

         $ docker image inspect <IMAGE_ID>

      Estos valores persistirán al momento de lanzar un contenedor de la imagen creada y pueden ser usados dentro de cualquier fichero del entorno, por ejemplo un script ejecutable. Pueden ser sustituida pasando la opción -env en docker run. Ejemplo:

         $ docker run -env <key>=<valor>



+ ## Todo en uno:
      Ejemplo de uso:
      ```
         FROM alpine:latest
         LABEL maintainer="Russ McKendrick <russ@mckendrick.io>"
         LABEL description="This example Dockerfile installs Apache & PHP."
         
         ENV PHPVERSION 7

         RUN apk add --update apache2 php${PHPVERSION}-apache2 php${PHPVERSION} && \
               rm -rf /var/cache/apk/* && \
               mkdir /run/apache2/ && \
               rm -rf /var/www/localhost/htdocs/index.html && \
               echo "<?php phpinfo(); ?>" > /var/www/localhost/htdocs/index.php && \
               chmod 755 /var/www/localhost/htdocs/index.php
         
         EXPOSE 80/tcp
         ENTRYPOINT ["httpd"]
         CMD ["-D", "FOREGROUND"]
      ```
      Para construir esta imagen
         
         $ docker build --tag local/apache-php:7 .

      ```
      Para inicializar el containter:
         
         $ docker container run -d -p 8080:80 --name apache-php7 local/apache-php:7

      PAra cambiar la version de PHP de 7 al 5:

         $ docker image build --tag local/apache-php:5 .

      PAra ejecutar este container:
       
         $ docker container run -d -p 9090:80 --name apache-php5 local/apache-php:5

      Por ultimo listamos las imagenes

         $ docker image ls

# Algunas notas más
Siguiendo el ejemplo, para detener el contenedor se puede ejecutar cualquiera de los siguientes comandos:
```
	$ docker kill a842945e2414 (envía SIGKILL)
	$ docker stop a842945e2414 (envía SIGTERM).
```
Si quieres saber más, echa un vistazo a las instrucciones de [STOPSIGNAL](https://docs.docker.com/engine/reference/builder/#stopsignal), [ONBUILD](https://docs.docker.com/engine/reference/builder/#onbuild) y [SHELL](). Además, algunas opciones muy útiles durante la compilación son --no-cache (especialmente en un servidor de integración continua, si quiere estar seguro de que la compilación se puede hacer en una nueva instalación de Docker), y --squash (más aquí).

