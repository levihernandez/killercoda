## Error de Nodo y Carga de Trabajo de CockroachDB

* Inicialice la carga de trabajo de CockroachDB llamada `movr`

```
cockroach workload init movr 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Acceda a la consola SQL para ver la base de datos `movr` y ejecute una consulta

```
cockroach sql --insecure
```{{exec}}

```
SHOW TABLES FROM movr;
```{{exec}}

```
SELECT * FROM movr.users WHERE city='new york';
```{{exec}}

* Exit the CockroachDB SQL console

```
\q
```{{exec}}

* Ejecute una carga de trabajo con una duración de `1m` contra el clúster a través del primer nodo con el puerto SQL `26257`

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Compruebe la actividad en [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster) para determinar la latencia del servicio con 3 nodos.

### Test Resiliency of the CockroachDB Cluster

* Desactive el nodo con el puerto SQL 26257, desde esta perspectiva solo no hay un punto de entrada disponible, pero el clúster continúa ejecutándose y se puede acceder a través de los puertos `26258 y 26259`. Ejecutemos la carga de trabajo en el puerto SQL 26258 y verifiquemos la consola de la interfaz de usuario para revisar la actividad en el clúster; notará que un nodo no está disponible.

> Elimine la identificación del proceso para el primer nodo de CockroachDB

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

* Ver el resultado del bloqueo del nodo en [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}})

* Ejecute una nueva carga de trabajo en el siguiente puerto SQL disponible y observe que CockroachDB continúa ejecutándose:

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26258?sslmode=disable'
```{{exec}}

* Ver la actividad del clúster durante la ejecución de la carga de trabajo en [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

Tenga en cuenta que las declaraciones de SQL proporcionan actividad para los dos nodos restantes y también proporcionan los percentiles de latencia del servicio mientras el nodo está inactivo.


* Haga una copia de seguridad del nodo y vea la actividad en la consola de la interfaz de usuario: [Puerto de interfaz de usuario de CockroachDB 8081]({{TRAFFIC_HOST1_8081}}) o [Puerto de interfaz de usuario de CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

* Finalmente, ejecutemos algunos comandos SQL para ver la configuración del clúster y los rangos de la base de datos.

```
cockroach sql --insecure
```{{exec}}

```
SHOW RANGES FROM DATABASE movr;
```{{exec}}

* Get the CockroachDB version installed in the cluster, cluster settings can be extracted in two ways:

```
SELECT variable, value FROM [SHOW ALL CLUSTER SETTINGS] WHERE variable IN ('version');
```{{exec}}

```
SHOW CLUSTER SETTING version;
```{{exec}}

A continuación, procederemos a eliminar y desmantelar un nodo. Nos quedaremos con 2 nodos pero agregaremos uno nuevo, que siguen siendo completamente funcionales. Para conocer las mejores prácticas, asegúrese de que su clúster tenga los recursos adecuados para ser resistente, mínimo 3 nodos.