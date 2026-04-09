# Relacionamentos e Junções

_Inner Join_
_Left Join_
_Right Join_
_Alter Join_

## INNER JOIN

- INNER JOIN é o mais utilizado
- permite conectar 2 ou mais tabelas em que se deseja apenas as linhas que tenham correspondência entre si
- Juntar as informações em tabelas diferentes
- relacionamento entre as chaves primárias e estrangeiras
- SELECT normal busca

Sintaxe:

```sql
SELECT a.*, b.*
FROM tabela_a AS a
INNER JOIN tabela_b AS b
ON a.chave = b.chave;
```

- uso dos _aliases_ para simplificar a escrita e leitura quando as tabelas tem nomes longos
- `FROM customers AS c`; o 'c' é um _alias_ para a tabela _customers_

```sql
SELECT p.product_id, p.product_name, c.category_name
FROM products AS p -- tabela da esquerda
INNER JOIN categories AS c -- tabela da direita
  ON p.category_id = c.category_id -- essa é a regra para 'juntar'
ORDER BY p.product_id;

SELECT p.product_id AS pid, p.product_name AS nome_produto, p.price AS preco, c.category_id, c.category_name
FROM products AS p
INNER JOIN categories AS c
  ON p.category_id = c.category_id
WHERE p.price > 100
ORDER BY pid;
```

## INNER JOIN com múltiplas colunas (junção múltipla)

- Adicionar outra 'regra' no INNER JOIN, ou seja, outras colunas
- Tudo o que estiver no INNER JOIN, tudo o que existe na tabela_a tem que existir na tabela_b
- Sintaxe

```sql
SELECT *
FROM tabela_a AS a
INNER JOIN tabela_b AS b
  ON a.id = b.id
  AND a.data = b.data; -- neste contexto, ambas as condições tem que ser verdadeiras
```

- Exemplo:

```sql
SELECT
  o.order_id,
  o.customer_id,
  c.first_name || ' ' || c.last_name AS cliente,
  o.order_date,
  o.status,
  o.total_amount
FROM orders AS o
INNER JOIN customers AS c
  ON o.customer_id = c.customer_id -- vinculada ao pedido - cliente
  AND o.order_date >= c.created_at -- garante que o pedido veio após a criação do cliente
ORDER BY o.order_date DESC
LIMIT 20;
```

## INNER JOIN com três ou mais tabelas

- utilização comum de uso com três ou mais tabelas
- relatórios, BI, ...
- ordenação do `INNER JOIN` => tabela que restringe mais os registros de retorno colocar primeiro na lista
- tomar cuidado sempre com os tipos de dados das regras
- cada `INNER JOIN` vai filtrando os resultados para o próximo `INNER JOIM`

- Sintaxe:

```sql
SELECT *
FROM pedidos AS p
INNER JOIN clientes AS c ON p.cliente_id = c.id -- nesse cenário a tabela clientes vai ser mais restritiva
INNER JOIN produtos AS p ON p.produto_id = pr.id
INNER JOIN pagamentos AS pg ON p.id = pg.pedido_id

```

- Exemplo:

```sql
SELECT
  o.order_id,
  o.order_date,
  c.first_name || ' ' || c.last_name AS cliente,
  p.product_name,
  oi.quantity,
  oi.unit_price,
  (oi.quantity * oi.unit_price) AS subtotal
FROM orders AS o
INNER JOIN customers AS c ON o.customer_id = c.customer_id
INNER JOIN order_items AS oi ON o.order_id = oi.order_id
INNER JOIN products AS p ON oi.product_id = p.product_id -- aqui não está diretamente ligado com a tabela 'orders'
WHERE o.status = 'DELIVERED';
```

## Seleção de colunas e uso do DISTINCT

- boas práticas: não usar o `SELECT *` para não retornar tudo; selecionar apenas as que serão utilizadas
- cada INNER JOIN com o `SELECT *` vai 'agrupando' o retorno
  - se cada tabela tiver 20 colunas, 2 INNER JOIN retornará 40 / 60 colunas (considerando a coluna esquerda e as demais)
- utilizar o `DISTINCT` para não ter cenários duplicados
  - cenários onde não quero trazer dados duplicados

- Exemplos:

