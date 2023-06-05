## Error de Nodo y Carga de Trabajo de CockroachDB

Consola UI de CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

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

* Revise la actividad de latencia cuando el workload esta en ejecución, abra la liga de la consola en un nuevo tab para ver la actividad: [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

### Pruebe la Resiliencia del Clúster

* Desactive el nodo con el puerto SQL 26257, desde esta perspectiva solo no hay un punto de entrada disponible, pero el clúster continúa ejecutándose y se puede acceder a través de los puertos `26258 y 26259`. Ejecutemos la carga de trabajo en el puerto SQL 26258 y verifiquemos la consola de la interfaz de usuario para revisar la actividad en el clúster; notará que un nodo no está disponible.

> Elimine el proceso unix para el primer nodo de CockroachDB

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

* Ver el resultado de la falla del nodo en [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}})

* Ejecute una nueva carga de trabajo en el siguiente puerto SQL disponible y observe que CockroachDB continúa ejecutándose:

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26258?sslmode=disable'
```{{exec}}

* Ver la actividad del clúster durante la ejecución de la carga de trabajo en [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

Tenga en cuenta que las declaraciones de SQL proporcionan actividad para los dos nodos restantes y también proporcionan los percentiles de latencia del servicio mientras el nodo está inactivo.


* Restaure el nodo y vea la actividad en la consola de la interfaz de usuario: [Puerto de interfaz de usuario de CockroachDB 8081]({{TRAFFIC_HOST1_8081}}) o [Puerto de interfaz de usuario de CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

**Observación:** el storage del nodo no es afectado dado que se restaura dentro de una ventana de tiempo, usualmente 5 min. De otra forma, si pasa mas tiempo el clúster marcara al nodo como `dead`. Al restaurarse un nodo en status `dead` el clúster reintegrara los mas recientes registros para actualizar al nodo.

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

* Finalmente, ejecutemos algunos comandos SQL para ver la configuración del clúster y los rangos KV de la base de datos.

```
cockroach sql --insecure
```{{exec}}

```
SHOW RANGES FROM DATABASE movr;
```{{exec}}

* Obtenga la versión de CockroachDB instalada en el clúster, la configuración del clúster se puede extraer de dos maneras:

```
SELECT variable, value FROM [SHOW ALL CLUSTER SETTINGS] WHERE variable IN ('version');
```{{exec}}

```
SHOW CLUSTER SETTING version;
```{{exec}}

A continuación, procederemos a eliminar y desmantelar un nodo. Nos quedaremos con 2 nodos pero agregaremos uno mas para tener un clúster resistente, resiliente, que escala y provee una experiencia 'always-on' para los aplicativos de misión crítica.