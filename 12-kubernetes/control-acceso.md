# Control de Acceso RBAC [(CC El lado del mal )](http://www.elladodelmal.com/2019/03/kubernetes-como-gestionar-autorizacion.html)

## Intro

El Control de Acceso Basado en Roles (Role Based Access Control - RBAC), es uno de los mecanismos que nos ofrece Kubernetes a la hora de autorizar el acceso a sus recursos (pods, nodos, secretos, etcétera). Éste es de hecho el mecanismo de autorización más usado (disponible en versión estable desde la versión 1.8 de Kubernetes).

Prácticamente toda interacción con los recursos se realiza a través de su servidor API, lo que implica, que al final todo se limita a hacer peticiones HTTP a dicho servidor (componente esencial del nodo/s maestro o Control Panel). Para hablar de autorización basada en roles, lo primero es hablar de los roles en sí. Vamos a ello.

## Roles en Kubernetes

En Kubernetes existen dos tipos llamados Role y ClusterRole. La mayor diferencia entre ambos es que Role pertenece a un nombre de espacio con concreto, mientras que el ClusterRole es global al clúster. Por lo que, en el caso de ClusterRole, su nombre debe ser único ya que pertenece al clúster. En el caso del Role, dos espacios de nombre distintos pueden tener un Role con el mismo nombre.

Otra de las diferencias que cabe mencionar, es que Role permite dar acceso a recursos que están dentro del mismo espacio de nombres, mientras que ClusterRole, además de poder dar acceso a recursos en cualquier espacio de nombres, también puede dar a acceso a recursos del clúster, como nodos entre otros.
Ahora que sabemos los tipos de roles, lo siguiente es saber quién le podemos asignar dichos roles. En este caso tenemos: cuentas de usuario, cuentas de servicio y grupos.

- Las cuentas de usuarios, como puedes imaginar son cuentas asignadas a un usuario en particular.
- las cuentas de servicios son usadas por procesos. Por ejemplo, imagina que nuestra aplicación necesita acceder de forma programática a recursos del clúster, para ello usaríamos una cuenta de servicio.

Por último, necesitamos el "pegamento" que enlace un role a una cuenta (de usuario o servicio) o grupo. Para ello existen dos recursos en Kubernetes: ***RoleBinding*** y ***ClusterRoleBinding***. 
 
 - RoleBinding puede referenciar un role que esté en el mismo espacio de nombres.
 - ClusterRoleBinding puede referenciar a cualquier role en cualquier espacio de nombres y asignar permisos de forma global.

## RBAC en Minikube

Una vez hemos visto todos elementos que entran en juego en el campo del RBAC, veamos algunos ejemplos. Para estos, vamos a usar minikube. Asegúrate que RBAC está habilitado, para ello puedes arrancar minikube con el siguiente comando: 

Vamos a crear dos usuarios: cybercaronte y tuxotron, ambos pertenecientes a un grupo llamado developers.

1. - Creamos los certificados para cada usuario:

    mkdir ~/certs && cd ~/certs

2. - Creamos la clave y petición de firma para cybercaronte

    openssl genrsa -out cybercaronte.key 2048
    openssl req -new -key cybercaronte.key -out cybercaronte.csr -subj "/CN=cybercaronte/O=developers"

3. - Creamos la clave y petición de firma para tuxotron

    openssl genrsa -out tuxotron.key 2048
    openssl req -new -key tuxotron.key -out tuxotron.csr -subj "/CN=tuxotron/O=developers"

4. - Creamos los certificados para ambos usuarios

    Si tienes instalado minikube en un directorio distinto de ~/.minikube, ajusta el comando

    openssl x509 -req -in cybercaronte.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out cybercaronte.crt -days 90
    openssl x509 -req -in tuxotron.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out tuxotron.crt -days 90

Ya tenemos creado los certificados para ambos usuarios, lo siguiente sería:

5. - Creamos los propios usuarios en nuestro clúster (minikube):

    kubectl config set-credentials cybercaronte --client-certificate=cybercaronte.crt --client-key=cybercaronte.key
    kubectl config set-credentials tuxotron --client-certificate=tuxotron.crt --client-key=tuxotron.key

Para poder crear usuarios tenemos que ser administradores del clúster. Cuando arrancamos minikube por defecto somos administradores, por eso podemos ejecutar los dos comandos anteriores.

Puedes comprobar que dichos usuarios han sido creados con el siguiente comando:

    kubectl config view

O bien, puedes imprimir el contenido del fichero ~/.kube/config

    cat ~/.kube/config

En ambos casos deberías ver los usuarios cybercaronte y tuxotron. Para ejecutar comandos con un usuario u otro, tenemos que decírselo a kubectl a través del contexto. Para ello vamos a añadir dos contextos nuevos, uno para cada usuario.

6. - Creamos dos contextos nuevos:

    kubectl config set-context cybercaronte --cluster=minikube --user=cybercaronte
    kubectl config set-context tuxotron --cluster=minikube --user=tuxotron

Para ver los contextos que tenemos, podemos ejecutar:

kubectl config get-contexts

    CURRENT        NAME       CLUSTER         AUTHINFO    NAMESPACE
     cybercaronte   minikube   cybercaronte
    *minikube       minikube   minikube
     tuxotron       minikube   tuxotron

Como vemos tenemos tres contextos, los dos que acabamos de crear, más minikube, que es al que nos da acceso como administrador al clúster. Además, éste es actualmente el contexto activo, denotado por el asterisco.

7. - Cambiamos de contexto:

Para cambiar de contexto, es decir, para cambiar de usuario y/o clúster (podemos trabajar con varios clústers a la vez), podemos hacerlo con el siguiente comando:

    kubectl config use-context tuxotron

Para comprobar que el contexto ha sido cambiado, o bien ejecutamos el comando `kubectl config get-contexts` y buscamos la entrada con el asterisco:

    kubectl config current-context

Hasta ahora todo lo que hemos hecho es crear dos usuarios, pero no le hemos dado ningún permiso. Por lo que, si intentamos acceder a cualquier recurso, la petición debería ser denegada. Como nuestro contexto actual es tuxotron, intentemos leer los pods del espacio de nombres default y veremos el error:

    kubectl get pods -n default                                                                                                   
    Error from server (Forbidden): pods is forbidden: User "tuxotron" cannot list resource "pods" in API group "" in the namespace "default"

Volvamos a usar nuestro contexto minikube para poder realizar algunas tareas administrativas:

    kubectl config use-context minikube

8. - Creamos espacios de nombres:

Vamos a crear 3 espacios de nombres, uno personal para cada usuario y un tercero al cual llamaremos equipo, en el que ambos usuarios podrán interactuar con recursos:

    kubectl create namespace cybercaronte
    kubectl create namespace tuxotron
    kubectl create namespace equipo

Para comprobar que nuestros espacios de nombres han sido creados:

    kubectl get namespace

    NAME             STATUS     AGE
    cybercaronte    Active    14m
    default        Active    17h
    equipo          Active    14m
    kube-public     Active    17h
    kube-system     Active    17h
    tuxotron        Active    14m

En este ejemplo, que vamos a usar de base para explicar sus componentes, podemos ver un fichero de creación de un role para tener control total total al espacio de nombres de cybercaronte (luego veremos como se crea):

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
        name: cybercaronte-full-control
        namespace: cybercaronte
    rules:
    - apiGroups: [""]
      resources: ["*"]
      verbs: ["*"]

Como todo objeto en Kubernetes, vemos como en este caso tenemos definidos los atributos kind, apiVersion y metadata. Ojo a este caso, como estamos creando un role, tenemos que especificar el espacio de nombres donde queremos crearlo con el atributo namespace dentro de metadata. 

