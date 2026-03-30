# Consultas Avançadas

## Subtipos do SQL

**DDL, DML, DCL, TCL e DQL**

_DDL - Data Definition Language_

- engloba comandos responsáveis pela definição e estrutura do banco de dados
- criar, alterar eremover objetos como tabelas, índices, visões, esquemas, ...
- comandos principais:
  - `CREATE` - cria novos objetos
  - `ALTER` - altera a estrutura de um objeto existente (adicionar, remover, modificar)
  - `DROP` - remove objetos do banco (operação irreversível)
  - `TRUNCATE` - remove todas as linhas de uma tabela e não remove a estrutura (mais rápido que um DELETE sem WHERE)
  - `RENAME` - renomear o nome de um objeto (tabela)
  - `COMMENT` - adiciona ou altera comentários de descrição para os objetos

_DML - Data Manipulation Language_

- engloba os comandos para inserir, atualizar, deletar e selecionar
- operações _CRUD_
  - `INSERT` - inserir dados em objetos
  - `UPDATE` - atualizar dados em objetos
  - `DELETE` - remover dados em objetos
  - `UPSERT` => (INSERT ... ON CONFLITCT) => inserir dados em objetos e, se existente, atualizar (update)

_DQL - Data Query Language_

- comando `SELECT` e correlacionados
- cláusulas: WHERE, GROUP BY, HAVING, ORDER BY, JOIN, LIMIT, FROM, ..

_DCL - Data Control Language_

- engloba permissões (privilégios) sobre objetos
  - `GRANT` - concede permissão
  - `REVOKE` - remove os privilégios

_TCL - Transaction Control Language_

- engloba as transações nos objetos
  - `BEGIN` => START TRANSACTION
  - `COMMIT` => salvar os dados da transação
  - `ROOLBACK`
  - `SAVEPOINT`
  - `SET TRANSACTION`

## WHERE

- estrutura do `where` => filtro
- fazer filtro dos dados que queremos selecionar
- muito importante aplicar o `where` para processamento, não é necessário trazer todos os dados
- `WHERE` trabalha com exatidão
- pode trabalhar com qualquer tipagem (string, int, date, ... )

```sql
-- where simples
SELECT * FROM customers WHERE city = 'Porto Alegre'
```

- altamente recomendável uso do `WHERE`

### Operações de Comparação com WHERE

- _> maior_
- _< menor_
- _= igual_
- _<> diferente_
- _>= maior ou igual_
- _<= menor ou igual_

```sql
-- selecionando os produtos que o preço for maior do que 100
SELECT product_id, product_name, price FROM products
WHERE price > 100;

SELECT product_id, product_name, category_id
FROM products
WHERE category_id = 3; -- filtra categorias igual a 3

SELECT oder_id, status
FROM orders
WHERE status <> 'DELIVERED'; -- seleciona os pedidos com status diferente de 'delivered'

SELECT customer_id, created_at
FROM customers
WHERE created_at < '2024-06-01'; -- filtra os clientes cadastrados antes de '01-06-2024'
-- menor, maior, menor igual ou maior igual funciona para números e para data

SELECT product_id, price
FROM products
WHERE price <= 50; -- filtra produtos com preço menor ou igual a 50
```

- _|| concatenando_
  - uso de '2 pipes' para concatenar a informação e criar um 'novo'
  - na consulta, 'criar' uma nova coluna para receber essa 'nova informação'
  - `AS` como se fosse um 'apelido' para a coluna, é temporário e não cria uma nova coluna em nenhuma tabela

```sql
-- concatenando o first com o last, concatenando com um 'espaço'

SELECT customer_id, first_name || ' ' || last_name AS full_name, city
FROM customers
WHERE city = 'Sao Paulo'
```

- filtrando por algo 'nulo'
  - usa `IS NULL`
  - se quiser os não nulos, usar `IS NOT NULL`

