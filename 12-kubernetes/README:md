# Introducción a Kubernetes: Arquitectura

La arquitectura y componentes de Kubernetes.

Kubernetes es un sistema “open source” para automatización de configuración y manejo de contenedores. La lista de componentes es como sigue:

- **“Pods”**: Este es el elemento más pequeño de Kubernetes. El término en inglés “pod” significa vaina, como las vainas de guisantes. Muy apropiado, ya que, así como una vaina puede contener varios guisantes, cada “pod” puede contener varios contenedores. Los contenedores dentro del “pod” comparten todos los recursos, incluyendo número de dirección IP. Cada “pod” solo puede tener un número de dirección IP.
    
- Nodos “master” y “worker”: Estos servidores tienen instalado alguna tecnología de contenedores y kubelet, esto es Kubernetes. Hay dos clases de nodos, el “master” y el “worker”.

         - Nodos “master” : Este nodo es un servidor Linux con Kubernetes instalado que administra el “cluster” de Kubernetes. Para administrar el cluster de Kubernetes utiliza base de datos etcd, kube-apiserver, kube-scheduler, kube-controller-manager y cloud controller-manager. En producción, la única función del nodo “master” es la administración y manejo de nodos “workers” y no opera los “pods” con contenedores para las aplicaciones del usuario.

                     - kube-apiserver: Este es el corazón de Kubernetes. Todos los comandos de manejo de Kubernetes son enviados a este componente. Este es el único componente que se comunica con la base de datos etcd.
                     - etcd: Este es el componente que guarda la configuración de todos los componentes de Kubernetes.
                     - kube-scheduler: Este componentes se encarga de distribuir los “pods” entre los nodos de Kubernetes.
                     - kube-controller-manager: Este componente se asegura de que el estado del cluster esté en conformidad con las configuraciones del “cluster” de Kubernetes. Cuando detecta un diferencia, escala los pods de alguna aplicación de acuerdo a su configuración.
                     - cloud-controller-manager: Maneja configuraciones específicas a tecnologías en la nube (por ejemplo: Azure, AWS, etc.).

        - Nodos “worker”: Estos nodos son los que operan los “pods” con las aplicaciones del usuario. kubelet interactúa con software de contenedores como Docker y es el componente que se comunica con kube-apiserver sobre la condición de los pods en el nodo. kube-proxy maneja la conectividad con la red.

        - Servicios: Expone una aplicación que se ejecuta en un grupo de “pods” como un servicio de red. Distribuye el tráfico a los “pods” de acuerdo a su configuración. Por ejemplo: uno de los servicios que se configuran en zalando/postgres-operator se asegura de que se comunique con el pod “master”, o el único pod que acepta inserción y actualización de datos.

        - “Namespace”: Cada uno de los elementos que se crean en Kubernetes deben habitar en una agrupación abstracta llamada “namespace”. Los privilegios a los usuarios y las cuentas de servicios pueden ser restringidos por “namespace”.

        - Red: Dentro de Kubernetes existe una red virtual. Los diferentes elementos de Kubernetes tienen restringido la comunicación de acuerdo con las políticas de red asignadas en conjunto con los roles que tengan vinculados en determinados “namespace”.

        - Almacenamiento: Volúmenes persistentes y sus reclamaciones permiten que los datos importantes que residan en la aplicación sobrevivan al volátil “pod” cuando este es removido.

Esta es una descripción básica de lo que es Kubernetes. En siguientes artículos seguiré compartiendo mas sobre como opera Kubernetes. Para más información, puede ir a la página oficial de documentación de Kubernetes.

