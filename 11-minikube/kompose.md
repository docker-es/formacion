---
title: Kompose, Como pasar de Docker Compose a Kubernetes
description: Como usar Kompose
author: Mario Ezquerro
tags: Docker, 
date_published: 2019-07-22
---

# Kompose

Kompose, una herramienta que permite:
- Levantar con un solo comando una aplicación en Kubernetes usando la definición del archivo compose.
- Generar los archivos de deployment y de service necesarios para levantar la aplicación.

La herramienta se encuentra disponible en el sitio de versiones del proyecto en Github: [Aqui](https://github.com/kubernetes/kompose/releases?source=post_page---------------------------)


NOTA:  La transformación del formato Docker Compose al manifiesto de recursos de Kubernetes puede no ser exacta, pero ayuda enormemente cuando se implementa por primera vez una aplicación en Kubernetes.

## Instalación

Tenemos múltiples formas de instalar Kompose. Nuestro método preferido es descargar el binario de la última versión de GitHub.

Nuestra lista completa de métodos de instalación se encuentra en el documento [installation.md](https://github.com/kubernetes/kompose/blob/master/docs/installation.md).

Métodos de instalación:

- Binario (método preferido)
- Go
- CentOS
- Fedora
- openSUSE / SLE
- macOS (homebrew)
- Windows

El metodo General:

Linux and macOS:
```
	# Linux
	curl -L https://github.com/kubernetes/kompose/releases/download/v1.18.0/kompose-linux-amd64 -o kompose

	# macOS
	curl -L https://github.com/kubernetes/kompose/releases/download/v1.18.0/kompose-darwin-amd64 -o kompose

	chmod +x kompose
	sudo mv ./kompose /usr/local/bin/kompose
```



## Levantar directo desde Compose

Si no se quiere generar los archivos de kubernetes, se pueden levantar todos los containers de la aplicación mediante el comando:
	
	$ kompose up

La principal desventaja de usar la herramienta de esta forma es que puede parecer un método “mágico” ya que no sabemos lo que pasa por detrás y a menudo falla si no se siguen las convenciones de Kubernetes para los recursos.

## Generar archivos de Kubernetes

Este comando creará dos archivos por cada servicio en el docker-compose, el “user-api-deployment.yaml” para el deployment, y el “user-api-service.yaml” para el service.

	$ kompose convert

La única contra que tiene generar los archivos con Kompose es la gran cantidad de labels específicas que la herramienta deja en el archivo. Queda en la preferencia de cada uno si las borra o no.
