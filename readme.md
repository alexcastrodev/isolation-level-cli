# Isolation level CLI

To fazendo um visualizador de isolamento de transações em SQL.

# Casos de Uso

## Dirty Writes

[Simulação de Dirty Write](sim_dirty_write.md)

# Exemplo de uso

```bash
> p1 BEGIN READ_COMMITTED
p1: BEGIN transaction (isolation=read_committed)
> p2 BEGIN READ_UNCOMMITTED
p2: BEGIN transaction (isolation=read_uncommitted)
> p1 INSERT price_cents=1000
p1: queued INSERT 1,1000
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
| 1  | 1000        |
+----+-------------+
> p2 select
p2: SELECT (read_uncommitted)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 1000        |
+----+-------------+
> p3 BEGIN READ_COMMITED
p3: Unknown isolation level READ_COMMITED
> p3 BEGIN READ_COMMITTED
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
+----+-------------+
> p3 BEGIN READ_UNCOMMITTED
p3: BEGIN transaction (isolation=read_uncommitted)
> p3 select
p3: SELECT (read_uncommitted)
+----------------+------------+---------+
| Column Name    | Type       | Default |
+----------------+------------+---------+
| id             | integer    |         |
| price_cents    | integer    | 0       |
+----------------+------------+---------+
+----+-------------+
| ID | PRICE CENTS |
+----+-------------+
| 1  | 1000        |
+----+-------------+
```


# Referências

https://dzone.com/articles/sql-phenomena-for-developers

https://pmg.csail.mit.edu/papers/icde00.pdf
