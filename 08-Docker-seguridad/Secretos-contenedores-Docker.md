# Gestión de secretos en contenedores Docker

Las soluciones del mundo legacy para la gestión de secretos generalmente se basan en tener almacenados estos secretos en el sistema de manera permanente, o bien disponer de software que se encargue de pedir a terceros dichos secretos.

Si analizamos la posibilidad de generar imágenes con secretos almacenados, en su definición observaremos los siguientes problemas:

- Las imágenes de los contenedores deben de ser inmutables. Es decir, las imágenes se crean una vez y esa misma imagen debe funcionar en cualquier entorno. Dicho esto, es evidente que si la imagen se crea con unas credenciales directamente grabadas en su definición tenemos dos posibilidades:

    -  O tenemos que compartir esas credenciales en múltiples entornos, y estaremos incurriendo en problemas graves de confidencialidad y segmentación de privilegios;
    - O bien tenemos que crear imágenes distintas para cada entorno ignorando la inmutabilidad que deben de tener las imágenes.

- La solución consistente en pedir secretos en tiempo de ejecución (run-time) mediante algún software incluido en la imagen o imágenes sidekick soluciona el problema anterior, ya que la imagen es inmutable y no contiene información sensible. Este hecho en sí, es un problema. Si la imágen no tiene información sensible, ¿los secretos se pueden consultar sin realizar un proceso de autenticación/autorización previo?. 

## Requisitos

Cuando nos enfrentamos al reto de la gestión segura de seguridad, nos impusimos los siguientes requisitos:

- El sistema que proporcione los certificados debe funcionar en cualquier plataforma de Docker, independientemente del orquestador usado o de la cloud. Por lo tanto, no podíamos usar métodos técnicos solo existentes en orquestadores como Kubernetes, Rancher u OpenShift, o servicios propios de clouds públicas como los ofrecidos por AWS y GCE.
- Uso de certificados de corta duración para la autenticación de microservicios. Para minimizar el impacto en caso de robo de credenciales, se requiere el uso de certificados de corta duración para la autenticación de nuestros servicios.
- Los certificados se deberían generar automáticamente en cada arranque de los contenedores. Al ser certificados de corta duración, siempre deben generarse certificados válidos para cada conexión.
- Los certificados se generarían por servicio y por entorno, no por contenedor. En este caso, llamamos servicio a un conjunto de contenedores que realicen una tarea en concreto. Si tenemos un servicio llamado “timeline”, se generará un certificado válido para cualquier contenedor del servicio “timeline”.
- Los desarrolladores no deberían acceder a credenciales de los entornos productivos. Es decir, el sistema que proporcione los credenciales tiene que identificar los contenedores sin necesidad de que los desarrolladores que crean las imágenes para los contenedores conozcan información sensible.


Por ejemplo, podemos usar etiquetas para identificar a un contenedor como una aplicación del entorno de producción y que ejecuta la aplicación timeline de la siguiente manera:

environment=production
app=timeline
Las etiquetas pueden ser configuradas en el momento de creación de la imágen o en el momento de la ejecución del contenedor. El momento y la forma de asignar estas etiquetas es importante ya que identifican al contenedor. En base a esta identificación se le asignará un certificado para un uso concreto. Queda por lo tanto claro que el sistema que asigna las etiquetas debe ser auditado y correctamente protegido, ya que la asignación incorrecta de etiquetas puede derivar en acceso a certificados con más privilegios de los debidos.
Generalmente, estas etiquetas deben ser gestionadas por el orquestador que lance los contenedores. Estos orquestadores ya introducen varias etiquetas que pueden ayudar a identificar el entorno concreto y el servicio al que pertenece el contenedor. No obstante, las etiquetas pueden ser configuradas por otros actores, e incluso pueden ser configuradas en la propia imagen antes de llegar al orquestador. Todo esto siempre que se utilicen métodos para validar que las etiquetas no han sido alteradas por terceros. Véase Docker notary como herramienta para garantizar la integridad de las imágenes.

Para generar los certificados, el inyector se autentica con Vault usando un token previamente configurado que le permita realizar peticiones al módulo de PKI para los dominios requeridos en el entorno específico (por ejemplo: timeline.production.example.io y para el entorno de producción).

Vault tiene varios mecanismos de autenticación, siendo el uso de tokens el más simple. Los tokens se generan asociados a un rol y caducan. Este token se le debe proporcionar al inyector en su configuración y su generación y renovación debe automatizarse en el despliegue del Docker host o bien usando mecanismos proporcionados por un orquestador o proveedor de cloud como por ejemplo secrets-bridge de Rancher.

Una vez obtenido el certificado, el inyector introducirá el certificado en un directorio del filesystem del contenedor. Este directorio puede ser el filesystem raíz del contenedor o un volumen definido para este cometido. Lo recomendable es que sea un volumen en memoria para que no existan certificados escritos a disco y el almacenamiento de estos sea lo más volátil posible.

## Flujo del proceso

1. Cuando un contenedor se instancia en un host, el inyector recibirá un evento desde Docker indicando que un container ha arrancado, junto con los labels de este contenedor.

2. El inyector usará los label de identificación para construir un certificado válido y realizará una petición vía HTTP REST a Vault para que este genere un certificado de corta duración.

3. El inyector introducirá en la ruta especificada en su configuración la clave pública, la privada y el certificado de la “certification authority” que ha firmado el certificado. Un proceso ejecutado en el contenedor deberá procesar estos ficheros y adaptar el formato a el que necesite el software. Por ejemplo, si la aplicación es Java, se deberá generar un keystore y un truststore.

4. Finalmente la aplicación principal del contenedor será ejecutada, y se conectará al servicio requerido de manera segura usando el certificado recién creado.