## ¿Qué es Docker?

**Docker** es un set de productos de plataforma como servicio que usa la virtualización de Sistema Operativa para servir software como paquetes, los que denomina contenedores.

Los contenedores Docker están aislados unos de otros y contienen su propio software, librerías, archivos de configuración, ... Pero, a pesar de estar aislados, pueden comunicarse entre ellos si así se desea.

## Componentes básicos de Docker

* **Docker registry.-** Servicio de registro de imágenes Docker. Puede ser público ([Docker Hub]((https://hub.docker.com/))) o propio de tu empresa.
* **Docker engine.-** Es la pieza fundamental. Se encarga, principalmente, de la orquestación de las imágenes y contenedores. Está compuesto por:

    - Docker daemon.
    - Docker Host (API).
    - Docker CLI.

![docker-engine](./../images/docker-engine.png)

* **Docker image.-** Se podría decir que es la plantilla para la construcción de un contenedor. Es un componente estático que únicamente contiene un SO base y el conjunto de componentes, librerías, instrucciones, ... que aportarán funcionalidad.
* **Docker container.-** Es la instancia de una imagen

[ 👉 Siguente - Imágenes y Contenedores](images-and-containers.md)