```sql
SELECT * FROM customers
WHERE last_name IS NULL; -- filtra os registros que o last_name é nulo

SELECT * FROM customers
WHERE last_name IS NOT NULL; -- filtra os registros que o last_name não é nulo
```

### AND e OR com WHERE

- concatenação de filtros
  - AND = operator 'E'
    - ambas condições precisam ser verdadeiras para buscar;
  - OR = operator 'OU'
    - uma ou outra precisa ser verdadeira

```sql
SELECT customer_id, first_name, last_name, city, created_at
FROM customers
WHERE city = 'Sao Paulo'
AND created_at >= '2024-01-01' -- adicionando 2 filtros de uma vez (atende os dois)

SELECT product_id, product_name, price
FROM products
WHERE price < 50 OR price > 90; -- aqui ou é menor que 50 ou maior que 90;

-- mais complexo --
SELECT order_id, total_amount, status
FROM orders
WHERE (status = 'DELIVERED' OR status= 'SHIPPED') -- primeiro avalia o parênteses
AND total_amount > 600; -- depois avalia o total aqui
```

### IN e NOT IN com WHERE

- `IN` uso para ou um ou outro também com sintaxe específica
  - como se fossem vários `OR` em paraelo

- `NOT IN` - filtro não está nos filtros determinados

```SQL
SELECT order_id, status
FROM orders
WHERE status IN ('PENDING', 'SHIPPED'); -- 'IN' em caso de um ou outro status

SELECT product_id, category_id
FROM products
WHERE category_id IN (1,3,5);
```

```sql
SELECT product_id, category_id
FROM products
WHERE category_id NOT IN (2,4);
```

### BETWEEN com WHERE

- buscar por um intervalo de valor para filtros
- o valor do intervalo é incluído na busca/filtro
- ÓTIMO para dados numéricos e datas

```sql
SELECT product_id, price
FROM products
WHERE price BETWEEN 150 AND 300;

SELECT order_id, order_date
FROM orders
WHERE order_date BETWEEN '2024-05-01' AND '2024-05-31';
```

- dá para usar em strings, mas não tão assertivo, não ideal

### LIKE com WHERE

- trazer um filtro mais 'flexível' de acordo com o cenário a ser pesquisado
- permite fazer pesquisas com 'parte' de strings
  - `%` importante para pesquisa, pois permite 'qualquer carectere ou sequência desejada';
  - `_` underline indica um caractere qualquer na pesquisa

```sql
SELECT customer_id, first_name, last_name
FROM customers
WHERE last_name LIKE 'Ju%'; -- segredo o %, nesse caso começa com "Ju"

SELECT customer_id, first_name, last_name
FROM customers
WHERE last_name LIKE '%Silva';

SELECT customer_id, first_name, last_name
FROM customers
WHERE first_name LIKE '%ia%';

SELECT product_id, product_name
FROM products
WHERE product_name LIKE '_roduto 1%';
-- o underline significa que pode ser qualquer caractere
-- nessa consulta o 'roduto 1' é uma condição verdadeira
```

### COUNT e DISTINCT com WHERE

**COUNT**

- contar/contabilizar a quantidade de registros
- ao inserir a coluna, automaticamente não retorna os 'nulos'

```sql
SELECT COUNT (*) AS total_pedidos
-- coluna 'temp' para contabilizar
FROM orders;
-- inclui todas as linhas, mesmo que sejam 'NULL'

SELECT COUNT (orders.total_amount) AS total_amount_not_null
FROM orders;
-- nesse cenário os 'não nulos' não serão contabilizados

SELECT COUNT (customers.last_name) AS total_customers_not_null
FROM customers;
```

**DISTINCT**

- retorna os registros sem ser duplicados

```sql
SELECT COUNT(DISTINCT customers.first_name) AS total_customers_not_null
FROM customers;

-- nessa segunda chamada retorna os 'null'
SELECT DISTINCT customers.first_name
FROM customers;
```

## SUM, AVG, MAX, MIN e Agrupamentos

**SUM**

- somar valores numéricos, ignora 'null' para nçao dar erro na soma

