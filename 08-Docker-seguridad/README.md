---
title: Usando Docker compose.
description: Aprendemos Docker-compose y a usar las regras
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

# Secrets

Acerca de los secretos
En términos de los servicios de Docker Swarm, un secreto es un blob de datos, como una contraseña, una clave privada SSH, un certificado SSL u otro dato que no debe transmitirse a través de una red o almacenarse sin cifrar en un Dockerfile o en su aplicación. código fuente.

En Docker 1.13 y superior, puede usar los secretos de Docker para administrar centralmente estos datos y transmitirlos de manera segura solo a aquellos contenedores que necesitan acceso a ellos. Los secretos se cifran durante el tránsito y en reposo en un Swarm Docker. Un secreto dado solo es accesible para aquellos servicios a los que se le ha otorgado acceso explícito, y solo mientras se ejecutan esas tareas de servicio.

Puede usar secretos para administrar cualquier información confidencial que necesite un contenedor en el tiempo de ejecución, pero no desea almacenar en la imagen o en el control de origen, como:

- Nombres de usuario y contraseñas
- Certificados y claves TLS
- Llaves SSH
- Otros datos importantes, como el nombre de una base de datos o un servidor interno
- Cadenas genéricas o contenido binario (hasta 500 kb de tamaño)

Nota: los secretos de Docker solo están disponibles para servicios de Swarm, no para contenedores independientes. Para usar esta función, considere adaptar su contenedor para que se ejecute como un servicio. Los contenedores con estado generalmente pueden ejecutarse con una escala de 1 sin cambiar el código del contenedor.

Otro caso de uso para usar secretos es proporcionar una capa de abstracción entre el contenedor y un conjunto de credenciales. Considere un escenario en el que tenga entornos de desarrollo, prueba y producción separados para su aplicación. Cada uno de estos entornos puede tener diferentes credenciales, almacenadas en los enjambres de desarrollo, prueba y producción con el mismo nombre secreto. Sus contenedores solo necesitan saber el nombre del secreto para funcionar en los tres entornos.

También puede usar secretos para administrar datos no confidenciales, como archivos de configuración. Sin embargo, Docker 17.06 y versiones posteriores admiten el uso de configuraciones para almacenar datos no confidenciales. Las configuraciones se montan en el sistema de archivos del contenedor directamente, sin el uso de un disco RAM.

## Ejemplos de uso [code](https://github.com/picodotdev/blog-ejemplos/tree/master/DockerSwarm)

Los secretos de Docker se proporcionan a los contenedores que los necesitan y se transmiten de forma cifrada al nodo en el que se ejecuten. Los secretos se montan en el sistema de archivos en la ruta /run/secrets/<secret_name> de forma descifrada al que el servicio del contenedor puede acceder.

Usando un stack de servicios con un archivo de Docker Compose en la sección secrets de los servicios se indica cuales usa, en la sección secrets se definen los secretos de los servicios con sus nombres y su contenido referenciando archivos que pueden ser binarios o de text no superior a 500 KiB.

Al servicio de nginx la clave privada y certificado para configurar el acceso mediante el protocolo seguro HTTPS se le proporciona a través de secretos que son referenciados en el archivo de configuración del servidor web nginx.conf.

file: docker-compose-stack-app.yml
```
version: "3.1"

services:
  nginx:
    image: nginx:stable-alpine
    ports:
      - "80:80"
      - "443:443"
    networks:
      - proxy
    secrets:
      - nginx.conf
      - localhost.pem
      - localhost.key
    command: sh -c "exec nginx -c /run/secrets/nginx.conf -g 'daemon off;'"
  app:
    image: localhost:5000/spring-boot-app:1.0
    ports:
      - "8080:8080"
    networks:
      - proxy
    volumes:
      - app:/data
    secrets:
      - message.txt
networks:
  proxy:
volumes:
  app:
    driver: rexray
    external: true
secrets:
  nginx.conf:
    file: ./nginx.conf
  localhost.pem:
    file: ./localhost.pem
  localhost.key:
    file: ./localhost.key
  message.txt:
    file: ./message.txt
```

