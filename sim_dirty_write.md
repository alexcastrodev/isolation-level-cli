# Simulação de Dirty Write

Passo 1: João tenta pagar as contas de Maria. Primeiro, sua transação executa um SELECT para consultar o valor devido. Maria tenta pagar essas contas ao mesmo tempo. Portanto, ela executa exatamente a mesma consulta e obtém o mesmo resultado que João ($345).

Passo 2: A transação de João tenta pagar todo o valor devido. Consequentemente, o valor a pagar é atualizado para $0.

Passo 3: A transação de Maria não está ciente dessa atualização e tenta, com sucesso, pagar metade do valor devido (sua transação é confirmada). O UPDATE executado define o valor a pagar como $173.

Passo 4: Infelizmente, a transação de João não consegue ser confirmada e precisa ser revertida. Portanto, o valor a pagar é restaurado para $345. Isso significa que Maria acabou de perder $172.

Simulação:

Primeiro, vamos criar uma tabela de preços:

```bash
> p1 BEGIN READ_COMMITTED
p1: BEGIN transaction (isolation=read_committed)
> p1 INSERT price_cents=345
p1: queued INSERT 1,345
> p1 COMMIT
p1: COMMIT successful
> p1 select
p1: SELECT (read_committed)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 345         |
+----+-------------+
```

Passo 1: João (p1) e Maria (p2) leem o mesmo valor

```bash
> p1 BEGIN READ_COMMITTED
p1: BEGIN transaction (isolation=read_committed)
> p2 BEGIN READ_COMMITTED
p2: BEGIN transaction (isolation=read_committed)
> p1 select
p1: SELECT (read_committed)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 345         |
+----+-------------+
> p2 select
p2: SELECT (read_committed)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 345         |
+----+-------------+
```

Passo 2: João paga tudo

```bash
> p1 UPDATE id=1 price_cents=0
p1: queued UPDATE id=1 price_cents=0
> p1 COMMIT
p1: COMMIT successful
```

Passo 3: Maria, sem saber do pagamento de João, se ferra

```bash
> p2 UPDATE id=1 price_cents=173
p2: queued UPDATE id=1 price_cents=173
> p2 COMMIT
p2: COMMIT successful
```

Passo 4: resultado? perdeu dindin mané

```bash
p3: BEGIN transaction (isolation=read_committed)
> p3 select
p3: SELECT (read_committed)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 173         |
+----+-------------+
````
