---
title: Usando Conexión
description: Aprendemos a usar docker.
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-05-10
---

### Configuración de TLS (cont.) [cc](https://raw.githubusercontent.com/macknilan/Cuaderno/master/Docker/Docker.md)

- Docker provee mecanismos para autenticar el cliente y el demonio entre ellos.
- Agrega autenticación y autorización y encriptación para las conexiones a través de la red
- Las claves pueden ser distribuidas a clientes autorizados
- Requisitos
    - Tener instalado OpenSSL 1.0.1 o superior
    - Crear una carpeta donde guardar las claves protegidas (chmod 700)


### Pasos a seguir
- Crear una **A**utoridad de **C**ertificación **(CA)**
    - **(b)**. `Es necesario una clave privada y un certificado.`
- Configurar la clave privada del servidor
- Crear un requerimiento de firma **(CSR)** de certificado para el servidor
- Firmar la clave del servidor con el **CSR** _mediante nuestro_ **CA**
- Crear una _clave privada_ y un **CSR** para el _cliente_
- _Firmar la clave del cliente_ con el **CSR** mediante nuestro **CA**
- _Iniciar el demonio_ de docker con la opción de **TLS** y _especificar_ la _ubicación_ de la _clave privada_ del **CA**, el certificado del servidor y la clave privada del servidor
- _Configurar_ el _cliente de docker_ para que utilice la _clave privada del cliente_ y la clave privada del **CA**

#### 1 Crear la clave privada y un certificado.

Se crea una carpeta donde se guarde la llave de la _Autoridad Certificante_ (**CA**)
```
$ mkdir docker-ca
```

(**1ra OPCION**) Se crea la llave **.pem** con el comando **"genrsa"** (`ESTA OPCIÓN ESTA DREPECADA`)
```
$ openssl genrsa -aes256 -out ca-key.pem 2048
```
+ **"-genrsa"** -> Comando para decirle a **openssl** que se creara una llave privada **rsa**
+ **"-aes256"** -> Es el algoritmo que se ocupa para crear la llave **.pem**
+ **"-out"** -> Se le dice cual es el archivo en el cual esta la llave (`ca-key.pem`)
+ **"2048"** -> La cantidad de bits que vamos a ocupar para nuestra encriptación.
_Posteriormente pide una frase para la llave._

(**2da OPCION**) Se crea la llave **.pem** con el comando **"genpkey"**
```
$ openssl genpkey -algorithm RSA -out ca-key.pem -pkeyopt rsa_keygen_bits:4096
```
+ **"-algorithm RSA"** -> Se le indica que deseamos que ocupe el algoritmo **RSA**.
+ **"-out"** -> Se le indica cual es el nombre del archivo `.pem (ca-key.pem)`
+ **"-pkeygen"** -> Se setea el algoritmo a ocupar.
+ **"rsa_keygen_bits:4096"** -> Es el algoritmo que se ocupara para crear la llave y ocupara 4096 bits para crearlo.

(**3da OPCION**) Se crea la llave **.pem** con el comando **"genpkey"** `ENCRIPTADA (RECOMENDADA)` - _Clave privada de la autoridad autorizante_
```
$ openssl genpkey -aes-256-cbc -algorithm RSA -out ca-key.pem -pkeyopt rsa_keygen_bits:4096
```
+ **"-aes-256-cbc"** -> Algoritmo de cifrado.
+ **"-algorithm RSA"** ->Se le indica que deseamos que ocupe el algoritmo **RSA**.
+ **"-out"** -> Se le indica cual es el nombre del archivo `.pem  (ca-key.pem)`
+ **"-pkeygen"** -> Se setea el algoritmo a ocupar.
+ **"rsa_keygen_bits:4096"** -> Es el algoritmo que se ocupara para crear la llave y ocupara 4096 bits para crearlo.

