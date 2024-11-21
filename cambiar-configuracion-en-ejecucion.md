# Cómo cambiar la configuración de los contenedores Docker en ejecución – CloudSavvy IT

*2022-01-12* *[0 ](https://gruposaedal.com/como-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it/#comments)* *Por* [LJXIH](https://gruposaedal.com/author/ljxih/)

- [Facebook](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fgruposaedal.com%2Fcomo-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it%2F&t=Cómo+cambiar+la+configuración+de+los+contenedores+Docker+en+ejecución+–+CloudSavvy+IT)
- [Twitter](https://gruposaedal.com/como-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it/#)
- [Pinterest](https://gruposaedal.com/como-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it/#)
- [LinkedIn](https://www.linkedin.com/shareArticle?mini=true&ro=true&trk=EasySocialShareButtons&title=Cómo+cambiar+la+configuración+de+los+contenedores+Docker+en+ejecución+–+CloudSavvy+IT&url=https%3A%2F%2Fgruposaedal.com%2Fcomo-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it%2F)
- [WhatsApp](whatsapp://send?text=Cómo cambiar la configuración de los contenedores Docker en ejecución – CloudSavvy IT https%3A%2F%2Fgruposaedal.com%2Fcomo-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it%2F)
- [Reddit](http://reddit.com/submit?url=https%3A%2F%2Fgruposaedal.com%2Fcomo-cambiar-la-configuracion-de-los-contenedores-docker-en-ejecucion-cloudsavvy-it%2F&title=Cómo+cambiar+la+configuración+de+los+contenedores+Docker+en+ejecución+–+CloudSavvy+IT)



![img](https://www.cloudsavvyit.com/p/uploads/2021/09/993634a1.png?width=1198&trim=1,1&bg-color=000&pad=1,1)

Los contenedores Docker generalmente se tratan como inmutables una vez que comienzan a ejecutarse. Sin embargo, puede actualizar dinámicamente algunas opciones de configuración, como el nombre del contenedor y sus límites de recursos de hardware.



En esta guía, le mostraremos cómo usar los comandos integrados de Docker para cambiar la configuración seleccionada de los contenedores en ejecución. También veremos lo que no debe cambiar y una solución alternativa que puede usar si cree que debería hacerlo.

## Cambiar el nombre de un contenedor

La modificación más simple es cambiar el nombre de un contenedor creado. Los nombres se asignan a través de la `--name` bandera para `docker run`. Cuando no se proporciona ningún nombre, el demonio de Docker asigna uno al azar. Puede usar nombres para hacer referencia a contenedores en los comandos de la CLI de Docker; elegir un recuerdo adecuado evita correr `docker ps` para encontrar el nombre o ID asignado automáticamente a un contenedor.

el `docker rename` El comando se usa para cambiar los nombres de los contenedores después de la creación. Se necesitan dos argumentos, el ID actual o el nombre del contenedor de destino y el nuevo nombre para asignar:

```
# docker rename <target ID or name> <new name>
docker rename old_name new_name
```

## Cambiar la política de reinicio

Las políticas de reinicio determinan si los contenedores deben iniciarse automáticamente después de reiniciar su host o iniciar el demonio Docker. Los cuatro [pólizas disponibles](https://docs.docker.com/config/containers/start-containers-automatically) le permite forzar el inicio del contenedor, mantenerlo detenido o iniciar condicionalmente según el código de salida o el estado de ejecución anterior del contenedor.

Docker admite cambiar las políticas de reinicio sobre la marcha. Esto es útil si planea reiniciar su host o el demonio Docker y desea que un determinado contenedor permanezca detenido, o se inicie automáticamente, después del evento específico.

```
docker update --restart unless-stopped demo_container
```

El ejemplo anterior cambia la política de reinicio de `demo_container` en `unless-stopped`. Esta regla inicia el contenedor con el demonio, a menos que se detuviera manualmente antes de que el demonio saliera por última vez.

## Cambiar los límites de recursos de hardware

el `docker update` El comando también se puede usar para cambiar los límites de recursos aplicados a los contenedores. Deberá transmitir uno o más identificadores o nombres de contenedores, así como una lista de indicadores que definan los límites a definir sobre estos contenedores.

[**Leer también** Ziryab responde a Genshin Impact](https://gruposaedal.com/ziryab-responde-a-genshin-impact/)

Las banderas están disponibles para todos los límites de recursos admitidos por [`docker run`](https://docs.docker.com/engine/reference/commandline/run). Aquí hay una lista resumida de las opciones que puede usar:

- **`--blkio-weight`** – Modificar el peso relativo Block IO del contenedor.
- **`--cpus`** – Definir el número de CPU disponibles para el contenedor.
- **`--cpu-shares`** – Definir el peso relativo de la cuota de CPU.
- **`--memory`** – Modificar el límite de memoria del contenedor (ej. `1024M`).
- **`--memory-swap`** – Configure la cantidad de memoria que el contenedor puede intercambiar en el disco; use un tamaño como `1024M` para establecer un límite específico, o `-1` para intercambio ilimitado.
- **`--kernel-memory`** – Cambiar el límite de memoria del núcleo del contenedor. Contenedores de memoria de kernel privados [puede tener un impacto negativo](https://docs.docker.com/config/containers/resource_constraints) otras cargas de trabajo en la máquina host.
- **`--pids-limit`** – Configure la cantidad máxima de ID de procesos permitidos dentro del contenedor, limitando la cantidad de procesos que se pueden iniciar.

Aquí hay un ejemplo de uso. `docker update` para cambiar el límite de memoria y la cantidad de procesadores para dos de sus contenedores:

```
docker update --cpus 4 --memory 1024M first_container second_container
```

Todas las banderas disponibles excepto `--kernel-memory` se puede usar con contenedores de Linux en ejecución. Para cambiar el límite de memoria del núcleo, debe detener el contenedor con `docker stop` primero.

Tenga en cuenta que ninguno de estos indicadores se admite actualmente para contenedores de Windows. Sin embargo, funcionarán con contenedores de Linux que se ejecuten en una máquina host de Windows.

## ¿Cuándo no usar estos comandos?

el `docker update` y `docker rename` Los comandos deben usarse con contenedores que creó manualmente a través de `docker run`. Cuidado con usarlos con contenedores de otras herramientas como `docker-compose`.

Cambiar el nombre de un contenedor podría dejarlo indetectable por la herramienta de origen, lo que podría dañar otros componentes de su pila. Además, si establece límites de recursos de forma declarativa en un `docker-compose.yml` archivo, ejecutando el `docker-compose up` El comando volverá a aplicar esos límites originales a su contenedor nuevamente.

[**Leer también** ¡Buena oportunidad! Obtenga Amazon Fire HD 10 con hasta un 33% de descuento](https://gruposaedal.com/buena-oportunidad-obtenga-amazon-fire-hd-10-con-hasta-un-33-de-descuento/)

Por lo tanto, debe apegarse a su solución de administración de contenedores existente si está utilizando una. Para Compose, esto significa cambiar los nombres de los contenedores y los límites de recursos en su `docker-compose.yml` archivo, luego ejecutando `docker-compose up -d` para aplicar automáticamente el cambio. Esto garantiza que no dejará contenedores huérfanos de forma involuntaria ni causará efectos secundarios no deseados.

## ¿Qué pasa con otras propiedades (imagen/puertos/volúmenes)?

Los límites de hardware, las políticas de recursos y los nombres de los contenedores son los únicos ajustes de configuración que la CLI de Docker le permite cambiar. No puede cambiar la imagen de un contenedor en ejecución; tampoco puede cambiar fácilmente otras opciones, como enlaces de puertos y volúmenes.

Debe crear otro contenedor si estos valores se vuelven obsoletos. Destruye tu instancia actual y usa `docker run` para iniciar un reemplazo con su nueva imagen y la configuración corregida.



Debido a que se supone que los contenedores no tienen estado y son efímeros, debería poder reemplazarlos en cualquier momento. Use volúmenes para almacenar datos de contenedores persistentes; este mecanismo le permite adjuntar archivos con estado al nuevo contenedor repitiendo el `-v` banderas cambiadas a originales `docker run` pedido:

```
docker run -v config-volume:/usr/lib/config --name demo example-image:v1

docker rm demo

# Existing data in /usr/lib/config retained
docker run -v config-volume:/usr/lib/config --name demo2 example-image:v2
```

## Zona de peligro: modificación de otras propiedades del contenedor

Si bien debe intentar reemplazar los contenedores siempre que sea posible, es posible modificar las propiedades de los contenedores existentes editando directamente los archivos de configuración de Docker. Tenga cuidado al usar este método: no es compatible en absoluto y una edición fuera de lugar podría romper su contenedor.

Si bien esta opción proporciona una forma de modificar arbitrariamente los contenedores existentes, no funcionará mientras se estén ejecutando. Utilizar el `docker stop my-container` para detener el contenedor que desea editar y, a continuación, continúe realizando los cambios.



Los archivos de configuración del contenedor tienen la siguiente ruta en su host:

```
/var/lib/docker/containers/<container id>/config.v2.json
```

Debe conocer el ID completo del contenedor, no la versión truncada que muestra `docker ps`. Puedes usar el `docker inspect` comando para obtener esto:

```
docker inspect <short id or name> | jq | grep Id
```

![img](https://www.cloudsavvyit.com/p/uploads/2022/01/c484bc4a.png?trim=1,1&bg-color=000&pad=1,1)

Una vez llegado a un contenedor `config.v2.json`, puede abrirlo en un editor de texto para realizar los cambios necesarios. El JSON almacena la configuración del contenedor creada en tiempo de ejecución `docker run`. Puede editar el contenido para cambiar propiedades como enlaces de puerto, variables de entorno, volúmenes y el punto de entrada y control del contenedor.

[**Leer también** Las nuevas tabletas Fire de Amazon ejecutan una versión más nueva y desactualizada de Android – Review Geek](https://gruposaedal.com/las-nuevas-tabletas-fire-de-amazon-ejecutan-una-version-mas-nueva-y-desactualizada-de-android-review-geek/)

Para agregar un enlace de puerto, busque el `PortBindings` ingrese el archivo, luego inserte un nuevo elemento en el objeto:

```
{
    "PortBindings": {
        "80/tcp": {
            "HostIp": "",
            "HostPort": "8080"
        }
    }
}
```

Aquí, el puerto 80 del contenedor se une al puerto 8080 del host. Es igual de fácil agregar variables de entorno: encuentre el `Env` clave, luego inserte nuevos elementos en la matriz:

```
{
    "Env": [
        "FOO=bar",
        "CUSTOM_VARIABLE=example"
    ]
}
```

Una vez que los cambios estén completos, reinicie el servicio Docker y su contenedor:

```
sudo service docker restart

docker start my-container
```

El contenedor ahora se ejecutará con su configuración actualizada.

## Conclusión

Los contenedores Docker están destinados a ser unidades efímeras que reemplaza cuando su configuración se vuelve obsoleta. A pesar de esta intención, existen escenarios en los que es necesario modificar un contenedor existente. Docker maneja los casos de uso más comunes (cambios de nombre y ajustes de límite de recursos en tiempo real) a través de comandos CLI integrados como `docker update`.

Cuando desee cambiar otra propiedad, siempre intente reemplazar el contenedor como primer curso de acción. Esto minimiza el riesgo de romper objetos y se ajusta al modelo de inmutabilidad de Docker. Si se encuentra en una situación en la que es necesario editar un contenedor existente, puede editar manualmente los archivos de configuración internos de Docker como se muestra arriba.



Finalmente, no olvide que los contenedores administrados por otras herramientas del ecosistema como Docker Compose deben modificarse utilizando estos mecanismos. De lo contrario, es posible que los contenedores se queden huérfanos o se sobrescriban inesperadamente si la herramienta desconoce los cambios que ha realizado.