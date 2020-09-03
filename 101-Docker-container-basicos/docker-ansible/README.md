# Trabajando con Dockerfiles y ansible

Los archivos Docker contienen las instrucciones que Docker utiliza para construir contenedores.

- "docker build" construye Dockerfiles y genera imágenes de contenedor.
- "docker images" enumera todas las imágenes presentes en el sistema.
- "docker Run" ejecuta imágenes creadas.
- "docker ps -a" enumera todos los contenedores, tanto en ejecución como detenidos.

Al desarrollar Dockerfiles para contener sus propias aplicaciones, es probable que
desea familiarizarse con Docker CLI y cómo funciona el proceso desde un manual
perspectiva. Pero al construir las imágenes finales y ejecutarlas en sus servidores,
Ansible puede ayudar a facilitar el proceso



### Usando Ansible para construir images de docker.

Ansible tiene módulos Docker integrados que se integran perfectamente con Docker para contenedor
administración. Los vamos a usar para automatizar la construcción y el funcionamiento del
contenedor (administrado por el Dockerfile) que acabamos de crear.
Mueva el Dockerfile que tenía a un subdirectorio y cree un nuevo ansible playbook
(llámelo main.yml) en el directorio raíz del proyecto. El diseño del directorio debería verse así:

```bash
docker/
	main.yml
	test/
		Dockerfile
```

Inside the new playbook main.yml, add the following:


```yml
---
- hosts: localhost
  connection: local
  
  tasks:
    name: Ensure Docker image is built from the test Dockerfile.
    docker_image:
      name: test
      source: build
      build:
        path: test 
      state: present
```



El playbook usa el módulo docker_image para construir una imagen. Proporcione un nombre para
la imagen, dile a Ansible que la fuente de la imagen es una compilación, luego proporciona la ruta
al Dockerfile en los parámetros de compilación (en este caso, dentro del directorio de prueba).

Finalmente, diga a Ansible a través del parámetro de estado que la imagen debe estar presente, para garantizar que se crea la imagen.
La integración de Docker de Ansible puede requerir que instales una  adicional
biblioteca Docker de Python en el sistema que ejecuta el  Ansible playbook . 

Por ejemplo, en ArchLinux, si obtiene el error "no se pudo importar el módulo de Python", usted
necesitará instalar el paquete python2-docker.

 En otras distribuciones, es posible que deba instalar la biblioteca de Docker Python a través de Pip ( pip install docker ).

Ejecute el playbook:

```
$ ansible-playbook main.yml
```

y luego enumere todos los Docker imágenes (`$ docker images`). Si todo fue correcot, debería ver una nueva imagen de prueba en la lista.

Sin embargo, ejecute `docker ps -a` nuevamente, y verá que la nueva imagen de prueba nunca fue
ejecutar y está ausente de la salida. Remediamos eso agregando otra tarea a nuestro  Ansible playbook:

```
name: Ensure the test container is running.
docker_container:
image: test:latest
name: test
state: started
```



Si vuelve a ejecutar el playbook  Ansible iniciará el contenedor Docker. Revisa la lista
de contenedores con `docker ps -a`, y notará que el contenedor de `test` está nuevamente presente.
Puede eliminar el contenedor y la imagen a través de ansible cambiando el estado
parámetro ausente para ambas tareas.



Este playbook  asume que tienes Docker y Ansible instalados en cualquier host que esté utilizando para probar los contenedores Docker. Si este no es el caso, es posible que deba modificar el ejemplo para que el playbook de Ansible esté dirigido a los hosts correctos y usando la configuración de conexión correcta. Adicionalmente, si la cuenta de usuario con la que ejecuta playbook no puede ejecutar Docker
comandos, es posible que necesite usar convertido playbook.