**AVG**

- faz a média dos valores numéricos de acordo com a quantidade de registros

```sql
SELECT SUM(total_amount) AS faturamento_geral
FROM orders;

SELECT AVG(total_amount) AS media_pedidos
FROM orders;
```

- fazendo tudo de uma vez (contando, somando e fazendo a média)

```sql
SELECT COUNT (total_amount), SUM(total_amount) AS faturamento_geral,  AVG(total_amount) AS media_pedidos
FROM orders;
```

**MAX e MIN**

- retorna o maior valor de uma coluna e o menor valor de uma coluna

```sql
SELECT MAX(price) AS preco_max, MIN(price) AS preco_min
FROM products;

SELECT MIN(created_at) AS primeira_data_cadastro
FROM customers;
```

## ORDER BY

- ordenação dos registros de acordo com os valores de uma das colunas
  - `ASC` => menor para o maior, se numérico; ordem alfabética, data, ...; por padrão já é ASC;
  - `DESC` => maior para o menor, se numérico;

```sql
SELECT product_id, product_name, price
FROM products
ORDER BY price ASC;

SELECT product_id, product_name, price
FROM products
ORDER BY price DESC;
```

- ordenação conjunta

```sql
SELECT order_id, status, order_date
FROM orders
ORDER BY status ASC, order_date ASC;
```

- segue uma sequencia 'lógica': faz a primeira ordenação (status) paa depois fazer a segunda ordenação (order_date)

## GROUP BY

- agrupamento de registros de acordo com os valores de uma ou mais colunas
- muito utilizado para fazer análises e sumarizações de dados
- as colunas não agregadas (que não estão em funções de agregação) precisam estar no `GROUP BY`
- muito utilizado com funções de agregação (COUNT, SUM, AVG, MAX, MIN) para obter resultados mais significativos

```sql
-- quantidade de vendas por produto
SELECT product_id, -- coluna para agrupar e aparecer no resultado
COUNT(*) AS total_vendas
FROM order_items
GROUP BY product_id;

-- quantidade de itens vendidos por produto (considerando a quantidade de cada item)
SELECT product_id,
SUM(quantity) AS total_quantidade
FROM order_items
GROUP BY product_id;

-- quantidade de pedidos por status
SELECT status,
COUNT(*) AS quantidade_pedidos
FROM orders
GROUP BY status;

-- quantidade de clientes por cidade
SELECT city,
COUNT(*) AS total_clientes
FROM customers
GROUP BY city;

```

## Uso do HAVING

- adiciona um filtro para os grupos criados pelo `GROUP BY`
- funciona como um `WHERE` para os grupos, filtrando os resultados agregados

```sql
SELECT category_id,
COUNT(*) AS quantidade_produtos
FROM products
GROUP BY category_id
HAVING COUNT(*) > 2; -- filtra grupos com mais de 2 produtos

SELECT o.customer_id,
COUNT(o.order_id) AS total_pedidos_2024
FROM orders o -- apelido para a tabela orders 'o' para facilitar a escrita
WHERE o.order_date BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY o.customer_id
HAVING COUNT(o.order_id) > 1; -- filtra os clientes que fizeram mais de 1 pedido em 2024

SELECT oi.product_id,
SUM(oi.quantity) AS unidades_vendidas
FROM order_items oi
GROUP BY oi.product_id
HAVING SUM(oi.quantity) > 2; -- filtra os produtos que venderam mais de 2 unidades
```

## Uso de Agregação e LIMIT

- `LIMIT` limita a quantidade de registros retornados
- usado para retornar os top 5, top 10, ... de acordo com a ordenação definida

```sql
SELECT oi.product_id,
SUM(oi.quantity) AS quantidade_total
FROM order_items oi
GROUP BY oi.product_id
ORDER BY SUM(oi.quantity) DESC
LIMIT 10;
-- nesse cenário, o `LIMIT` é aplicado depois do `GROUP BY` e do `ORDER BY`, ou seja, ele limita os resultados já agrupados e ordenados, retornando apenas os top 10 produtos mais vendidos.


SELECT o.customer_id,
SUM(o.total_amount) AS faturamento_total
FROM orders o
GROUP BY o.customer_id
ORDER BY faturamento_total DESC
LIMIT 10;
```

