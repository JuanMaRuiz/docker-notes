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

Veamos qué vemos cuando, antes de crear nada miramos las redes disponibles, vamos a lanzar el comando:

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

[ 👉 Siguente - Docker Compose](docker-compose.md)