```sql
-- Não criar a query desse modo, com o SELECT *
SELECT *
FROM orders AS o
INNER JOIN customers AS c ON o.customer_id = c.customer_id
INNER JOIN order_items AS oi ON o.order_id = oi.order_id
INNER JOIN products AS p ON oi.product_id = p.product_id -- aqui não está diretamente ligado com a tabela 'orders'
WHERE o.order_date BETWEEN '2024-06-01' AND '2024-08-31'
ORDER BY o.order_date DESC;

-- aqui não retornará as cidades duplicadas nos pedidos
SELECT DISTINCT c.city
FROM customers AS c
INNER JOIN orders AS o ON c.customer_id = o.customer_id;

-- concatenação e formatação de data
SELECT
  CONCAT(c.first_name, ' ', c.last_name) AS fullname,
  TO_CHAR(o.order_date, 'DD/MM/YYYY') AS formated_date
FROM customers AS c
INNER JOIN orders AS o ON c.customer_id = o.customer_id

SELECT
  o.order_id,
  CONCAT(c.first_name, ' ', c.last_name) AS fullname,
  TO_CHAR(o.order_date, 'DD-Mon-YYYY') AS readable_date -- 'Mon' traz o mês abreviado
FROM orders AS o
INNER JOIN customers AS c ON o.customer_id = c.customer_id
ORDER BY o.order_date DESC;
```

## SQL JOIN

- INNER JOIN => retorna todos os registros que possuem igualdade em 'ambos' os lados, ou seja, registros que possuem na tabela A e na tabela B;
  - se um registro da tabela a não existir na tabela b, não retorna o registro de a

## LEFT JOIN

- faz o INNER JOIN e retorna todos os registros da 'tabela da esquerda'
- Sintaxe:

```sql
SELECT *
  FROM tabela_a AS a -- tabela_a é a 'tabela da esquerda' nesse contexto
  LEFT JOIN tabela_b AS b
    ON a.chave = b.chave;
```

Exemplo:

```sql
SELECT
  p.product_id,
  p.product_name,
  oi.order_id
FROM products AS p -- essa é a tabela da esquerda
LEFT JOIN order_items AS oi
  ON p.product_id = oi.product_id;
```

- com o inner join, retorna apenas os registros que são 'comuns' entre ambas as tabelas, no left join retorna o que existe em comum mais todos os demais registros da tabela da esqueda;

```sql
SELECT
  p.product_id,
  p.product_name,
  oi.order_id
FROM products AS p
LEFT JOIN order_items AS oi
  ON p.product_id = oi.product_id
WHERE oi.order_item_id IS NULL -- basicamente esse WHERE 'remove' os retornos do INNER JOIN
ORDER BY p.product_name;
```

## RIGHT JOIN

- faz o INNER JOIN e retorna todos os registros da 'tabela da direita'
- Sintaxe:

```sql
SELECT *
  FROM tabela_a AS a
  RIGHT JOIN tabela_b AS b -- tabela_b é a tabela da direita nesta sintaxe
    ON a.chave = b.chave;
```

- Exemplo:

```sql
SELECT
  c.customer_id,
  c.first_name,
  c.last_name,
  o.order_id
FROM orders AS o
RIGHT JOIN customers AS c -- tabela customers é a tabela da direita nesse contexto
  ON o.customer_id = c.customer_id
```

- dica: a ordem das tabelas na query importa para que o retorno seja o esperado;
  - se a tabela da direita, por exemplo, já for a tabela com a FK, não retorna os registros NULL;

## FULL JOIN (OUTTER JOIN)

- junção completa (INNER JOIN + LEFT JOIN + RIGHT JOIN)
- Sintaxe:

```sql
SELECT *
  FROM tabela_a AS a
  FULL JOIN tabela_b AS b
    ON a.chave = b.chave;
```

## COALESCE

- coalesce: retorna o primeiro que não é NULL
- faz um tratamento para campos `NULL`;
- não altera a informação no banco, apenas no resultado da consulta
- string vazia é diferente de "NULL"
- usado para substituição de campos 'null'
- trabalha com strings e outros tipos de dados

- Exemplo:

```sql
-- seleciona clientes, se não tiver o primeiro nome, coloca 'sem nome'; se sobrenome for nulo, coloca desconhecido
SELECT
  COALESCE (c.first_name, 'SEM NOME') AS first_name_name,
  COALESCE (c.last_name, 'DESCONHECIDO') AS last_name_name
FROM customers AS c;
```

