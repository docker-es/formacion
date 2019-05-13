# WEAVESCOPE

## Instalación en Docker

### Single Node

[web](https://www.weave.works/oss/scope/)

Para instalar Scope en un solo nodo, ejecute los siguientes comandos:

```
sudo curl -L git.io/scope -o /usr/local/bin/scope
sudo chmod a+x /usr/local/bin/scope
scope launch
```


Este script descarga y ejecuta una imagen de alcance reciente de Docker Hub. El alcance debe instalarse en cada máquina que desee monitorear.

Después de instalar Scope, abra su navegador en http://localhost:4040.

Si está utilizando docker-machine, puede encontrar la IP ejecutando, 

    $ docker-machine ip <nombre de máquina virtual>.

Dónde, <Nombre de máquina virtual> es el nombre que le dio a su máquina virtual con docker-machine.