## Cluster de 3 Nós

Criar um ambiente sandbox no CockroachDB é tão simples:

IU do console do CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

* Implante o primeiro node em segundo plano e acesse o console de interface do usuário do CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Atribuímos `--http-addr=0.0.0.0:8080` neste nó para expô-lo a um IP público.

* Implante o segundo node em segundo plano e acesse o console de interface do usuário do CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach2 --listen-addr=localhost:26258 --http-addr=0.0.0.0:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Atribuímos `--http-addr=0.0.0.0:8081` neste nó para expô-lo a um IP público.


* Implante o terceiro node em segundo plano e acesse o console de interface do usuário do CockroachDB

```
cockroach start --insecure --store=cockroach-data/cockroach3 --listen-addr=localhost:26259 --http-addr=0.0.0.0:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

Atribuímos `--http-addr=0.0.0.0:8082` neste nó para expô-lo a um IP público.


* Inicializar o cluster

Até agora, cada node está em execução, mas será exibido como inativo até que o cluster seja inicializado com o seguinte comando:

```
cockroach init --insecure --host=localhost:26257
```{{exec}}

A inicialização do cluster pode ser feita em qualquer porta SQL do CockroachDB (conforme indicado neste exemplo, pode ser `26257, 26258 ou 26259`).

O CockroachDB elimina a necessidade de um único mestre ou de um único node de gravação (write). Todos os nós são pontos de entrada para leituras e gravações (`read & write`).



* Finalmente, acesse o console SQL do CockroachDB para executar alguns comandos através de qualquer porta SQL: `26257, 26258 ou 26259`

```
cockroach sql --insecure --host=localhost:26257
```{{exec}}

```
SHOW DATABASES;
```{{exec}}

Como você pode ver, o banco de dados `movr` que tínhamos durante a primeira demonstração não existe. Podemos prosseguir para criá-lo por meio de uma carga de trabalho e testar a capacidade do cluster. Vamos sair do console SQL do CockroachDB por enquanto.

```
\q
```{{exec}}

* Opcional: Embora não seja abordado em profundidade neste workshop, a maneira mais prática de acessar o cluster seria por meio de balanceadores de carga. Veja um exemplo com [`cockroach gen haproxy`](https://www.cockroachlabs.com/docs/stable/cockroach-gen.html), que lê a configuração do seu cluster e gera automaticamente o `haproxy.cfg` com:

```
cockroach gen haproxy --insecure --host=localhost --port=26257
```{{exec}}

* Instale e configure o HAProxy para executar em segundo plano

```
apt install -y haproxy
```{{exec}}

```
haproxy -f haproxy.cfg -D
```{{exec}}

* Como estamos executando o haproxy e todas as 3 instâncias do CockroachDB na mesma VM, o balanceador de carga não pode ser vinculado a uma porta já em uso pelo CockroachDB. Este cenário funciona quando o CockroachDB está sendo executado em instâncias de VM dedicadas ou pods/contêineres do Kubernetes.

`[ALERT] 155/142030 (23803) : Starting proxy psql: cannot bind socket [0.0.0.0:26257]`

Por enquanto, vamos ignorar a abordagem do balanceador de carga e passar para o teste de cargas de trabalho transacionais, além de como testar com falha e remover nós.