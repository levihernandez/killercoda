## Inicio Rápido de CockroachDB: Self-Hosted Insecure

El taller actual lo ayudará a implementar un clúster CockroachDB inseguro (sin SSL) paso a paso.

Objetivos:

* Descargue el binario mas reciente o una versión específica. Consulte los lanzamientos enumerados en [CockroachDB Github](https://github.com/cockroachdb/cockroach/tags)
* Implemente 3 nodos, uno por uno, en el mismo host. Las mejores prácticas son implementar una instancia de CockroachDB por VM.
     * Supervise la actividad del clúster a través de cualquiera de los puertos de interfaz de usuario:
         * [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
         * [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
         * [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})
* Ejecute la pequeña carga de trabajo `movr` y las consultas SQL que enumeran la actividad en el clúster
* Eliminar un nodo del clúster
* Resumen