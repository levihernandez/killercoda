## Resumen

Hay diferentes formas de implementar CockroachDB en instancias autohospedadas, como máquinas virtuales, contenedores de Kubernetes, modos seguros (SSL) o inseguros. Todos están destinados a instalarse como un enfoque sencillo y facilitar su mantenimiento. Si está buscando un enfoque en el que no necesite mantener el clúster, Cockroach Labs ofrece [Cockroach Cloud](https://cockroachlabs.cloud/) como un servicio dedicado o sin servidor.

* Aprendimos a implementar el clúster de 3 nodos de CockroachDB
* Ejecutamos una carga de trabajo que simula transacciones
* Supervisamos la actividad del clúster mientras un nodo estaba inactivo y luego lo restauramos
* Vimos el valor de usar la base de datos distribuida y cómo ayuda a mitigar los tiempos de inactividad en cargas de trabajo críticas