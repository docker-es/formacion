# Introducción

[Ansible](https://www.ansible.com/) es una herramienta de administración automatizada de ordenadores, creada por Michael DeHaan en 2012. La primera versión se publicó en marzo de 2012 y la versión 1.0 se publicó en febrero de 2013. RedHat compró Ansible en octubre de 2015.

Ansible es software libre GPL 3.0 y es la base de [Ansible Tower](https://www.ansible.com/products/tower), un producto comercial de RedHat.



# Instación de Ansible
```
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt full-upgrade -yq
sudo DEBIAN_FRONTEND=noninteractive apt install software-properties-common -yq
sudo apt-add-repository ppa:ansible/ansible -y
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt install ansible -yq
```

## Nodos administrados
CC [Bartolomé Sintes Marco](http://www.mclibre.org/consultar/webapps/lecciones/ansible-1.html)

En los nodos administrados no es necesario instalar Ansible.


## Configuración inicial y primera conexión

La máquina de control Ansible se conecta con los nodos administrados por SSH. Para poder realizar esa conexión y comprobar su correcto funcionamiento, realice estos pasos en la máquina de maestra:

Edite el archivo /etc/ansible/hosts para añadir la dirección IP de los nodos administrados:

    sudo nano /etc/ansible/hosts

En el archivo /etc/ansible/hosts Ansible permite crear grupos de nodos. Los nombre de los grupos se escriben entre corchetes ([XXX]).

Las opciones de configuración se pueden escribir en grupo [XXX-vars] (y afectan a todos los nodos incluidos en el grupo:

```
    [clients]
    192.168.1.16

    [clients:vars]
    ansible_python_interpreter=/usr/bin/python3
```
- Como Ubuntu 18.04.2 incluye Python 3 (pero no Python 2), es necesario indicar a Ansible que utilice python3..

- Las opciones de configuración se pueden escribir en cada nodo (y afectan únicamente a dicho nodo):
```
    [clients]
    192.168.1.16 ansible_python_interpreter=/usr/bin/python3
```
- Ansible se comunica con los nodos mediante SSH por lo que se debe crear una clave SSH en la máquina de control y enviarla a los nodos administrados:
```
    ssh-keygen -t rsa
    ssh-copy-id mclibre@192.168.1.16
```
- Compruebe que se puede conectar con los nodos mediante la orden:
```
    ansible all -m ping
```
- Por cada nodo administrado debe mostrarse una respuesta similar a esta:

```
    192.168.1.16 | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
```
## Playbooks de Ansible

Ansible trabaja mediante playboooks. Los playbooks son ficheros YAML que describen las operaciones a realizar sobre los nodos administrados.

### Ejemplo 1: Ejecución de comandos de shell

Para ejecutar comandos de la shell, Ansible dispone del módulo [shell](https://docs.ansible.com/ansible/latest/modules/shell_module.html).

- Cree el archivo playbook-1-1.yaml con el siguiente contenido:
```
    ---
    - hosts: 'clients'
      tasks:
      - name: 'ejemplo 1'
        shell: 'ls -al >> prueba.txt'
```
- Ejecute el archivo playbook-1-1.yaml:
```
ansible-playbook playbook-1-1.yaml
```
- Obtendrá una respuesta similar a esta:
```
    PLAY [clients] ******************************************************************
    TASK [Gathering facts] **********************************************************
    ok: [192.168.1.16]
    TASK [ejemplo 1] ****************************************************************
    changed: [192.168.1.16]
    PLAY RECAP **********************************************************************
    192.168.1.16                 : ok=2     changed=1   unreachable=0    failed=0
```
- En el nodo administrado puede comprobar que se ha creado el fichero listado.txt con el listado del directorio.
- Para obtener más información sobre la ejecución del playbook, ejecute el archivo con la opción--verbose:
```
    ansible-playbook playbook-1-1.yaml --verbose
```

### Ejemplo 2: Actualización del sistema

Para tareas de mantenimiento de paquetes, Ansible dispone del módulo apt.

Cree el archivo __playbook-1-2.yaml__ con el siguiente contenido:
```
    ---
    - hosts: 'clients'
      tasks:
      - name: 'ejemplo 2'
        become: true
        apt:
          update_cache: true
          upgrade: true
```

Ejecute el archivo playbook-1-2.yaml con la opción -K. La opción -K es necesaria porque apt requiere lcontraseña del usuario:
```
    ansible-playbook playbook-1-2.yaml -K
```
Obtendrá una respuesta similar a esta:
```
    SUDO password:

    PLAY [clients] ******************************************************************

    TASK [Gathering facts] **********************************************************
    ok: [192.168.1.16]

    TASK [ejemplo 2] ****************************************************************
    [WARNING]: Could not find aptitude. Using apt-get instead.
    changed: [192.168.1.16]

    PLAY RECAP **********************************************************************
    192.168.1.16                 : ok=2     changed=1   unreachable=0    failed=0
```
Nota:
    El aviso [WARNING] se debe a que el paquete aptitude no está instalado.

### Ejemplo 3: Instalación y desinstalación de paquetes

Para tareas de mantenimiento de paquetes, Ansible dispone del módulo apt.

En el ejemplo siguiente, se instala el paquete aptitude, un interfaz en modo texto de APT.

- Compruebe en el nodo administrado que el paquete aptitude no está instalado con la orden:
```
    apt-cache policy aptitude
```
- Si aptitude no está instalado, se mostrará la respuesta:
```
    aptitude:
      Instalados: (ninguno)
      ...
```
- Cree el archivo playbook-1-3.yaml con el siguiente contenido:
```
    ---
    - hosts: 'clients'
      tasks:
      - name: 'ejemplo 3'
        become: true
        apt:
          name: 'aptitude'
          state: 'present'
```
- Ejecute el archivo __playbook-1-3.yaml__ con la opción -K. La opción -K es necesaria porque apt requiere la contraseña del usuario:
```
    ansible-playbook playbook-1-3.yaml -K

    Compruebe en el nodo administrado que el paquete aptitude sí que está ahora instalado con la orden:

    apt-cache policy aptitude

    Si aptitude está instalado, se mostrará una respuesta similar a esta:

    aptitude:
      Instalados: 0.8.10-6ubuntu1
      ...
```
- Para desinstalar el paquete, el playbook sería:
```
---
    - hosts: 'clients'
      tasks:
      - name: 'ejemplo 3'
        become: true
        apt:
          name: 'aptitude'
          state: 'absent'
```


# Buenas prácticas

El formato YAML que utiliza Ansible admite diferentes sintaxis. Aunque Ansible no dispone de un libro de estilo oficial, sí que se recomienda seguir ciertas directrices:

    el manual de Ansible incluye un apartado de buenas [prácticas](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
    la empresa WhiteCloud ofrece una [guía de estilo](https://github.com/whitecloud/ansible-styleguide)

## Solicitud de contraseña de superusuario

Si un playbook contiene tareas que no requieren privilegios de superusuario, para ejecutar un playbook hay que escribir el comando:
```
    ansible-playbook playbook.yaml
```
Pero si un playbook contiene una tarea que requiere privilegios de superusuario, al ejecutar el playbook se debe añadir la opción -K, para que Ansible solicite la contraseña de superusuario.
```
    ansible-playbook playbook.yaml -K
```
Valores fijos, variables o entrada de usuario

Un playbook puede incluir valores como valores fijos, como variables o solicitar los valores al usuario en la ejecución.

Los siguientes playbook añaden a un fichero de texto la línea "¡Hola, mundo!", utilizando el módulo lineinfile. El nombre del fichero de texto se define de forma distinta en cada uno de ellos

- Mediante un valor fijo:
```
    ---
    - hosts: 'clients'
      tasks:
      - name: 'Añade una línea a un fichero'
        lineinfile:
          path: '~/prueba.text'
          create: true
          state: 'present'
          line: '¡Hola, mundo!'
```
- Mediante una variable definida al principio del playbook:
```
    ---
    - hosts: 'clients'
      vars:
        file_name: 'prueba.txt'

      tasks:
      - name: 'Añade una línea a un fichero'
        lineinfile:
          path: '~/{{ file_name }}'
          create: true
          state: 'present'
          line: '¡Hola, mundo!'
```
- Mediante un valor solicitado al usuario:
```
    ---
    - hosts: 'clients'
      vars_prompt:
      - name: 'file_name'
        prompt: 'File name'
        private: false

      tasks:
      - name: 'Añade una línea a un fichero'
        lineinfile:
          path: '~/{{ file_name }}'
          create: true
          state: 'present'
          line: '¡Hola, mundo!'
```