## Manipulação de STRINGS

- transformação de dados em strings
- concatenar

```SQL
-- duas formas de concatenar
SELECT customer_id,
CONCAT(first_name, ' ', last_name) AS nome1,
first_name || ' ' || last_name AS nome2
FROM customers
LIMIT 10;
```

- tamanho da string (caracteres)

```SQL
SELECT customer_id,
  last_name,
  CHAR_LENGTH(last_name) AS tamanho_last_name
FROM customers
LIMIT 10;
```

- modificações, padronizações
  - UPPER e LOWER

```SQL
SELECT customer_id,
UPPER(first_name) AS nome_maiusculo,
LOWER(last_name) AS sobrenome_minusculo
FROM customers
```

- TRIM - remover espaços indesejados

```SQL
SELECT '   Carlos   ' AS texto_original,
TRIM('   Carlos   ') AS texto_trim,
LTRIM('   Carlos   ') AS texto_ltrim,
RTRIM('   Carlos   ') AS texto_rtrim;
```

- extrair parte de um texto - substring

```SQL
SELECT product_id, product_name,
SUBSTRING (product_name FROM 1 FOR 3) AS first_3_charc
FROM products
LIMIT 5;
```

- saber a posição da string

```SQL
SELECT customer_id, last_name,
POSITION ('Silva' IN last_name) AS position_last_name
FROM customers
WHERE last_name LIKE '%Silva%';
```

- replace

```SQL
SELECT customer_id, last_name,
REPLACE (last_name, 'Silva', 'S.' ) AS last_name_ab
FROM customers
WHERE last_name LIKE '%Silva%';
```

## Manipulação de NÚMEROS e conversões com CAST

- arredondar - ROUND

```SQL
SELECT product_id, price,
  ROUND(price, 0) AS price_arr, -- arredonda para inteiro
  ROUND(price, 1) AS price_arr2, -- 1 casa decimal
  ROUND(price, 2) AS price_arr3 -- 2 casas decimais
FROM products
LIMIT 10;
```

- truncate - 'truncar valores' - trunc
  - não aplica o arredondamento

```SQL
SELECT product_id, price,
  TRUNC(price, 0) AS price_arr, -- inteiro (não arredonda)
  TRUNC(price, 1) AS price_arr2, -- 1 casa decimal
  TRUNC(price, 2) AS price_arr3 -- 2 casas decimais
FROM products
LIMIT 10;
```

- CEIL e FLOOR
  - arredonda para o menor e maior inteiro de acordo com o atributo selecionado

```SQL
SELECT product_id, price,
  CEIL(price) AS price_ceil, -- menor inteiro >= price
  FLOOR(price) AS price_floor -- maior inteiro <= price
FROM products
LIMIT 10;
```

- ABS, POWER, SQRT
  - valor absoluto, potenciação e radiciação

```SQL

SELECT product_id, price,
  ABS(price * -1) AS price_abs,
  POWER(price, 2) AS price_power,
  SQRT(price) AS price_square
FROM products
LIMIT 10;
```

- MOD
  - resto da divisão

```SQL
SELECT product_id, price,
  MOD(CAST(price AS INT), 7) AS rest_7
FROM products
LIMIT 10;
```

- CAST / CONVERT (SGBD - SQl Server)
  - converter um tipo de valor para outro tipo de dado
  - tomar cuidado

```SQL
SELECT product_id, price,
  CAST(price AS INT) AS price_int -- CAST (column AS type)
FROM products
LIMIT 10;
```

```SQL
SELECT customer_ID, created_at,
CAST(created_at AS VARCHAR(20)) AS creted_at_text
FROM customers
LIMIT 10;
```

## Explorando Relatórios 1 e 2

