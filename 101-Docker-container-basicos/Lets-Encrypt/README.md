La solución que mejor se ajusta a estos requisitos es la de Pierre Prinetti, disponible en el siguiente [enlace](https://github.com/pierreprinetti/certbot)

Básicamente consiste en crear un volumen docker donde se guardarán los certificados y generarlos ejecutando su contenedor. Por lo tanto, si generamos certificados en un volumen compartido por el contenedor de los certificados y el de nginx, no necesito tocar ninguno de los otros contenedores, solamente configurar dónde están los certificados generados por let’s encrypt.

Pasos a seguir:

1.  Creamos un volumen con el nombre “certs”

    $ docker volume create --name certs

2. Paramos momentáneamente el contenedor del servidor http en el que tenemos desplegado nuestra aplicación web para poder generar los certificados.

    $ docker stop http-server

3. Ejecutamos el contenedor con el certificado o certificados que queremos generar
```
    docker run \
    -v certs:/etc/letsencrypt \
    -e http_proxy=$http_proxy \
    -e domains="my.domain.com" \
    -e email="firstName.lastName@domain.com" \
    -p 80:80 \
    -p 443:443 \
    --rm pierreprinetti/certbot:latest
```

4. Volvemos a levantar el servidor http.

    $ docker start http-server

Podemos comprobar que los certificados están bien generados inspeccionando el volumen “certs”

    $ docker volume inspect certs

Una vez generados, sólo tenemos que montar el volumen de los certificados en nuestro servidor http, en nuestro caso, orquestamos todos los contenedores con docker-compose y cambiar la configuración del servidor para que utilice los nuevos certificados.

En nuestro caso, con nginx,
```
    listen 443 http2;
    listen [::]:443 http2;
    server_name example.com;

    ssl on;
    ssl_certificate /etc/nginx/certs/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/live/example.com/privkey.pem;
```

Una vez cambiado, sólo queda relanzar docker-compose con los cambios que hemos hecho en el contenedor del servidor http.

docker-compose up -d