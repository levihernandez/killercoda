## Resumo

Existem diferentes maneiras de implantar o CockroachDB em instâncias auto-hospedadas, como máquinas virtuais, contêineres Kubernetes, modos seguro (SSL) ou inseguro. Todos devem ser instalados com uma abordagem simples e fácil de manter. Se você está procurando uma abordagem em que não precise manter o cluster, o Cockroach Labs oferece [Cockroach Cloud](https://cockroachlabs.cloud/) como um serviço dedicado ou serverless.

* Aprendemos como implementar o cluster de 3 nós do CockroachDB
* Executamos uma carga de trabalho, workload, que simula transações
* Monitoramos a atividade do cluster enquanto um nó estava inativo e depois o restauramos
* Vimos o valor de usar o banco de dados distribuído e como ele ajuda a reduzir o tempo de inatividade em workloads críticas