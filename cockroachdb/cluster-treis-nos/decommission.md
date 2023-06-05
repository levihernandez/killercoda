## Remover um Node

Consulte as práticas recomendadas para desligamento de nó na documentação [Remoção de node](https://www.cockroachlabs.com/docs/stable/node-shutdown.html)

IU do console do CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})


Há duas etapas para remover um node do cluster:

* Dreno de node
* Descomissionamento de nodes

### Dreno del Nodo

É importante drenar um nó antes de removê-lo completamente para interromper as conexões do balanceador de carga. Caso um nó falhe, o lado do aplicativo pode contar com o método de nova tentativa de conexão do aplicativo (ou retry) e/ou do balanceador de carga.

**OBSERVAÇÃO**: Estamos abordando os destaques para desativar um nde com um cluster de 3 nodes. Na prática, a arquitetura deve ter uma maneira de adicionar novos nós quando um node falha ou 5 nodes no mínimo para contabilizar o tempo para restaurar um node, arquiteturas e casos de uso podem variar de acordo com a necessidade.

```
SET CLUSTER SETTING server.shutdown.drain_wait = '8s';
```{{exec}}

```
SET CLUSTER SETTING server.time_until_store_dead = '15m0s';
```{{exec}}

```
\q
```{{exec}}



* As melhores práticas ditam que nosso cluster deve ter um número mínimo de nós para se manter íntegro. Adicionaremos um quarto nó para garantir que possamos retirar o primeiro nó rapidamente.

Isso garante que os dados do primeiro nó sejam transferidos para o novo nó. Observe como os dados se movem para o novo nó no gráfico "Réplicas por nó" para [Réplicas da porta de interface do usuário 8083 do CockroachDB no cluster]({{TRAFFIC_HOST1_8083}}/#/metrics/replication/cluster).

```
cockroach start --insecure --store=cockroach-data/cockroach4 --listen-addr=localhost:26260 --http-addr=0.0.0.0:8083 --join=localhost:26258,localhost:26259,localhost:26260 --background
```{{exec}}

Atribuímos `--http-addr=0.0.0.0:8083` neste nó para expô-lo a um IP público.

Para acessar o console do terceiro node, abra a url [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}) após inicializar o cluster.


* Verifique o andamento do desligamento do node no log com o comando `cockroach node`

```
cockroach node drain 1 --host=localhost:26257 --drain-wait=15m --insecure
```{{exec}}

* O log fornece a sequência de eventos durante a drenagem do node:

```
grep 'drain' cockroach-data/cockroach1/logs/cockroach.log
```{{exec}}



* Uma vez que o nó que usa a porta `26257` está sendo esvaziado, vamos usar a próxima porta disponível `26258` para verificar o status do nó que estamos desativando. A saída SQL na coluna `is_draining` registrou `id=1` como `true` para drenar o nó.

```
cockroach node status --decommission --insecure --host=localhost:26258
```{{exec}}

* Por fim, vamos eliminar o node desativando-o e observar as réplicas em todos os nodes em [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}/#/overview/list). Seus dados acabaram de ser rebalanceados com `Zero Downtime`.

```
cockroach node decommission 1 --insecure --host=localhost:26258
```{{exec}}


> Mate o processo unix para o primeiro node CockroachDB

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

Parabéns! Você removeu e retirou um node do cluster CockroachDB.