**1.1** - Crear un certificado que ocupara el servidor y el cliente. Se crea el archivo **C**ertificación de **A**utoridad (**CA**) Es la solicitud de firma del certificado.
```
$ openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```
+ **"req -new"** -> Solicita generar un nuevo certificado. **request new**
+ **"x509"** -> Es el tipo de certificado, que también sirve para poder gestionar el ciclo de vida del certificado.
+ **"-days 365"** -> Es el tiempo de vida que se le asigna al certificado.
+ **"[NOMBRE_/DE_LA/LLAVE].pem"** -> Es el nombre de la llave que se genero en el paso anterior **CA**, la _clave privada de la autoridad autorizante_.
+ **"-sha256"** -> Comando de encriptación para el certificado, que también puede ser "**-sha512**"
+ **"-out [NOMBRE_/DE_LA/LLAVE].pem"** -> Es en donde queremos guardar nuestro certificado. `(ca.pem)`

**NOTA**: AL CREAR EL CERTIFICADO PREGUNTARA POR LA **"frase"** DE LA LLAVE QUE SE ASIGNO AL CERTIFICADO **CA**


#### 2 Configurar la llave privada del servidor.
**NOTA**: Se crea la llave privada del servidor, _SIN LA FRASE DE SEGURIDAD_

Se crea la llave **.pem** con el comando **"genpkey"**
```
$ openssl genpkey -algorithm RSA -out server-key.pem -pkeyopt rsa_keygen_bits:4096
```
+ **"-algorithm RSA"** -> Se le indica que deseamos que ocupe el algoritmo RSA.
+ **"-out [NOMBRE_/DE_LA_/LLAVE].pem"** -> Se le indica cual es el nombre del archivo .pem (server-key.pem)
+ **"-pkeygen"** -> Se setea el algoritmo a ocupar.
+ **"rsa_keygen_bits:4096"** -> Es el algoritmo que se ocupara para crear la llave y ocupara 4096 bits para crearlo.

#### 3 - Crear un requerimiento de firma (Certificate Signing Request - CSR) de certificado para el servidor
```
$ openssl req -subj "/CN=localhost" -sha256 -new -key server-key.pem -out server.csr
```
+ **"openssl req"** -> Se solicita que **openssl** requiera la forma del certificado.
+ **"-subj "/CN=localhost""** -> Ahora que se tiene una **CA**, podemos crear la clave para el servidor y
la petición de firmado del certificado (**certificate signing request, CSR**).
Por favor, _verifica que Common Name (es decir, el FQDN o YOUR Name) coincide con el nombre del host que vas a usar para conectar a Docker_. 
Es la máquina en donde se ejecuta el docker al cual nos vamos a conectar remotamente.
+ **"-sha256"** -> Comando de encriptación para el certificado, que también puede ser "-sha512"
+ **"-new -key"** -> Se solicita a openssl una nueva clave sobre la clave del servidor, previamente realizada.
+ **"-out [NOMBRE_DEL_ARCHIVO]-csr"** -> Se crea el archivo .csr el cual es CSR de certificado para el servidor.

#### 4 - Firmar la clave del servidor con el CSR(Certificate Signing Request) mediante nuestro CA
`ESTE PASO ES EN DOS PARTES`

+ **PASO UNO** -> Crear un archivo de para configuración del servidor y decirle de que maquinas puede recibir requerimientos. _En caso de que sea una VPS se pone la IP publica_
    - Como las conexiones **TLS** pueden realizarse usando la dirección **IP** o un nombre **DNS**, _deben especificarse durante la creación del certificado._
```
$ echo subjectAltName = IP:192.168.1.20,IP:127.0.0.1 > extfile.cnf
```
+ **PASO DOS** -> Firmar la clave del servidor con la autoridad autorizante - **Firmar la clave pública con nuestra CA**