- exemplos =>
  - 5 produtos mais vendidos, agrupar produto, ordenar do maior para o menor
  - Receita mensal em 2024?
  - produtos vendidos em jaeiro e fevereiro 2024 com mais de 100 unidades

```sql
-- select mais elaborando, aninhando um novo select 'interno'
SELECT product_id,
  SUM(quantity) AS total_jan_feb
FROM order_items
WHERE order_id IN (
  SELECT order id
  FROM orders
  WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
)
GROUP BY product_id
HAVING SUM(quantity) > 1
ORDER BY SUM(quantity DESC);
```

```sql
-- no 'quarter' pode ser 'month' para mensal,
SELECT DATE_TRUNC('quarter', order_Date) AS quarter_year
  SUM(total_amount) AS quarter_invoice
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY DATE_TRUCN('quarter', order_date)
ORDER BY DATE_TRUCN('quarter', order_date);
```

## Subconsultas e EXISTS IN

- subconsulta: consulta interna que retorna um conjunto de valores utilizado com critério de filtro na cláusula `WHERE`

_Sintaxe_

```sql
SELECT *
FROM tabela_principal
WHERE coluna_principal IN (
  SELECT coluna_secundaria
  FROM tabela_secundaria
  WHERE condicao
)

```

- exists / not exists: testa a existência (ou não) de linhas retornadas por uma subconsulta relacionada

_Sintaxe_

```sql
SELECT *
FROM tabela_principal tp
WHERE EXISTS (
  SELECT 1
  FROM tabela_secundaria ts
  WHERE ts.chave = tp.chave
)

```

**Diferença entre IN e EXISTS**

- IN: compara a coluna com um conjunto de valores estáticos retornados por uma subconsulta não correlacionada (bom para listas pequenas)
  - é mais intuitivo quando filtramos contra valores literais ou o conjunto retornado é pequeno e estático
- EXISTS: percorre a subconsulta correlacionada linha a linha e retorna (TRUE / FALSE) na primeira ocorrência, geralmente mais eficiente em grandes volumes de dados
  - ideal para saber se há pelo menos um registro relacionado (sem precisar comparar valores exatos)

Exemplos

```sql
-- IN
SELECT  o.order_id, o.customer_id, o.order_date, o.total_amount
FROM orders AS o
WHERE o.customer_id IN ( -- essa subconsulta será feita primeiro da consulta principal
  SELECT c.customer_id
  FROM customers AS c
  WHERE c.city = 'São Paulo'
)
ORDER BY o.order_date;

-- EXISTS
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.customer_id = c.customer_id
  AND o.status = 'DELIVERED'
)
ORDER BY c.customer_id

-- NOT EXISTS
SELECT p.product_id, p.product_name
FROM products p
WHERE NOT EXISTS (
  SELECT 1
  FROM order_items oi
  WHERE oi.product_id = p.product_id
)
ORDER BY p.product_id;
```

## CASE e FILTER

- CASE - usado dentro de funções de agregação
  - SUM, AVG, ...
  - ideia de um IF - ELSE da programação
- FILTER - permite a inserção de uma cláusula WHERE dentro

```sql
--CASE
SELECT o.status,
SUM (CASE WHEN o.status = 'DELIVERED' THEN o.total_amount ELSE 0 END) AS total_entregue,
SUM (CASE WHEN o.status = 'PENDING' THEN o.total_amount ELSE 0 END) AS total_pendente,
SUM (o.total_amount) AS total_geral
FROM orders o
GROUP BY o.status;

--FILTER
SELECT DATE_TRUNC('month', o.order_date) AS mes,
SUM(o.total_amount) AS total_geral,
SUM(o.total_amount) FILTER (WHERE o.status = 'DELIVERED') AS total_entregue,
SUM(o.total_amount) FILTER (WHERE o.status = 'PENDING') AS total_pendente
FROM orders o
GROUP BY DATE_TRUNC('month', o.order_date)
ORDER BY mes;

```
