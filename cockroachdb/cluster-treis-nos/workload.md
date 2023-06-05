## Erro de Node e Workload do CockroachDB

IU do console do CockroachDB:
* [CockroachDB 8080]({{TRAFFIC_HOST1_8080}})
* [CockroachDB 8081]({{TRAFFIC_HOST1_8081}})
* [CockroachDB 8082]({{TRAFFIC_HOST1_8082}})

* Inicialize a carga de trabalho CockroachDB chamada `movr`

```
cockroach workload init movr 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Acesse o console SQL para visualizar o banco de dados `movr` e executar uma consulta

```
cockroach sql --insecure
```{{exec}}

```
SHOW TABLES FROM movr;
```{{exec}}

```
SELECT * FROM movr.users WHERE city='new york';
```{{exec}}

* Saia do console SQL do CockroachDB

```
\q
```{{exec}}

* Execute uma carga de trabalho com duração de `1m` contra o cluster através do primeiro node com a porta SQL `26257`

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Verifique a atividade de latência quando a carga de trabalho estiver em execução, abra o link do console em uma nova guia para ver a atividade: [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

### Testar a Resiliência do Cluster

* Desative o nó com a porta SQL 26257, nesta perspectiva em uma arquitetura monolítica não há ponto de entrada disponível. A vantagem do CockroachDB é que ele continua rodando e pode ser acessado através de outras portas como `26258 e 26259`. Vamos executar a carga de trabalho na porta SQL 26258 e verificar o console de interface do usuário para revisar a atividade no cluster; você notará que um nó não está disponível.

> Mate o processo unix para o primeiro nó CockroachDB

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

* Veja a saída de falha do node em [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}})

* Execute uma nova workload na próxima porta SQL disponível e observe que o CockroachDB continua em execução:

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26258?sslmode=disable'
```{{exec}}


* Veja a atividade do cluster durante a execução da carga de trabalho em [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

Observe que as instruções SQL fornecem atividade para os dois nodes restantes e também fornecem percentis de latência para o serviço enquanto o node está parado.


* Restaure o nó e visualize a atividade no console da IU: [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}) ou [CockroachDB UI Port 8082]({{TRAFFIC_HOST1_8082} })

**Observação:** o armazenamento do node não é afetado, pois é restaurado dentro de uma janela de tempo, geralmente 5 min. Caso contrário, se passar mais tempo, o cluster marcará o node como `dead`. Ao restaurar um nodo em estado `dead`, o cluster irá reintegrar os registros mais recentes para atualizar o nodo.


```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

* Por fim, vamos executar alguns comandos SQL para ver a configuração do cluster e os ranges KV do banco de dados.

```
cockroach sql --insecure
```{{exec}}

```
SHOW RANGES FROM DATABASE movr;
```{{exec}}

* Obtenha a versão do CockroachDB instalada no cluster, a configuração do cluster pode ser puxada de duas maneiras:

```
SELECT variable, value FROM [SHOW ALL CLUSTER SETTINGS] WHERE variable IN ('version');
```{{exec}}

```
SHOW CLUSTER SETTING version;
```{{exec}}

Em seguida, procederemos à remoção e descomissionamento de um node. Ficaremos com 2 nós, mas adicionaremos mais um para ter um cluster forte e resiliente que dimensiona e fornece uma experiência 'always-on' para aplicativos de missão crítica.