:link: [Link Protect the Docker daemon socket](https://docs.docker.com/engine/security/https/)
__________________________________
`Since TLS connections can be made via IP address as well as DNS name, the IP addresses need to be specified when creating the certificate. 
For example, to allow connections using 10.10.10.20 and 127.0.0.1:`
```
$ echo subjectAltName = DNS:$HOST,IP:10.10.10.20,IP:127.0.0.1 >> extfile.cnf
```
FRAGMENTO DE PÁGINA
__________________________________
```
$ openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```
+ **"openssl x509 -req -days 365"** -> Es el tipo de certificado, que también sirve para poder gestionar el ciclo de vida del certificado.Es el tiempo de vida que se le asigna al certificado.
+ **"-sha256"** -> Comando de encriptación para el certificado, que también puede ser **"-sha512"**
+ **"-in server.csr"** -> Se le dice a openssl que ocupe el archivo **CSR** que se creo en el paso anterior.
+ **"-CA ca.pem"** -> Se solicita a openssl el certificado de autoridad creado en el paso anterior
+ **"-CAkey ca-key.pem"** -> Se lolicita la clave privada de la autoridad autorizante
+ **"-CAcreateserial -out server-cert.pem"** -> Es el parámetro que dice que estamos haciendo las credenciales para el servidor con salida al fichero **server-cert.pem** _para el servidor_.
+ **"-extfile extfile.cnf"** -> Le mandamos los parámetros al archivo creado en el paso anterior

#### 5 - Crear una clave privada y un CSR para el `cliente`
**SON DOS PASOS**. Se realizan los mismos pasos que se hicieron que del lado del servidor, pero en la creación de la clave no se hará con la frase de seguridad, y se ocupara la **"1ra OPCIÓN"**

##### PASO UNO CREAR UNA CLAVE PRIVADA

(**1ra OPCION**) Se crea la llave **.pem** con el comando **"genpkey"** _---SIN ENCRIPTAR---_ **Clave para el cliente**
```
$ openssl genpkey -algorithm RSA -out client-key.pem -pkeyopt rsa_keygen_bits:4096
```
+ **"-algorithm RSA"** -> Se le indica que deseamos que ocupe el algoritmo RSA.
+ **"-out"** -> Se le indica cual es el nombre del archivo .pem (client.key.pem)
+ **"-pkeygen"** -> Se setea el algoritmo a ocupar.
+ **"rsa_keygen_bits:4096"** -> Es el algoritmo que se ocupara para crear la llave y ocupara 4096 bits para crearlo.

(**2da OPCION**) Se crea la llave **.pem** con el comando **"genpkey"** _---ENCRIPTADA---_  **Clave privada para el cliente.**
```
$ openssl genpkey -aes-256-cbc -algorithm RSA -out client-key.pem -pkeyopt rsa_keygen_bits:4096
```
+ **"-aes-256-cbc"** -> Algoritmo de cifrado.
+ **"-algorithm RSA"** ->Se le indica que deseamos que ocupe el algoritmo **RSA**.
+ **"-out"** -> Se le indica cual es el nombre del archivo **.pem  (client-key.pem)**
+ **"-pkeygen"** -> Se setea el algoritmo a ocupar.
+ **"rsa_keygen_bits:4096"** -> Es el algoritmo que se ocupara para crear la llave y ocupara **4096 bits** para crearlo.

##### PASO DOS Crear un requerimiento de firma (Certificate Signing Request - CSR) de certificado para el cliente
```
$ openssl req -subj "/CN=client" -sha256 -new -key server-key.pem -out client.csr
```
+ **"openssl req"** -> Se solicita que openssl requiera la forma del certificado.
+ **"-subj "/CN=client""** -> Se le puede escribir un nombre o un email
+ **"-sha256"** -> Comando de encriptación para el certificado, que también puede ser **"-sha512"**
+ **"-new -key"** -> Se solicita a openssl una nueva clave sobre la clave del servidor, previamente realizada.
+ **"-out [NOMBRE_DEL_ARCHIVO]-csr"** -> Se crea el archivo **.csr** el cual es **CSR de certificado para el servidor**.

#### 6 - Firmar la clave del cliente con el CSR(Certificate Signing Request) mediante nuestro CA
**ESTE PASO ES EN DOS PARTES**
Para simplificar, los siguientes dos pasos pueden realizarse desde la máquina donde se encuentra el Docker daemon.
_Lo mismo que se aplica para el servidor en el paso 4, aplica pero del lado del cliente._

**PASO UNO**
```
$ echo "extendedKeyUsage = clientAuth" > extfile.cnf
```

**""extendedKeyUsage = clientAuth""** -> Se le dice a la autoridad autorizante **(CA)**; esto _es para una cliente_ y no para un servidor

**PASO DOS**
```
$ openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem -extfile extfile.cnf
```
+ **"openssl x509 -req -days 365"** -> Es el tipo de certificado, que también sirve para poder gestionar el ciclo de vida del certificado.Es el tiempo de vida que se le asigna al certificado.
+ **"-sha256"** -> Comando de encriptación para el certificado, que también puede ser **"-sha512"**
+ **"-in client.csr"** -> Se le dice a openssl que **ocupe el archivo CSR** que _se creo en el paso anterior._
+ **"-CA ca.pem"** -> Se solicita a openssl el _certificado de autoridad creado_, el certificado del servidor.
+ **"-CAkey ca-key.pem"** -> Se solicita la clave privada de la autoridad autorizante
+ **"-CAcreateserial -out client-cert.pem"** -> Es el parámetro que dice que estamos haciendo las credenciales para el servidor con salida al fichero **client-cert.pem** para el servidor.
+ **"-extfile extfile.cnf"** -> Le mandamos los parámetros al archivo creado en el paso anterior.

### Asegurando nuestras claves
1. Asegurarse que las claves del cliente y del servidor sólo puedan ser leídas por el usuario actual
```
chmod -v 0400 ca-key.pem client-key.pem server-key.pem
```

2. Remover el acceso de escritura a todos los certificados
```
chmod -v 0444 ca.pem server-cert.pem client-cert.pem
```
3. Crear la carpeta `/etc/docker` en caso que no exista.

4. Cambiar los permisos de la carpeta `/etc/docker`.

```
sudo chown <username>:docker /etc/docker
sudo chmod 700 /etc/docker
```

5. Copiar las claves de servidor a la nueva carpeta
```
sudo cp ~/docker-ca/{ca,server-key,server-cert}.pem /etc/docker
```


### Usando Docker con TLS
1. Iniciar el demonio con los siguientes parámetros:
```
DOCKER_OPTS="-H tcp://0.0.0.0:2376 --tlsverify
--tlscacert=/etc/docker/ca.pem
--tlscert=/etc/docker/server-cert.pem
--tlskey=/etc/docker/server-key.pem”
```

2. Reiniciar el servicio de Docker en caso de ser necesario
```
sudo service docker restart
```

3. Utilizar las credenciales correspondientes en el cliente (`carpeta docker-ca`)
```
docker --tlsverify \
--tlscacert=ca.pem \
--tlscert=client-cert.pem \
--tlskey=client-key.pem \
-H tcp://127.0.0.1:2376 \
```

### Tips

- Podemos emitir especificar todas las claves en el cliente creando una carpeta **.docker** en el home de nuestro usuario
- Sin embargo, es necesario renombrar nuestros archivos a: **ca.pem, cert.pem and key.pem**
- Una vez realizados los pasos anteriores cada vez que utilicemos el comando docker, el cliente utilizará las claves automáticamente. Sólo es necesario especificar el comando `--tlsverify y -H`
```
docker --tlsverify -H 127.0.0.1:2376 ps -a
```
- Para simplificar aún más el uso del cliente, podemos utilizar las variables de entorno `DOCKER_HOST` y `DOCKER_TLS_VERIF`