Lo siguiente es definir las reglas (rules). En este case vemos como tenemos entradas bajo rules:

    • apiGroups: aquí especificamos la(s) API(s) que queremos autorizar. Si lo dejamos vacío (""), implica que dicha regla (rule), se aplica a las APIs core de Kubernetes. Alguno de los grupos disponibles, además del core, son: extensions, apps, etc. Si quieres ver todas las APIs disponible en tu clúster, puedes ejecutar el siguiente comando: 

    kubectl api-versions

    • resources: aquí, si queremos aplicar la regla a todos los dispositivos, podemos poner un asterisco, o bien listar los recursos que deseamos. Por ejempo: pods, secrets, configmaps, deployments, services, etc.

    • verbs: en Kubernetes el recibe todas las peticiones es el component API Server, que no es ni más ni menos que un servicio Rest. Por lo que toda petición se basa en un recurso y un verbo: GET, POST, PUT, etc. En el caso de Kubernetes, éste define su propia lista de verbos como son: get, list, watch, create, update, patch, delete, bind, etcetera.

Si creamos ese role y se lo asignamos a un usuario, dicho usuario prácticamente tendría control total sobre el espacio de nombres donde dicho role es creado. Un apunte importante es que podemos añadir varias reglas a un role:

    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
    - apiGroups: ["batch", "extensions"]
      resources: ["jobs"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

En este caso, tenemos dos reglas, la primera indica que podemos leer, listar y observar pods a través del uso de cualquier API de Kubernetes. La segunda indica que podemos obtener, listar, observar, crear, actualizar, parchear y borrar recursos del tipo (Kind) job definidos bajo las APIs batch y extensions.

Ahora sí, vamos a crear los dos roles, uno para el espacio de nombres cybercaronte:

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cybercaronte-full-control
      namespace: cybercaronte
    rules:
    - apiGroups: [""]
      resources: ["*"]
      verbs: ["*"]

Y otro para el espacio de nombres tuxotron:

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tuxotron-full-control
      namespace: tuxotron
    rules:
    - apiGroups: [""]
      resources: ["*"]
      verbs: ["*"]

Creamos dos ficheros (cybercaronte-fullcontrol-role.yaml y tuxotron-fullcontrol-role.yaml) con dicho contenido y ejecuta:

  kubectl apply -f cybercaronte-fullcontrol-role.yaml
  kubectl apply -f tuxotron-fullcontrol-role.yaml

Para comprobar que los roles se han creado:

  kubectl get role -n cybercaronte
  kubectl get role -n tuxotron

Ahora que tenemos los usuarios y los roles creados, tenemos que asignarles los roles a los usuarios. Para ello necesitamos crear los objectos RoleBinding.
Primero vamos a asignar al usuario cybercaronte el role cybercaronte-full-control. Para ello creamos un fichero (cybercaronte-fullcontrol-binding.yaml) con el siguiente contenido

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: full-control
      namespace: cybercaronte
    subjects:
    - kind: User
      name: cybercaronte
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: cybercaronte-full-control
      apiGroup: rbac.authorization.k8s.io

Crea el RoleBinding:

    kubectl apply -f cybercaronte-fullcontrol-binding.yaml

Para probar si funciona, cambiemos de contexto a cybercaronte y a ver si ahora podemos consultar los pods del espacio de nombres cybercaronte:
kubectl config use-context cybercaronte kubectl get pods -n cybercaronte No resources found.
Como podemos ver esta vez no recibimos ningún error. Si intentamos consultar los pods del espacio de nombres tuxotron:

  kubectl get pods -n tuxotron
  Error from server (Forbidden): pods is forbidden: User "cybercaronte" cannot list resource "pods" in API group "" in the namespace "tuxotron"

Vemos como recibimos un error porque el usuario cybercaronte no tiene acceso a los pods del espacio de nombres tuxotron. 

Hagamos lo mismo con el usuario tuxotron. Crea un fichero (tuxotron-fullcontrol-binding.yaml) con el siguiente contenido:

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: full-control
      namespace: tuxotron
    subjects:
      - kind: User
        name: tuxotron
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      - kind: Role
        name: tuxotron-full-control
        apiGroup: rbac.authorization.k8s.io

Antes de poder crear este RoleBinding tenemos que cambiar de contexto a minikube primero:

    kubectl config use-context minikube
  kubectl apply -f tuxotron-fullcontrol-binding.yaml

Vamos a explicar un poco la estructura de este objeto. Lo primero a recalcar es el nombre (metadata.name), si te fijas bien en los dos RoleBindings que hemos creado usan exactamente el mismo nombre. Esto es posible, porque los RoleBindings pertenecen a un nombre de espacio, y en nuestro ejemplo hemos creado cada RoleBidning en distintos espacios de nombre. Si hubiéramos usado el CLusterRoleBinding, esto no sería posible porque en ese caso habría un conflicto de nombres.

El apartado subjects es donde decimos a quién la vamos a asignar el role. El atributo Kind debe ser User, Group o ServiceAccount. Luego tenemos name, éste corresponde al nombre de usuario, grupo o cuenta de servicio y por último el apiGroup que es rbac.authorization.k8s.io

Luego tenemos el apartado roleRef. Aquí es dónde especificamos el role que queremos asignar. El atributo Kind define el tipo de role: Role o ClusterRole. Name es el nombre del role en sí y finalmente el apiGroup que al igual que el caso anterior es rbac.authorization.k8s.io

Por último, veamos un ejemplo donde daremos acceso al espacio de nombres equipo al grupo developers. Vamos a crear un role a nivel de clúster para dar acceso total sobre pods, deployments, servicios, replicasets y secretos:

    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: full-control
    rules:
      - apiGroups: ["extensions", "apps", ""]
        resources: ["pods", "deployments", "secrets", "services", "replicasets"]
        verbs: ["*"]

Copiamos el contenido a un fichero (developers-role.yaml) y ejecuta el siguiente comando:

    kubectl apply -f developers-role.yaml

Un par de apuntes sobre este fichero. Lo primero es que el tipo (Kind) es ClusterRole y como vemos no tiene especificado el espacio de nombres (namespace). Este es un objeto global del clúster. Ahora creemos otro fichero (developers-binding.yaml) con el siguiente contenido:

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: full-control
      namespace: equipo
    subjects:
      - kind: Group
      name: developers
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: full-control
      apiGroup: rbac.authorization.k8s.io

Como vemos, el contenido de este fichero es similar a los anteriores, con par de diferencias. En el apartado subjects, usamos el tipo Group, porque estamos asignando el role a un grupo y no a un usuario. Y en el apartado roleRef, el tipo es ClusterRole y no Role, ya que el role al que estamos haciendo referencia es del tipo ClusterRole. Para crear dicho RoleBinding:

    kubectl apply -f developers-binding.yaml

Recuerda que cuando creamos los usuarios, ambos los creamos bajo la organización developers `/O=developers`. Por lo tanto, cybercaronte y tuxotron deberían tener acceso a los pods, servicios, secretos, replicasets y deployments del espacio de nombres. Comprobemos que cybercaronte tiene acceso:


kubectl config use-context cybercaronte

Switched to context "cybercaronte".

    kubectl get pod -n equipo
    No resources found.

    kubectl get deployment -n equipo
    No resources found.

    kubectl get configmap -n equipo 
    Error from server (Forbidden): configmaps is forbidden: User "cybercaronte" cannot list resource "configmaps" in API group "" in the namespace "equipo"

Como hemos podido comprobar, cuando consultamos los pods y los deployments no recibimos ningún error. “No resources found” es porque no hemos creado nada en el espacio de nombres. Cuando consultamos los configmaps vemos que recibimos un error, ya que ese recurso no tenemos acceso. Si hacemos las mismas pruebas con el usuario tuxotron, los resultados deberían ser el mismo.

Kubernetes no sólo permite el control de acceso a un recurso complete, sino que incluso a parte del mismo. Por ejemplo, podrías dar acceso a los logs de los pods, sin dar acceso a los pods en sí.
