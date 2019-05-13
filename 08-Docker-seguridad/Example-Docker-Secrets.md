# Ejemplo de Docker Secrets

## Sin Secretos

Antes de comenzar a usar secretos, veamos las desventajas de no usarlo. A continuación se muestra un archivo de composición con la definición de un servicio de Postgres y administrador (un cliente de base de datos):

```
version: '3.1'
services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mysupersecretpassword
      POSTGRES_DB: mydatabase
  adminer:
    image: adminer
    ports:
     - 8080:8080
```

Hemos proporcionado el nombre de usuario, la contraseña y el nombre de la base de datos para el servicio de Postgres al configurar las variables de entorno POSTGRES_USER, POSTGRES_PASSWORD y POSTGRES_DB (consulte la [documentación](https://hub.docker.com/_/postgres/) de la imagen de Postgres Docker para ver otras variables de entorno que se pueden usar).

Es posible que haya notado que los detalles confidenciales de nuestra base de datos están en texto simple para que todo el mundo los vea. Esta es una mala práctica y solo debe hacerse para implementaciones locales de contenedores. En la siguiente sección, modificaremos este ejemplo para usar secretos.

## Segurizando Our Swarm usando Secrets

Vamos a sumergirnos y ver cómo crear secretos.

Abra una ventana de terminal y cree un secreto para el nombre de usuario escribiendo el siguiente comando:

```
    echo "myuser" | docker secret create pg_user -
```

Hemos utilizado el comando **docker secret** create anterior para crear un secreto llamado **pg_user**. El guión "-" al final del comando es importante, le permite a docker saber que los datos del secreto se están tomando de la entrada estándar.

Para ver el secreto, escriba el siguiente comando:

```
    docker secret ls
```
Vamos a crear los secretos restantes para la contraseña y el nombre de la base de datos:
```
    echo "mysupersecretpassword" | docker secret create pg_password -
    echo "mydatabase" | docker secret create pg_database -
```
Ahora necesitamos modificar el archivo de composición para usar los secretos que creamos, vea a continuación:
```
version: '3.1'
services:
    db:
        image: postgres
        restart: always
        environment:
            POSTGRES_USER_FILE: /run/secrets/pg_user
            POSTGRES_PASSWORD_FILE: /run/secrets/pg_password
            POSTGRES_DB_FILE: /run/secrets/pg_database
        secrets:
           - pg_password
           - pg_user
           - pg_database
    adminer: 
        image: adminer 
        ports: 
         - 8080:8080
secrets:
  pg_user:
    external: true
  pg_password:
    external: true
  pg_database:
    external: true
```

Cree un archivo llamado **postgres-secrets.yml** y copie el texto de arriba. Este archivo se utilizará más adelante en esta publicación para implementar nuestros servicios.

Hemos añadido algunas cosas para que nuestros secretos funcionen. En primer lugar, las variables de entorno que utilizamos anteriormente tienen el sufijo "_FILE". La ruta al secreto también se especifica, ver más abajo:
```
POSTGRES_USER_FILE: /run/secrets/pg_user 
POSTGRES_PASSWORD_FILE: /run/secrets/pg_password 
POSTGRES_DB_FILE: /run/secrets/pg_database
```
Los secretos de Docker se almacenan en archivos en la carpeta / run / secrets del contenedor. Es por eso que tenemos que especificar nuevas variables de entorno para leer los secretos almacenados en estos archivos.

Nota importante: no todas las imágenes tienen variables de entorno compatibles con los secretos. En muchos casos tendrás que modificar la definición de imágenes (Dockerfile) para leer secretos.

En segundo lugar, especificamos el nombre de los secretos que está utilizando el servicio:
```
secrets:
- pg_password
- pg_user
- pg_database
```
Y por último, pero no menos importante, indicamos que los secretos son externos:
```
secrets:
 pg_user:
   external: true
 pg_password: 
   external: true
 pg_database: 
   external: true
```
## Desplegando el servicio

Implementemos nuestro servicio para determinar si nuestros secretos funcionan como se espera. Escriba el siguiente comando:
```
    docker stack deploy -c postgres-secrets.yml postgres
```
Deles a los servicios un minuto para comenzar, y navegue a http://127.0.0.1:8080/ para ver el Interfaz de administrador.