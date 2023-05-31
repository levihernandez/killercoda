### Descarga de Paquete de CockroachDB

1. Desplegar el primer nodo es fácil y rápido:

    a) Cambia directorio a

    `cd /home/ubuntu`{{exec}}

    b) Descargar el binario de CockroachDB para Linux, y extraer el paquete binario:

    `curl https://binaries.cockroachdb.com/cockroach-v23.1.1.linux-amd64.tgz | tar -xz`{{exec}}

    c) Copia el paquete a `/usr/local/bin/cockroach`:

    `cp -i cockroach-v23.1.1.linux-amd64/cockroach /usr/local/bin/`{{exec}}


3. Verificar la instalación local

    a) Localiza el paquete con:

    `which cockroach`{{exec}}

    que debe estar localizado en:

    ```
    /usr/local/bin/cockroach
    ```

    b) Ejecutar el paquete y accesar la terminal SQL con el ejemplo `demo`:

    `cockroach demo`{{exec}}

    c) Ejecutar comandos SQL en la base de datos `default`:

    `SHOW DATABASES;`

    `USE defaultdb;`

    `SHOW TABLES;`