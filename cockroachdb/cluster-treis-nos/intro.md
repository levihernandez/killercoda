## Início rápido do CockroachDB: auto-hospedado inseguro

O workshop atual ajudará você a implementar um cluster CockroachDB inseguro (sem SSL) passo a passo.

Metas:

* Baixe o binário mais recente ou uma versão específica. Veja os lançamentos listados no [CockroachDB Github](https://github.com/cockroachdb/cockroach/tags)
* Implante 3 nós, um por um, no mesmo host. A melhor prática é implantar uma instância CockroachDB por VM.
     * Monitore a atividade do cluster por meio de qualquer uma das portas da interface do usuário:
         * [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
         * [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
         * [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})
* Execute a pequena carga de trabalho, workload, `movr` e consultas SQL que listam a atividade no cluster
* Remova um node do cluster
* Resumo