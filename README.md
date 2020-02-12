# Docker

Los namespaces aíslan el entorno operativo (filesystem, árbol de procesos, ...) de cada contenedor, los cgroups y recursos (CPU, memoria, bloqueos I/O de red, ...)

## Componentes de Docker

* **Docker registry.-** Servicio de registro de imágenes Docker. Puede ser público ([Docker Hub]((https://hub.docker.com/))) o propio de tu empresa.
* **Docker engine.-** Pieza fundamental. Se encarga, principalmente, de la orquestación de las imágenes y contenedores. Está compuesto por:

    - Docker daemon.
    - Docker Host (API)
    - Docker CLI

![docker-engine](/docker-engine.png)

* **Docker image.-** Se podría decir que es la plantilla para la construcción de un contenedor. Es un componente estático que únicamente contiene un SO base y el conjunto de componentes que aportarán funcionalidad.
* **Docker container.-** Es la instancia de una imagen

## Contendores

### Comandos útiles

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

## Imágenes

... TBF

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

Si el contenedor necesita escribir en el sistema de ficheros, se puede proveer un volumen para evitar errores y además, hacer persistente los cambios una vez muera el contenedor.

* **Utilizar imágenes base reducidas**.-
* **No utilizar la etiqueta `latest`**.- La etiqueta latest es la que se utiliza por defecto, cuando no se especifica ninguna otra etiqueta.

¿Por qué no utilizarla? Porqu la etiqueta latest apuntará a una imagen diferente cuando se publique una nueva versión y, por lo tanto, cada vez que realicemos la build de una imagen ésta estaría utilizando una versión diferente lo cual podría tener efectos no deseados.


### Cache

Al realizar la construcción de una imagen Docker, se crean una serie de capas de construcción. Estas capas funcionan como "caché" y ayudan al daemon de Docker a reutilizar dichas capas en construcciones futuras.

Aunque esto es muy útil ya que se disminuye el tiempo de construcción de nuevas imágenes, en algunas ocasiones da problemas. Por eso Docker provee de una herramienta para forzar a que la construcción de la imagen no utilice la caché:

```
docker build --no-cache -t image-name .
```

## Volúmenes

Los contenedores Docker son capaces de trabajar con almacenamiento alternativo. Esto es útil cuando se quieren persistir datos en disco como una los de una base de datos.

Docker nos permite manejar este almacenamiento de dos maneras:

* Utilizando los volúmenes.
* Utilizando contenedores como volúmenes.

Estos volúmenes se utilizan para almacenar y compartir datos (información) entre el host y el contenedor de manera independiente a la vida del contenedor.

Además éstos, los volúmenes, nos facilitan el intercambio de datos entre contenedores, o, que 2 o más contenedores, puedan utilizar la misma fuente de datos.

Los volúmenes no son más que directorios en el host que tienen su réplica en el contenedor.

En el caso de que el directorio que se monte como volúmen ya existiera en el contenedor, su contenido no sería eliminado.

Docker provee de una serie de drivers que permiten la integración de un volumen con servicios de almacenamiento en la nube como Azure FileSystem, S3 de Amazon, IPFS, ...

El driver que utilizar Docker por defecto es el local que permite el uso del propio filesystem del host como volumen.

Cada volumen es identificado con una etiqueta. No es necesario especificar un path en el SO host ya que Docker posee una zona específica donde crea todos los volúmenes dentro del filesystem del host. Dicha zona es diferente dependiendo del SO del host.

### Creación de volúmenes

#### Creación manual de volúmenes y asociación al contenedor
**Ejemplo 1**

**Creación de un volumen al arrancar un contenedor**

Utilizando una imagen de mysql, vamos a lanzar un contenedor. En el commando `run` vamos a especificar que queremos crear un volumen (`data`) asociado a directorio del contenedor `/var/lib/mysql`. Para ello lanzamos el siguiente comando:

```bash
$ docker run -d -v data:/var/lib/mysql -p 3306:3306 mysql
```

Confirmamos que el contenedor está corriendo `docker ps`, en caso afirmativo, confirmamos que se ha creado un volumen `data`:

```bash
$ docker volume ls
```

Deberías ver algo parecido a esto:

![docker-volume-ls](docker-volume-ls.png)

> Si al lanzar el comando `docker volume ls` observaras un sin fin de volúmenes no te asustes, Docker crea volúmenes temporales al construir los contenedores. Para borrar dichos volúmenes basta con que lances el comando `docker volume prune` y confirmar la acción.

Los volúmenes, al igual que los contenedores y las imágenes, pueden inspeccionarse. En este caso debes usar el comando `docker volume inspect [nombre-volumen]`. En nuestro ejemplo sería `[nombre-volumen]` `data`. 

Se puede realizar la creación de un volumen de manera manual y posteriormente (al hacer el run del contenedor) asociarlo a un directorio dentro del contenedor. 

Con el fin de comprobar cómo funcionan los volúmenes vamos a realizar los siguiente:

* Creamos un volumen que montaremos (o asociaremos) al contenedor.
* Nos meteremos dentro del contenedor y crearemos una nueva base de datos.
* Pararemos y borraremos el contenedor para posteriormente volver a crear otro contenedor de mysql al que le montaremos el volumen que creamos en el paso uno. ¿Para qué? Para comprobar que la base de datos creada en el segundo paso está presente en el nuevo contenedor que hemos creado.

Vayamos paso por paso.

Creamos un volumen `docker volume create mysql-db-data` y posteriormente verificamos que se haya creado con `docker volume ls`.

Levantamos nuevamente el contendor de mysql, pero esta vez necesitaremos pasarle más parámetros en la construcción ya que vamos a interactuar con él una vez esté levantado. Le vamos a dar un nombre `--name mysql-db` y le pasaremos la clave para conectar a la bbdd mediante una variable de entorno `-e MYSQL_ROOT_PASSWORD=secret`. Por último, le asociamos el volumen que acabamos de crear para ello le pasamos el flag `mount` con un `src` y `dst`. Donde `src` será el volumen que acabamos de crear y `dst` el directorio dentro del contenedor donde queremos mapear:

```bash
$ docker run --name mysql-db -e MYSQL_ROOT_PASSWORD=my-secret-pw -d --mount src=db-data,dst=/var/lib/mysql mysql
```

Una vez creado el contenedor accedemos a él y creamos una base datos dentro del mismo:

```bash
$ docker exec -it my-sql-data mysql -p
```

Con este comando lo que estamos diciendo es que queremos ejecutar `exec` de manera interactiva `-it` en el contenedor `my-sql-data` el comando `un comando` con la password `-p`.

Debería aparece algo parecido a:

![accediendo-contenedor-mysql-con-pass](accediendo-contenedor-mysql-con-pass.png)

Una vez dentro del contenedor comprobamos las bases de datos existentes dentro del mismo:

```bash
show databases;
```

![databases inside container](container-db-1.png)

Vamos a crear una nueva base de datos, para ello ejecutamos el comando `CREATE DATABASE [nombreDataBase];` Una vez creada volvemos a listar las bases de datos y debería aparece la que acabamos de crear:

![Database list](db-list-with-new-db.png)

Una vez creada la base de datos paramos y eliminamos el contenedor `docker stop [my-volume-name]` y `docker rm [my-volume-name]`.

Confirmamos que el contenedor ya no está corriendo `docker ps` y que tampoco aparece entre el listado de los creados `docker ps -a`.

Volvemos a crear el contenedor mysql con el script anterior indicando el volumen a utilizar

```bash
$ docker run --name mysql-db -e MYSQL_ROOT_PASSWORD=my-secret-pw -d --mount src=db-data,dst=/var/lib/mysql mysql
```

Nos metemos en el contenedor nuevamente:

```bash
$ docker exec -it my-sql-data mysql -p
```

Y verificamos las bases de datos que contiene:

```bash
show databases;
```

![Database list](db-list-with-new-db.png)

Deberíamos ver la bbdd creada anteriormente (cuando creamos el primer contenedor).

#### Creación volúmenes y asociación al contenedor al arrancar el contenedor

Otra forma de crear volúmenes es realizándolo en el momento en el que se lanza el contenedor.

**Ejemplo 2**

Lo primero que vamos a hacer es para y eliminar el contenedor que hemos creado en el ejemplo anterior

```bash
docker stop [nombre-contenedor] && docker rm [nombre-contenedor]
```

Una vez parado eliminamos el volumen creado también en el ejemplo anterior

```bash
docker volume rm data
```

Para asociarle un volumen a un contenedor en el momento de levantarlo debemos pasarle el flag `-v`. El valor que recibe el parámetro es el nombre del volumen y el nombre del directorio en el contenedor separado por dos puntos `docker run -it --name contenedor2 -v vol1:/data ubuntu bash`.

#### Añadir volumen en el Dockerfile

Dentro de las instrucciones que se pueden añadir en el Dockerfile está `VOLUME`. Esta instrucción crea un punto de montaje asociado a un directorio dentro del contenedor. La sintáxis es `VOLUME ["/path-to-volume"]`. Si este path no existe en el contenedor que estás creando, Docker lo hará por ti, en caso contarario simplemente definirá dicho path como volumen.

Si te fijas en la sintáxis, en este caso no se especifica la carpeta del host donde se quiere mapear dicho volumen y no, no puedes lanzar el comando anterior con sólo el `src` y esperar a que Docker mapee el path local con el del contenedor de manera automática...entonces ¿Para qué sirve esta definición en el Dockerfile?

El `Dockerfile` es un manifiesto y como tal debe contener la definición de todo lo que el contenedor va a necesitar, incluido el VOLUMEN (si se quieren persistir datos), el puerto que se expone ... Esto ayudará a su mantenimiento y por su puesto a aquella/s persona/s que se va a encargar de poner en marcha el contenedor. Imaginate el caso anterior, una base de datos, si por alguna razón has de mover el contenedor a otra máquina ¿Cómo podrías saber si el contenedor va a necesitar un volumen y dónde se ha de mapear dicho volumen?

[Aquí](https://stackoverflow.com/questions/40163036/difference-between-volume-declaration-in-dockerfile-and-v-as-docker-run-paramet/#answer-40163757) puedes leer una explicación mejor acerca de qué diferencia el flag `-v` y la definición de `VOLUMEN` en el fichero Dockerfile.

#### Backups

Los volúmenes también puden utilizarse para la realización de backups.

Veamos un ejemplo, para ello vamos a crear un nuevo contenedor al que le vamos a asociar un volumen, pero dicho volumen no estará mapeado a una carpeta del host.

```bash
docker run -d -it -v /dbdata --name dbstore ubuntu /bin/bash
```

Con este comando creamos un nuevo contenedor `dbstore` partiendo de una máquina ubuntu. Le pasamos los parámetros `-d` para lanzarlo en modo `detached` y `-it` junto con `/bin/bash` para que el contenedor permanezca corriendo.

Una vez lanzado el contenedor, comprobamos que sigue corriendo `docker ps`. Deberías ver algo parecido a esto 

![backup-dbstore](backup-dbstore.png)

A accedemos al contenedor `docker exec -it dbstore /bin/bash`

Una vez dentro podemos ver que se ha creado la carpeta `dbstore` 

![dbdata-folder-in-container](dbdata-folder-in-container.png)

Accedemos a ella y creamos un fichero cualquiera, en nuestro caso `foo.txt`

![fichero-dbdata](fichero-dbdata.png)

Con esto hemos preparado el volumen del que queremos hacer el respaldo.

Vamos ahora borrar el contenedor que hemos creado pero a la vez haremos una copia de la carpeta dbdata en el host.

```bash
docker run --rm --volumes-from dbstore -v $(pwd):/dbdata ubuntu tar cvf /backup/backup.tar /dbdata
```

Con el comando anterior lo que estamos haciendo es crear un nuevo contenedor que parte de una imagen de ubuntu. Dicho contendor se borrará una vez no esté en ejecución (opción `--rm`). En este nuevo contenedor le decimos de dónde queremos que coja el volumen del que queremos realizar un backup, en nuestro caso del contenedor `dbstore`. A su vez montamos en el directorio actual `$(pwd)` un volumen local que contendrá un tar del contenido de la carpeta `/dbdata` `ubuntu tar cvf /backup/backup.tar /dbdata`.

Una vez realizado esto vamos acrar un nuevo contenedor al que llamaremos `dbstore2`

```bash
docker run -d -it -v /dbdata --name dbstore2 ubuntu /bin/bash
```

Posteriormente, descomprimimos el contendio del fichero `backup.tar` en el nuevo contenedor `dbstore2`

```bash
docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

Finalmente nos metemos dentro de `dbstore2` y confirmamos que existe un directorio `/dbdata` con un fichero `foo.txt`.

![db-store2-with-backup](db-store2-with-backup.png)

#### Eliminación de volúmenes

Al igual que creamos volúmenes estos puenden ser eliminados. Para ello sólo tendrías que ejecutar el comando `docker volume rm [nombre-volumen]`.

## Networking

Docker premite crear redes que faciliten: 

* La comunicación de los contenedores con el host.
* La comuniciación entre los contenedores.
* La comunicación entre los nodos de un clúster Docker.

Los tipos de redes disponibles en Docker son:

* **host**: Es la red del propio equipo y suele hacer referencia a eth0.
* **bridge**: Red puente que facilita la comunicación entre los contenedores y el host. Por defecto Docker crea la red bridge docker0 y a ella se conectan todos los contenedores.
* **none**: Representa la no conexión a ninguna interfaz de red.
* **overlay**: Es una red que sirve para comunicar diferentes nodos de un clúster. Se necesita un servicio de almacenamiento externo.


### Comandos básicos

El comando principal será `docker network` al que le podemos pasar diferentes opciones:

* **`ls`** para listar las networks que tengamos creadas.
* **`create --driver [DRIVER_NAME] [NETWORK_NAME]`** para crear una nueva red con un driver específico y un nombre de red.
* **`connect|disconnect [NETWORK_NAME] [CONTAINER]`** para conectar o desconectar un contenedor a una red concreta.
* **`inspect [NETWORK_NAME]`** para ver los detalles de una red, como por ejemplo los contenedores conectados a ella.
* **`rm [NETWORK_NAME]`** para borrar una red específica.

### Trabajando con redes

Veamos qué vemos cuando, antes de crear nada, miramos las redes que hay disponibles, para ello lanzamos el comando:

```bash
docker network ls
```

Deberías ver algo parecido a esto:

![default-docker-networks](default-docker-networks.png)

Estas tres 3 las crea Docker por defecto. Y será la red `bridge` la que se utilizará por defecto por todos los contenedores.

Si inspeccionamos la red `bridge` aparecerá algo como esto: 

```bash
[{
    ...
    "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
    ...
     "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            ...
        },
    "labels": {}
}]
...
```

Como hemos dicho anteriormente, cuando creamos un contenedor éste se conectará automáticamente a la red `bridge`. Creemos dos contenodores y veamos si es así.

```bash
docker run -d -it --name contenedor1 alpine /bin/sh
docker run -d -it --name contenedor2 alpine /bin/sh
```

Comprobamos que los contenedores están levantados

![alpine-network-example-1](alpine-network-example-1.png)

Inspeccionamos de nuevo la red `bridge`:

```bash
docker network inspect bridge // bridge puede ser sustituido por el hash
```

Ahora en la propiedad `Containers` deberían aparecernos los dos contenedores que hemos creado:

```bash
[{
    ...,
     "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "17aea0a1ee8a4110ea1e2ee00dbbeb73dc8deb886d7873e3650d32b9823c8bd3": {
                "Name": "contenedor1",
                "EndpointID": "db81ef1748cab8ae165b41b88963508684f9a7927e042285f49620037a2151c6",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "9137b57eeebcffab32a1b67b0f526c94b32faeb598fda7b73d8324ed940ad128": {
                "Name": "contenedor2",
                "EndpointID": "3da2618563bf3a136ef1df081781c9770acbe091692c2d02ab0b70d807c9a8c2",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
    ...
}]
```

Podemos conectar un contenedor a una red al arrancarlo. Vamos primero a crear una red y posteriormente conectar un contenedor a dicha red al arrancarlo.

Primero creamos la red `docker network create myNetwork`. A continuación arrancamos un contenedor y lo conectamos a la red `myNetwork`:

```bash
docker run -d -it --rm --network=myNetwork alpine /bin/sh
```

Confirmamos que el contenedor está corriendo:

```bash
docker ps
// Output
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
a10d6d28ff50        alpine              "/bin/sh"           2 seconds ago       Up 1 second                             elated_shannon
```

Inspeccionamos la red `myNetwork` para confirmar que nuestro contendor se encentra dentro dicha red `docker newtwork inspect myNetwork`:

```bash
[{
   "Name": "myNetwork",
   ...
   "Containers": {
            "a10d6d28ff50822ea309890f9a4702b7fe46d602ceaad458bc7b398001d646e8": {
                "Name": "elated_shannon",
                "EndpointID": "8d444158413b866b5816088e109a0d15fec884377c6c195d3d5a48d09cd90ad3",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}  
}]
```



## Compose
## Docker Machine
## Docker Swarm
