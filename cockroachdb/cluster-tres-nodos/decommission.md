## Retirar un nodo

Consulte las mejores prácticas para el desmantelamiento de los nodos en la documentacion [Node Removal](https://www.cockroachlabs.com/docs/stable/node-shutdown.html)


Consola UI de CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})


Hay dos pasos para eliminar un nodo del clúster:

* Drenaje del nodo
* Desmantelamiento de nodos

### Drenaje del Nodo

Es importante drenar un nodo antes de eliminarlo por completo para desmantelar las conexiones del balanceador de carga. En caso de que falle un nodo, el lado de la aplicación puede confiar en el método de reintento de conexión del aplicativo y/o en el balanceador de carga.

**NOTA**: Estamos cubriendo los aspectos más destacados para retirar un nodo con un clúster de 3 nodos. En la práctica, la arquitectura debe tener una forma de agregar nuevos nodos cuando falla un nodo o 5 nodos como mínimo para tener en cuenta el tiempo para restaurar un nodo, las arquitecturas y los casos de uso pueden variar según la necesidad.

```
SET CLUSTER SETTING server.shutdown.drain_wait = '8s';
```{{exec}}

```
SET CLUSTER SETTING server.time_until_store_dead = '15m0s';
```{{exec}}

```
\q
```{{exec}}



* Las mejores prácticas dictan que nuestro clúster debe tener una cantidad mínima de nodos para mantenerse en buen estado. Agregaremos un cuarto nodo para garantizar que podamos retirar el primer nodo rápidamente.

Esto garantiza que los datos del primer nodo se transfieran al nuevo nodo. Observe cómo los datos se mueven al nuevo nodo en el gráfico "Replicas per Node" de [CockroachDB UI Port 8083 Replicas en el Cluster]({{TRAFFIC_HOST1_8083}}/#/metrics/replication/cluster).

```
cockroach start --insecure --store=cockroach-data/cockroach4 --listen-addr=localhost:26260 --http-addr=0.0.0.0:8083 --join=localhost:26258,localhost:26259,localhost:26260 --background
```{{exec}}

Hemos asignado `--http-addr=0.0.0.0:8083` en este nodo para exponerlo a una IP pública.

Para acceder a la consola del tercer nodo, abra la url [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}) después de inicializar el clúster.



* Verifique el progreso del apagado del nodo en el registro con el commando `cockroach node`

```
cockroach node drain 1 --host=localhost:26257 --drain-wait=15m --insecure
```{{exec}}

* El registro `log` proporciona la secuencia de eventos durante el drenaje del nodo:

```
grep 'drain' cockroach-data/cockroach1/logs/cockroach.log
```{{exec}}



* Dado que el nodo que usa el puerto `26257` se está vaciando, usemos el siguiente puerto disponible `26258` para verificar el estado del nodo que estamos retirando. El resultado de SQL en la columna `is_draining` ha registrado `id=1` como `true` para drenar el nodo.

```
cockroach node status --decommission --insecure --host=localhost:26258
```{{exec}}

* Por último, eliminemos el nodo desactivándolo con `decommission` y observemos las replicas en todos los nodos en [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}/#/overview/list). Sus datos acaban de reequilibrarse con `Cero Downtime`.

```
cockroach node decommission 1 --insecure --host=localhost:26258
```{{exec}}


> Elimine el proceso unix para el primer nodo de CockroachDB

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

¡Felicidades! Ha eliminado y retirado un nodo del clúster de CockroachDB.