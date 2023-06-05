## Descargar, Instalar, y Probar CockroachDB

Cockroach Labs ha diseñado el proceso de instalación como un enfoque sencillo para aumentar la productividad.

* Primero establezca una versión de CockroachDB, elija una de la lista que se encuentra en [Instalar CockroachDB en Linux](https://www.cockroachlabs.com/docs/v23.1/install-cockroachdb-linux#install-binary)

```
export crdb_release="v23.1.2"; export architecture=$(dpkg --print-architecture)
```{{exec}}

* Descargue, descomprima e instale el binario CockroachDB

```
cd /home/ubuntu; curl https://binaries.cockroachdb.com/cockroach-${crdb_release}.linux-${architecture}.tgz | tar -xz
```{{exec}}

* Para instalar CockroachDB, Copie el binario al sistema:

```
cp -i cockroach-${crdb_release}.linux-${architecture}/cockroach /usr/local/bin/; which cockroach
```{{exec}}

* Hasta ahora hemos instalado el binario, en este punto podemos implementar la primera instancia de demostración de CockroachDB y probar la base de datos `movr` y ejecutar algunas consultas.

```
cockroach demo
```{{exec}}

* Ejecute un par de consultas en la base de datos `movr` de muestra

```
SHOW DATABASES;
```{{exec}}

```
USE movr;
```{{exec}}

```
SHOW TABLES;
```{{exec}}

```
SELECT * FROM users LIMIT 5;
```{{exec}}

* Salga de la consola SQL de CockroachDB

```
\q
```{{exec}}

A continuación, implementaremos los tres nodos y accederemos a la consola de interfaz de usuario de CockroachDB.