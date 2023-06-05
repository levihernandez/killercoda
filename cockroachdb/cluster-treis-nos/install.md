## Baixe, Instale e Experimente o CockroachDB

O Cockroach Labs projetou o processo de instalação como uma abordagem simples para aumentar a produtividade.

* Primeiro defina uma versão do CockroachDB, escolha uma da lista em [Instalar CockroachDB no Linux](https://www.cockroachlabs.com/docs/v23.1/install-cockroachdb-linux#install-binary)

```
export crdb_release="v23.1.2"; export architecture=$(dpkg --print-architecture)
```{{exec}}

* Baixe, descompacte e instale o binário CockroachDB

```
cd /home/ubuntu; curl https://binaries.cockroachdb.com/cockroach-${crdb_release}.linux-${architecture}.tgz | tar -xz
```{{exec}}

* Para instalar o CockroachDB, copie o binário para o sistema:

```
cp -i cockroach-${crdb_release}.linux-${architecture}/cockroach /usr/local/bin/; which cockroach
```{{exec}}

* Até agora nós instalamos o binário, neste ponto podemos implantar a primeira instância de demonstração do CockroachDB e testar o banco de dados `movr` e executar algumas consultas.

```
cockroach demo
```{{exec}}

* Execute algumas consultas no banco de dados `movr` de exemplo

```
SHOW DATABASES;
```{{exec}}

```
USE movr;
```{{exec}}

```
SHOW TABLES;
```{{exec}}

```
SELECT * FROM users LIMIT 5;
```{{exec}}

* Sair do console SQL do CockroachDB

```
\q
```{{exec}}

Em seguida, implantaremos todos os três nodes e acessaremos o console de IU do CockroachDB.