File: nginx.conf
```
..

http {
    ...

    upstream app {
    	server app_app:8080;
    }

    server {
        listen       80;
        server_name  localhost;

        return 301 https://$host$request_uri;
    }

    server {
        listen       443;
        server_name  localhost;

        ssl                  on;
        ssl_certificate      /run/secrets/localhost.pem;
        ssl_certificate_key  /run/secrets/localhost.key;

        ssl_session_timeout  5m;

        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers   on;

        location / {
            proxy_pass http://app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

## Lea más sobre los comandos secretos de docker.

Use estos enlaces para leer sobre comandos específicos, o continúe [con el ejemplo sobre el uso de secretos con un servicio](https://docs.docker.com/engine/swarm/secrets/#build-support-for-docker-secrets-into-your-images).

- docker secret create
- docker secret inspect
- docker secret ls
- docker secret rm
- --secret flag for docker service create
- --secret-add and --secret-rm flags for docker service updateer


Agrega un secreto a Docker. El comando **docker secret create** lee la entrada estándar porque el último argumento, que representa el archivo desde el cual se lee el secreto, se establece en -.

$ printf "This is a secret" | docker secret create my_secret_data -

Crea un servicio redis y concédele acceso al secreto. De forma predeterminada, el contenedor puede acceder al secreto en **/run/secrets/<secret_name>**, pero puede personalizar el nombre del archivo en el contenedor utilizando la opción de destino.

$ docker service create --name redis --secret my_secret_data redis: alpine


## Secretos con [VAULT](https://www.bbva.com/es/gestion-secretos-contenedores-docker/)



# Docker y seguridad


Como cualquier otra tecnología, Docker no está exento de posibles problemas de seguridad. Para minimizarlos lo mejor es aplicar buenas prácticas y auditar nuestra infraestructura frecuentemente en busca de vulnerabilidades. Aquí van cuatro consejos para comenzar de una manera sencilla.


## Dockerbench
Docker mantiene una herramienta de auditoria de la infraestructura donde se ejecuta Docker Engine. Como no podía ser de otra manera, mediante la ejecución de un contenedor comprueba nuestra instalación y analiza si hemos aplicado las buenas prácticas recogidas en este documento de casi 200 páginas publicado por e[l Centro para la seguridad en internet (CIS)](https://benchmarks.cisecurity.org/). 
 
Simplemente para auditar nuestro motor debemos ejecutar: 
```
docker run -it --net host --pid host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```

Esto nos generará una salida en pantalla con el diagnostico de cada uno de los escenarios programado similar a esto:

Existe una página del proyecto alojada en github donde podemos [ampliar información](https://github.com/docker/docker-bench-security) sobre sus funcionalidades. 

## Docker security scanning

Otra manera de comprobar cómo de seguras son nuestras imágenes es mediante el escaneo en búsqueda de [vulnerabilidades conocidas](https://babel.es/es/blog/blog/marzo-2017/4-sencillas-practicas-sobre-seguridad-en-docker). 

Docker Inc mantiene un servicio encargado de esto que utiliza [la base de datos de CVE](https://cve.mitre.org/) para encontrar en nuestras imágenes librerías, binarios y exploits de los programas que se ejecutan en ellas. Este servicio está disponible para los repositorios privados de Docker hub, en[ Docker cloud](https://docs.docker.com/docker-cloud/builds/image-scan/) y recientemente [on-premise en la versión 1.13 de Docker DataCenter](https://www.youtube.com/watch?v=180RC4tlwIQ), siendo en todos los casos un servicio de pago aunque hay [servicios de terceros](https://anchore.com/enterprise/) que ofrecen una funcionalidad similar de manera gratuita.

## Content trust
Si queremos que en nuestro motor solo se ejecuten imágenes firmadas en las que confiemos podemos habilitar [Content trust](https://docs.docker.com/engine/security/trust/content_trust/). 

Para habilitar esta opción que por defecto viene desactivada simplemente tenemos que definir la variable de entorno DOCKER_CONTENT_TRUST o ejecutar Docker Engine con la opción --disable-content-trust=false
 
Como el mantener una infraestructura basado en confianza mediante claves públicas y privadas puede ser algo tedioso, tenemos la posibilidad de instalar un componente llamado [Docker Notary](https://docs.docker.com/notary/) que hará la labor de "certificadora" entre el repositorio de imágenes (Docker Registry) y nuestro nodo.

https://github.com/theupdateframework/notary

En Docker Datacenter esta funcionalidad está integrada con Docker Trusted Registry y podemos [habilitarla de forma nativa añadiendo](https://www.youtube.com/watch?v=ZZWlo2YIfpY) toda la capilaridad de la gestión de roles y usuarios corporativos. 

## Contenedores en solo lectura

Docker utiliza la tecnología **copy on write** para acelerar el inicio de los contenedores y optimizar el uso de disco en las imágenes, es decir, por naturaleza podríamos decir que son inmutables (o siguen los preceptos de la infraestructura inmutable). Para ser exactos, los contenedores montan una capa temporal en modo lectura/escritura que perdura solo mientras el contenedor está en ejecución. Al desaparecer el contenedor esta capa desaparece con él. 
 
Una buena práctica, si nuestra aplicación no necesita tener acceso al disco, es crear esta capa en modo solo lectura con el fin de poner aún más difícil la tarea de explotar vulnerabilidades a un potencial atacante. Simplemente pondremos en ejecución nuestros contenedores con la opción --read-only. 
 
El experto en seguridad Diogo Monica explica [en este artículo](https://diogomonica.com/2016/11/19/increasing-attacker-cost-using-immutable-infrastructure/) un caso de uso práctico donde esta funcionalidad solventa un error de programación en PHP.
  
Con todo esto, el tema de la seguridad es un mundo complejo en el que conviene estar bien informado y al día. Para esto podemos consultar [la página de seguridad de Docker](https://github.com/docker/labs/tree/master/security), realizar los laboratorios oficiales sobre seguridad o empaparnos de [los principios de seguridad de Docker Engine](https://docs.docker.com/engine/security/security/).


# Dagda

La solución propuesta para dar solución para la auditoría de las aplicaciones que corren en entornos Docker ha sido [Dagda](https://github.com/eliasgranderubio/dagda), que se puede definir como una suite de seguridad Docker que permite tanto el análisis estático de vulnerabilidades de los componentes software presentes en imágenes Docker como el análisis de dependencias de distintos frameworks de aplicaciones. Además, permite detectar comportamientos anómalos en contenedores Docker en ejecución en base a reglas.

En primer lugar, Dagda no se apoya en el proyecto cve-search para su base de conocimiento de vulnerabilidades, ya que cve-search necesita, además de MongoDB, un Redis para su funcionamiento.

VulnDB: Base de datos de vulnerabilidades en Dagda

La necesidad de otro componente más como es Redis, unido a la rigidez de su API de consulta, ha desembocado en lo que en la Figura 13 se denomina VulnDB, que no es más que una base de datos MongoDB. Esta base de datos está fuera del servidor Dagda, por lo que puede ser cualquier MongoDB proporcionado por el usuario.

En la base de datos VulnDB de Dagda, además de almacenar los expedientes CVE e información de Exploit-db, se ha añadido la base de datos completa de BugTraq de Symantec, la cual no ha sido utilizada nunca hasta la fecha por ninguna otra herramienta.

Además, en dicha base de datos se almacena también el histórico de los análisis estáticos realizados sobre las imágenes Docker y el resultado de la monitorización de los contenedores en ejecución. Esta funcionalidad de histórico también es única entre las herramientas y productos Open Source ya que éstos únicamente utilizan su base de datos para consultar las vulnerabilidades conocidas, quedando fuera de su objetivo, la gestión del histórico de los análisis realizados. 

## Análisis estático de vulnerabilidades en Dagda

En cuanto al análisis estático de vulnerabilidades de los componentes software presente en las imágenes docker, Dagda extrae el listado del software instalado en la imagen y busca las coincidencias con productos y versiones vulnerables contra VulnDB. Dagda soporta un gran número de distribuciones como: Red Hat/CentOS/Fedora, Debian/Ubuntu, OpenSUSE y Alpine, aunque es fácilmente ampliable.

Por otro lado, en cuanto al análisis de dependencias de distintos framworks de aplicaciones, Dagda se integra con el proyecto [Docker dependency checker](https://hub.docker.com/r/deepfenceio/deepfence_depcheck/), el cual utiliza [OWASP Dependency Check](https://www.owasp.org/index.php/OWASP_Dependency_Check) y [Retire.js](http://retirejs.github.io/retire.js/) para analizar múltiples lenguajes como son: Java, Python, Node.js, JavaScript, Ruby & PHP. 

Dagda soporta también la detección de comportamientos anómalos en contenedores Docker en ejecución en base a reglas gracias a su integración con [Sysdig/Falco](http://www.sysdig.org/falco/), siendo dichas reglas, ampliables también según las necesidades del usuario en cada entorno de trabajo o producción.


## Temas complementarios (15% of exam)

[El proceso de firma de una imagen.](https://docs.docker.com/engine/security/trust/content_trust/#push-trusted-content)
[Com usar una imagen pasa un escaneo de seguridad.](https://docs.docker.com/docker-cloud/builds/image-scan/)
[Habilitar Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/)
[Configurar RBAC en UCP](https://docs.docker.com/datacenter/ucp/2.2/guides/access-control/)
[Integrar UCP con LDAP/AD](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/external-auth/)
[Usando la creación de paquetes de clientes UCP](https://blog.docker.com/2017/09/get-familiar-docker-enterprise-edition-client-bundles/)
[La seguridad del motor por docker por defecto](https://docs.docker.com/engine/security/security/)
[La seguridad en swarm.](https://docs.docker.com/engine/swarm/how-swarm-mode-works/pki/)
[Que es MTLSS](https://diogomonica.com/2017/01/11/hitless-tls-certificate-rotation-in-go/)