```sql
SELECT
  c.customer_id,
  COALESCE(c.first_name, 'SEM NOME') AS first_name,
  COALESCE(c.last_name, 'DESCONHECIDO') AS last_name,
  COALESCE(o.total_amount, 0) AS total_last_ship
FROM customers AS c
LEFT JOIN (
  SELECT
  customer_id,
  SUM(total_amount) AS total_amount
  FROM orders
  WHERE order_date >= '2024-01-01'
  GROUP BY customer_id
) AS o
  ON c.customer_id = o.customer_id;
```

## Filtros Pós Junção e Precedência

- dependendo do JOIN realizado, por exemplo, o retorno não é conforme esperado
- exemplo:
  - no caso abaixo, o WHERE maior que zero está transformando o LEFT JOIN em um INNER JOIN, uma vez que a quantidade deve ser maior do que zero, ou seja, os 'NULL' não aparecem.

```sql
SELECT *
FROM products AS p
LEFT JOIN order_items AS oi
  ON p.product_id = oi.product_id
WHERE oi.quantity > 0;
```

- nesse caso, usa-se o 'AND', que realiza o filtro de `quantity`, mas o LEFT JOIN é mantido

```sql
SELECT *
FROM products AS p
LEFT JOIN order_items AS oi
  ON p.product_id = oi.product_id
  AND oi.quantity > 0;

-- subconsulta
SELECT *
  FROM (
    SELECT
      p.product_id,
      p.product_name,
      oi.order_id,
      oi.quantity
    FROM products AS p
    LEFT JOIN order_items AS oi
      ON p.product_id = oi.product_id
  ) AS subq
  WHERE subq.quantity > 0 OR subq.quantity IS NULL;
```

## Subconsultas

- Não Correlacionada
  - Executa a subconsulta somente uma vez, retornando um conjunto de valores fixos para uso no WHERE ou outra cláusula

- Exemplo:

```sql
SELECT product_id, product_name
FROM products
WHERE price > (
  SELECT AVG(price) FROM products -- essa consulta é feita uma vez e o retorno será usado pelo WHERE
)
```

- Correlacionada
  - a subconsulta faz referência a colunas da consulta externa e é reexecutada para cada linha resultante

- Exemplo: (subconsulta relacionada ao SELECT)
  - cada retorno da consulta externa (c.customer_id, c.first_name, c.last_name,) vai executar a subconsulta no SELECT

```sql
SELECT
  c.customer_id,
  c.first_name,
  c.last_name,
  (SELECT COUNT (*) FROM orders o WHERE o.customer_id = c.customer_id) AS total_pedidos
FROM customers AS c;
```

- Performance: não correlacionada é mais rápida, executa uma vez só; a relacionada executa para cada linha da consulta externa

- exemplo de reescrita: JOIN + GROUP BY

```sql
SELECT
  c.customer_id,
  c.first_name,
  c.last_name,
  COUNT (o.oder_id) AS total_pedidos
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name;
```

## Junções com três ou mais tabelas

- Exemplo de Estudo:
  - tabelas: clientes, pedidos, itens_pedido e produtos

- vai fazendo o encadeamento dos 'JOINs'; lembrando que a ordem de encadeamento importa para a consulta

```sql
SELECT
  cli.customer_id,
  CONCAT(cli.first_name, ' ', cli.last_name) AS client_name,
  pd.order_id,
  TO_CHAR(pd.order_date, 'DD/MM/YYYY') AS ship_date,
  pr.product_id,
  pr.product_name,
  ip.quantity,
  ip.unit_price,
  (ip.quantity * ip.unit_price) AS subtotal
FROM customers AS cli
  -- 1) Cliente x Pedido
INNER JOIN orders AS pd ON cli.customer_id = pd.customer_id
  -- 2) Pedido x Itens Pedido
INNER JOIN orders_items AS ip ON pd.order_id = ip.order_id
  -- 3) Itens Pedido x Produto
INNER JOIN products AS pr ON ip.product_id = pr.product_id
WHERE pd.status = 'DELIVERED'
ORDER BY cli.customer_id, pd.order_date;
```
