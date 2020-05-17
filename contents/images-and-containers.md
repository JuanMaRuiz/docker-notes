## Imágenes

Básicamente, una imagen es una plantilla que define todo lo que un contenedor va a tener: Sistema Operativo, lenguaje de programación y versión del mismo, ...

Las imágenes, pueden ser:

* Públicas => Como aquellas publicadas en Docker Hub.
* Privadas => Aquellas publicadas en un registro privado de tu empresa en el Docker hub publicadas como no públicas.
* Locales => Puedes crear tu propia imagen en local usando tu propio `Dockerfile`.

## Contendores

Un contenedor de Docker es básicamente una instancia de una imagen de Docker. Si la imagen es la plantilla que define cómo está estructurado, qué funcionalidades, software, ... va a tener un contenedor, el contenedor es una instancia construída a partid de dicha plantilla.

Para construir un contenedor puedes utilizar una imagen predefinida que descargues de Docker Hub o del registro de imágenes de tu empresa o una imagen creada por ti. Las imágenes están definidas por el `Dockerfile`

### Comandos útiles para trabajar con contenedores

**Para inspeccionar contenedores**

Si necesitamos información específica sobre algún contenedor:

* `docker inspect <container-id> | grep Source`: nos da la ubicación del stdout y stderr del contenedor.
* `docker logs --follow <containerid>`: Esto sigue la salida del contenedor en ejecución. Esto es útil si no se configuró un controlador de registro en el daemon docker.
* `docker inspect <imageid>`: Para inspeccionar imágenes

Docker inspect admite las plantillas Go a través de la opción `--format` . Esto permite una mejor gestión de la información que queremos obtener, sin tener que recurrir a las herramientas tradicionales `pipe` / `sed` / `grep`. Por ejemplo:

```bash
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' <contenedor>: imprime la ip del contenedor
$ docker inspect --format '{{ .State.Pid }}' <contenedor>: Imprime el PID del contenedor
$ docker inspect --format 'Container {{ .Name }} listens on {{ .NetworkSettings.IPAddress }}:{{ range $index,
$elem := .Config.ExposedPorts }}{{ $index }}{{ end }}' <contenedor1> <contenedor2>: ejemplo de formato avanzado customizado
```

**Para ver la utilización de recursos del sistema**

```bash
docker stats
$ docker stats <contenedor>
$ docker stats $(docker ps --format '{{.Names}}')
$ docker top <contenedor>
```

**Actualización de la configuración de un contenedor en marcha**

Puede darse el caso en el que necesitemos actualizar la configuración de un contenedor en ejecución, para ello, tenemos el comando:

```bash
docker update
```

A través de este comando podemos cambiar parámetros de un contenedor como, por ejemplo, cambiar la política de reinicio del contenedor:

```bash
docker update --restart=on-failure:3 abebf7571666 hopeful_morse
```

## Buenas prácticas con contenedores

* **Una imagen por contenedor**.- La idea es que los contenedores tengan el mismo ciclo de vida que la aplicación que contienen y a la vez que éstos sean efímeros (que podamos destruirlos, levantarlos, levantar varias instancias del mismo contenedor,...).
* **Agrupar instrucciones** en una misma capa. Cada una de las instrucciones (línea) presentes en el Dockerfile crea una capa de construcción del contenedor. Esto hace que el peso del contenedor sea mayor. Un ejemplo de estas instrucciones (mala práctica) sería:

```
RUN apt-get update
RUN apt-get install -y nginx
# cada una de estas líneas crea una capa de construcción del contenedor
```

Las instrucciones de arriba podrían quedarse de la siguiente manera:

```
RUN apt-get update && \
    apt-get install -y nginx
# creamos una única capa en el contenedor
```

* **Eliminar herramientas innecesarias**.- En ocasiones es necesario instalar herramientas en el contenedor necesarias para la descarga de recursos o construcción de la aplicación. Estas herramientas, como `unzip`, `wget`,..., no son necesarias para mantener la aplicación corriendo, por lo que una vez que hayan realizado su función habría que desinstalarlas.
* **Iniciar los contenedores en modo sólo lectura**.- Es una buena práctica iniciar el contenedor en modo sólo lectura. Para ello, utilizar junto al comando docker run con el flag `read-only`:
*
```bash
docker run -d --read-only nginx
```

Si el contenedor necesita escribir en el sistema de ficheros, se puede proveer un [volumen](volumes.md) para evitar errores y además, hacer persistente los cambios una vez muera el contenedor.

* **Utilizar imágenes base reducidas**.-
* **No utilizar la etiqueta `latest`**.- La etiqueta latest es la que se utiliza por defecto, cuando no se especifica ninguna otra etiqueta.
    ¿Por qué no utilizarla? Porque la etiqueta latest apuntará a una imagen diferente cuando se publique una nueva versión y, por lo tanto, cada vez que realicemos la build de una imagen ésta estaría utilizando una versión diferente lo cual podría tener efectos no deseados.

### Cache

Al realizar la construcción de una imagen Docker, se crean una serie de capas de construcción. Estas capas funcionan como "caché" y ayudan al daemon de Docker a reutilizar dichas capas en construcciones futuras.

Aunque esto es muy útil ya que se disminuye el tiempo de construcción de nuevas imágenes, en algunas ocasiones da problemas. Por eso Docker provee de una herramienta para forzar a que la construcción de la imagen no utilice la caché:

```
docker build --no-cache -t image-name .
```

[ 👉 Siguente - Volúmenes](volumes.md)