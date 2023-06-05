## Clúster de 3 Nodos

Crear un entorno sandbox en CockroachDB es tan sencillo:

Consola UI de CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

* Implemente el primer nodo en segundo plano (background) y acceda a la consola de interfaz de usuario de CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Hemos asignado `--http-addr=0.0.0.0:8080` en este nodo para exponerlo a una IP pública.

* Implemente el segundo nodo en segundo plano y acceda a la consola de interfaz de usuario de CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach2 --listen-addr=localhost:26258 --http-addr=0.0.0.0:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Hemos asignado `--http-addr=0.0.0.0:8081` en este nodo para exponerlo a una IP pública.


* Implemente el tercer nodo en segundo plano y acceda a la consola de interfaz de usuario de CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach3 --listen-addr=localhost:26259 --http-addr=0.0.0.0:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Hemos asignado `--http-addr=0.0.0.0:8082` en este nodo para exponerlo a una IP pública.


* Inicializar el clúster

Hasta ahora, cada nodo se está ejecutando, pero se mostrará como no funcional hasta que el clúster se inicialice con el siguiente comando:

```
cockroach init --insecure --host=localhost:26257
```{{exec}}

La inicialización del clúster se puede realizar en cualquier puerto SQL de CockroachDB (como se indica en este ejemplo, podría ser `26257, 26258 o 26259`).

CockroachDB elimina la necesidad de un solo Master o un solo nodo de writer. Todos los nodos son puntos de entrada para lecturas y escrituras (`read & write`).



* Por último, acceda la consola SQL de CockroachDB para ejecutar algunos comandos a través de cualquier puerto SQL: `26257, 26258 o 26259`

```
cockroach sql --insecure --host=localhost:26257
```{{exec}}

```
SHOW DATABASES;
```{{exec}}

Como puede ver, la base de datos `movr` que tuvimos durante la primera demostración no existe. Podemos proceder a crearla a través de una carga de trabajo y probar la capacidad del clúster. Salgamos de la consola SQL de CockroachDB por ahora.
```
\q
```{{exec}}

* Opcional: aunque no se cubre en profundidad en este taller, la forma más práctica de acceder al clúster sería a través de balanceadores de carga. Vea un ejemplo con [`cockroach gen haproxy`](https://www.cockroachlabs.com/docs/stable/cockroach-gen.html), que lee la configuración de su clúster y genera automáticamente el haproxy.cfg con:

```
cockroach gen haproxy --insecure --host=localhost --port=26257
```{{exec}}

* Instale y configure HAProxy para que se ejecute en el fondo

```
apt install -y haproxy
```{{exec}}

```
haproxy -f haproxy.cfg -D
```{{exec}}

* Debido a que estamos ejecutando haproxy y las 3 instancias de CockroachDB en la misma VM, el balanceador de carga no puede vincularse a un puerto ya en uso por CockroachDB. Este escenario funciona cuando CockroachDB se ejecuta en instancias de VM dedicadas o pods/contenedores de Kubernetes.

`[ALERT] 155/142030 (23803) : Starting proxy psql: cannot bind socket [0.0.0.0:26257]`

Por ahora, omitiremos el enfoque del balanceador de carga y pasaremos a probar las cargas de trabajo transaccionales, así como también cómo realizar la prueba de falla y la eliminación de nodos.