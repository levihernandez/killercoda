### Descarga de Paquete de CockroachDB

Objetivos:
* Descarga e instalación del paquete
* Ejecución del demo
* Ejecución de un nodo
* Accesar CockroachDB Web Console


1. Desplegar el primer nodo es fácil y rápido:

    a) Cambia directorio a

    `cd /home/ubuntu`{{exec}}

    b) Descargar el binario de CockroachDB para Linux, y extraer el paquete binario:

    `curl https://binaries.cockroachdb.com/cockroach-v23.1.1.linux-amd64.tgz | tar -xz`{{exec}}

    c) Copia el paquete al directorio binario del sistema:

    `cp -i cockroach-v23.1.1.linux-amd64/cockroach /usr/local/bin/`{{exec}}


3. Verificar la instalación local e inicializar el ambiente demo:

    a) Localiza el paquete con:

    `which cockroach`{{exec}}

    que debe estar localizado en:

    ```
    /usr/local/bin/cockroach
    ```

    b) Ejecutar el paquete y accesar la terminal SQL con el ejemplo **demo**:

    `cockroach demo`{{exec}}

    c) Ejecutar comandos SQL en la base de datos `movr` y seleccionar 5 registros:

    `SHOW DATABASES;`{{exec}}

    `USE movr;`{{exec}}

    `SHOW TABLES;`{{exec}}

    `SELECT * FROM rides LIMIT 5`{{exec}}

    d) Salir del Shell de SQL:

    `\q`{{exec}}

4. Desplegar un nodo y accesar la interfaz Web UI de la consola

    a) Despliegue el nodo:

    `cockroach start-single-node --insecure --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --background`{{exec}}

    CockroachDB, en este caso, es ejecutado de forma insegura. El puerto de la base de datos es `26257`; el puerto Web UI de la consola es `8080` y la IP se asigna abierta `0.0.0.0`.
    En estos casos, la IP se puede asignar a una IP especifica.

    En este esenario no existe la base de datos `movr` que vimos anteriormente con `demo`.

    b) Conectarse a la base de datos y ejecutar comandos SQL en la base de datos `movr` y seleccionar 5 registros:

    `cockroach sql --insecure --host=localhost:26257`{{exec}}

    `SHOW DATABASES;`{{exec}}

    c) Accesa la interface Web UI en [CockroachDB UI]({{TRAFFIC_HOST1_8080}})

