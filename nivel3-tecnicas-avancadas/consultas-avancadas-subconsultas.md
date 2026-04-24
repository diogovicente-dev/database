# Consultas Avançadas e Subconsultas

## Subqueries (Subconsultas)

- _subquery_ é uma consulta SQL inserida dentro de uma outra consulta SQL principal
  -- a subconsulta funciona como uma 'filtragem' para a consulta principal

- quando usar
  -- filtrar dados com base em critérios dinâmicos
  -- realizar comparações com valores calculados
  -- executar agregações intermediárias
  -- reutilizar lógica de consulta
  -- reduzir a complexidade da lógica com CTEs ou JOINs

- tipos
  -- **scalar**
  --- retorna um único valor (linha x coluna)
  --- 1 linha x 1 coluna
  --- exemplo de uso: comparações (=, >, <, ...)
  -- **column**
  --- retorna uma coluna com vários valores
  --- N linhas x 1 coluna
  --- exemplo de uso: IN, NOT IN
  -- **row**
  --- retorna uma linha com várias colunas
  --- 1 linha x N colunas
  --- exemplo de uso: comparações com tuplas
  -- **table**
  --- retorna múltiplas linhas e colunas
  --- N linhas x N colunas
  --- exemplo de uso: EXISTS, IN, JOIN

```sql

-- exemplo 1
SELECT first_name, last_name
FROM customers
-- primero executa o WHERE e, dentro desse WHERE, existe uma subconsulta
WHERE customer_id IN (
  -- aqui é uma subconsulta, que retorna apenas os customer_id que existem na tabela orders;
  -- deste resultado, a consulta principal retorna o first_name e last_name apenas dos 'ids' identificados neste subconsulta
  SELECT customer_id
  FROM orders
);

-- exemplo 2
SELECT product_name, price
FROM products
WHERE price > (
  SELECT AVG(price)
  FROM products
);

-- exemplo 3
SELECT order_id, total_amount
FROM orders
WHERE total_amount = (
  SELECT MAX(total_amount)
  FROM orders
);

-- exemplo 4
SELECT product_id, product_name
FROM products p
WHERE EXISTS (
  SELECT 1
  FROM order_items oi
  WHERE oi.product_id = p.product_id
);
```

- a subconsulta não precisa necessariamente estar dentro da cláusula `WHERE`

```sql
SELECT product_name, price
  (SELECT AVG(price)
  FROM products p2
  WHERE p2.category_id = p1.category_id) AS category_medium
FROM products p1;
```

## Prática com Subconsultas

- exemplo: tabela `rh` => retornar os funcionários que possuem um salário maior do que a média do departamento

```sql
SELECT
  e.employee_id,
  e.first_name || ' ' || e.last_name AS fullname,
  d.department_name,
  s.salary_amount
FROM employees AS e
INNER JOIN departments AS d
  ON e.department_id = d.department_id
INNER JOIN salaries AS s
  ON e.employee_id = s.employee_id
  AND s.effective_to IS NULL
WHERE s.salary_amount > (
  -- essa subquery retorna a média salarial por departamento
  SELECT AVG(s2.salary_amount)
  FROM employees AS e2 -- alias diferente da consulta externa para não ter conflito
  INNER JOIN salaries AS s2
    ON e2.employee_id = s2.employee_id
    AND s2.effective_to IS NULL
  WHERE e2.department_id = e.department_id
)
ORDER BY d.department_name, s.salary_amount DESC;
-- a consulta externa vai listar apenas os funcionários que possuírem o salário maior do que a média do departamento
```

```sql
-- média salarial por departamento
SELECT AVG(s.salary_amount)
FROM employees AS e
INNER JOIN departments AS d
  ON e.department_id = d.department_id
INNER JOIN salaries AS s
  ON e.employee_id = s.employee_id
  AND s.effective_to IS NULL
WHERE d.deparment_name = 'Marketing'
```

## Conceitos Básicos de CTEs

**Common Table Expressions**

- expressões comuns para tabelas
- alternativas para consultas menos complexas

- CTE também é uma subconsulta nomeada temporária que pode ser referenciada dentro da query principal

```sql
-- crio a subconsulta 'fora' da query principal
WITH nome_cte AS (
  SELECT ...
)
SELECT * FROM nome_cte -- chamo a minha CTE criada depois
```

- Vantagens
  -- Legibilidade: facilita a leitura de consultas longas
  -- Reuso: reutiliza resultados intermediários sem duplicar lógica
  -- Modularização: divide uma consulta complexa em blocos
  -- Recursividade: permite percorrer hierarquias (em `WITH RECURSIVE`)

- Quando usar?
  -- consultas com múltiplos passos;
  -- agregações intermediárias reutilizadas;
  -- consultas recursivas

Exemplo: Calcular o valor média de cada pedido por cliente

```sql
-- no WITH é a CTE
-- essa subconsulta retorna o valor total de pedidos por cliente e a quantidade total de pedidos por cliente
WITH pedidos_por_cliente AS (
  SELECT
  customer_id,
  COUNT (*) AS total_pedidos, -- quantidade de pedidos por cliente (sem NULL)
  SUM(total_amount) AS valor_total
  FROM orders
  GROUP BY customer_id
)
-- consulta principal é essa abaixo
SELECT
  c.first_name,
  c.last_name,
  p.total_pedidos,
  ROUND(p.valor_total / p.total_pedidos, 2) AS ticket_medio
FROM pedidos_por_cliente p -- resultado da consulta CTE (subconsulta)
JOIN customers c -- JOIN da subconsulta com outra tabela
  ON c.customer_id = p.customer_id
Order BY ticket_medio DESC;
```

## CTE Não Recursivas e Recursivas

- Não Recursiva: executada apenas uma vez, sendo o cenário mais comum
- Recursiva: executada repetidamente, chamando a si mesma até uma condição de parada (como uma estrutura de repetição `while`)

Exemplo: total de vendas por cidade (não recursivo)

```sql
-- vendas por cidade executado apenas uma vez na consulta principal
WITH vendas_por_cidade AS (
  SELECT c.city, SUM(o.total_amount) AS total_vendas
  FROM customers c
  JOIN orders o ON c.customer_id = o.customer_id
  GROUP BY c.city
)
SELECT *
FROM vendas_por_cidade
WHERE total_vendas > 100
ORDER BY total_vendas DESC;
```

Exemplo 2: Não recursivo

```sql
WITH vendas_produto AS (
  SELECT p.product_name, SUM(oi.quantity) AS total_vendido
  FROM products p
  JOIN order_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_name
)
SELECT *
FROM vendas_produto
ORDER BY total_vendido DESC
LIMIT 10;
```

Exemplo 3: Recursivo

```sql
-- para ser recursiva tem que ter o 'recursive'
WITH RECURSIVE numeros AS (
  SELECT 1 AS n
  UNION ALL -- novo conceito (união de tudo)
  SELECT n + 1
  FROM numeros
  WHERE n < 10
)
SELECT * FROM numeros;
```

## Operações de Conjunto em SQL

- operações de conjunto em SQL servem para combinar os resultados de duas ou mais consultas com a mesma estrutura de colunas
- operadores:
  -- `UNION`: une o resultado de duas consultas e elimina duplicadas
  -- `UNION ALL`: une o resultado de duas consultas e mantém as duplicadas
  -- `INTERSECT`: retorna apenas os registros comuns entre as consultas (semelhante ao `INNER JOIN`)
  -- `EXCEPT`: retorna os registros da primeira consulta que não aparecem na segunda consulta

## UNION e UNION ALL

**UNION**

- elimina os registros duplicados
- regras:
  -- os conjuntos devem ter o mesmo número de colunas !!!
  -- tipos de dados devem ser compatíveis (por posição) !!!
  -- a ordenação final deve ser feita fora do `UNION` !!!

```sql
-- Exemplo 1: clientes que já compraram OU que ainda não compraram (sem duplicar)
SELECT customer_id, first_name, last_name -- número de colunas iguais nas duas consultas, tipos de dados iguais também
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders) -- essa subconsulta pega todos os clientes que estão em 'ordens', ou seja, que fizeram pedidos e desconsidera os duplicados; sem o DISTINCT, se o cliente fez 3 pedidos, considera 3 registros.
UNION

SELECT customer_id, first_name, last_name -- três colunas, conforme a primeira consulta
FROM customers
WHERE customer_id NOT IN (SELECT customer_id FROM orders);
```

**UNION ALL**

- considera os registros duplicados, não faz eliminação nenhuma
- processamento mais rápido que o `UNION`
- regras:
  -- os conjuntos devem ter o mesmo número de colunas !!!
  -- tipos de dados devem ser compatíveis (por posição) !!!
  -- a ordenação final deve ser feita fora do `UNION ALL` !!!

```sql
-- Exemplo 2: produtos vendidos e não vendidos (permitindo repetição)
SELECT product_id, product_name
FROM produtcs
WHERE product_id IN (SELECT product_id FROM order_items) -- não preciso me preocupar com registros duplicados

UNION ALL

SELECT product_id, product_name
FROM produtcs
WHERE product_id NOT IN (SELECT product_id FROM order_items);

--Exemplo 3:
WITH union_test AS (
  SELECT 'SP' AS origem, email FROM customers WHERE city = 'São Paulo'
  UNION ALL
  SELECT 'RJ' AS origem, email FROM customers WHERE city = 'Rio de Janeiro'
)
-- o select do union_test retorna uma tabela com todas as informações (RJ e SP como valor na coluna 'origem')
SELECT email, COUNT(*) AS ocorrencias
FROM union_test
GROUP BY email
HAVING COUNT(*) > 1;
```

## INTERSECT e EXCEPT

**INTERSECT**

- retorna apenas os registros que estão presentes em ambas consultas
- remove as duplicadas do resultado
- pode ser usado para verificar conincidência de dados

```sql
-- Exemplo: hipoteticamente, retorna os emails iguais que estão registrados na cidade de são paulo e na cidade do rio de janeiro
SELECT email FROM customers
WHERE city = 'São Paulo'

INTERSECT -- elimina as duplicadas e retorna apenas os registros que são em comum para as dias consultas

SELECT email FROM customers
WHERE city = 'Rio de Janeiro'
```

**EXCEPT**

- traz a diferença entre os conjuntos
- retorna os registros da primeira consulta que não existem na segunda consulta
- remove as duplicadas e que não tem na segunda consulta

```sql
-- Exemplo: produtos cadastrados que ainda não foram vendidos
SELECT product_id, product_name
FROM products

EXCEPT

SELECT DISTINCT p.product_id, p.product_name
FROM products p
JOIN order_items oi ON oi.product_id = p.product_id;
```

## WINDOW FUNCTION

- funções que calculam valores sobre um conjunto de linhas relacionadas, mantendo o detalhe de cada linha
- diferente do `GROUP BY`, não colapsam os dados: os detalhes individuais permanecem e os cálculos são feitos 'por cima'
- permite calcular médias, totais, rankings e contagens sem perder o contexto da linha
- útil para rankings, comparações, cumulativos, percentuais
- utilizam a cláusula `OVER (>>>)` para definir a janela de cálculo

Sintaxe Básica

```sql
FUNCAO_JANELA() OVER ( -- função janela () => SUM, COUNT, AVG, RANK, ...
  PARTITION BY coluna_de_divisao -- opcional, divide os dados em grupos, sem colapsar as linhas, semelhante ao GROUP BY
  ORDER BY coluna_de_ordenacao -- opcional, porém geralmente é utilizado
)
```

```sql
-- Exemplo 1:
SELECT
  o.customer_id,
  c.first_name,
  o.order_id,
  o.order_date,
  o.total_amount,
  SUM(o.total_amount) OVER ( -- aqui é a função de janela - SUM, indicando que vai somar o total amount de todos os pedidos
    PARTITION BY o.customer_id -- aqui ele indica que vai somar os totais por cliente, e mantém todas as linhas (registros) de ordens
    ORDER BY o.order_date
  ) AS total_acumulado
FROM orders o
JOIN customers c
  ON c.customer_id = o.customer_id
ORDER BY o.customer_id, o.order_date;

--Exemplo 2:
SELECT
  o.customer_id,
  c.first_name,
  o.order_id,
  o.total_amount,
  ROUND(100.0 * o.total_amount / SUM(o.total_amount) OVER ( -- ROUND a função de Janela
    PARTITION BY o.customer_id), 2
  ) AS percentual_sobre_total
FROM orders o
JOIN customers c
  ON c.customer_id = o.customer_id
ORDER BY o.customer_id, o.order_date;
```

## ROW NUMBER, RANK e DENSE RANK

- **ROW_NUMBER**
  -- atribui um número sequencial único para cada linha dentro de uma partição
  -- sempre sequencial: 1, 2, 3, ...
  -- ignora empates nos valores ordenados
- **RANK**
  -- atributi um número de ranking, com buracos em empates
  -- se duas linhas estão em 1º, por exemplo, a próxima linha será 3º (pula o 2º)
  -- ideal para ranking com posições visuais reais
- **DENSE_NUMBER**
  -- semelhante ao `RANK()`, mas sem os buracos nos empates
  -- ideal para ranking contínuo com pontuações iguais

```sql
-- Exemplo 1:
SELECT
  customer_id,
  order_id,
  order_date,
  total_amount,
  ROW_NUMBER() OVER (
    PARTITION BY customer_id ORDER BY total_amount DESC
  ) AS row_num
FROM orders;
-- nesse exemplo, se o cliente possuir 2 pedidos ou mais, para cada pedido do cliente vai ser atribuído um valor sequencial a partir do 1;
-- então, nesse caso, um cliente com três pedidos, a consulta vai retornar os 3 pedidos, sendo um com o valor 1, outro com o valor 2 e outro com o valor 3 na coluna 'row_num';

--Exemplo 2:
SELECT
  customer_id,
  order_id,
  order_date,
  total_amount,
  RANK() OVER (
    PARTITION BY customer_id
    ORDER BY total_amount DESC
  ) AS ranking
FROM orders;
-- mesma ideia do row_number, é um rank por 'agrupamento' feito
-- no exemplo faz um rank dos pedidos de cada cliente (1, 2, 3, ...)
-- se o cliente possuir apenas um pedido o rank a aparecer vai ser somente 1

--Exemplo 3:
SELECT
  customer_id,
  order_id,
  order_date,
  total_amount,
  DENSE_RANK() OVER (
    PARTITION BY customer_id
    ORDER BY total_amount DESC
  ) AS posicao
FROM orders;

-- Exemplo 4 trazendo tudo de uma vez
SELECT
  o.customer_id,
  c.first_name,
  o.total_amount,
  ROW_NUMBER() OVER (
    PARTITION BY o.customer_id
    ORDER BY o.total_amount DESC)
  AS rn,
  RANK() OVER (
    PARTITION BY o.customer_id
    ORDER BY o.total_amount DESC
  ) AS rk,
  DENSE_RANK() OVER (
    PARTITION BY o.customer_id
    ORDER BY o.total_amount DESC)
  AS drk
  FROM orders o
  JOIN customers c
    ON c.customer_id = o.customer_id
  ORDER BY o.customer_id, rk;
```

## Funções de Deslocamento (LAG e LEAD)

- funções de janela que permitem acessar valores de outras linhas relativas à linha atual, sem perder o detalhe linha a linha
- útil:
  -- comparações sequenciais
  -- análises de tendências (ex.: vendas no mês)
  -- identificação de variações, evolução ou regrassão

- **LAG**: referente à linha anterior, comparação com valores passados
- **LEAD**: referente à linha posterior, comparação com valores posteriores

Sintaxe:
_coluna_: coluna a ser analisada
_descolamento_ (offset): quantas linhas antes ou atrás será observado
_valorpadrao_: valor a retornar se não existir linha anterior / próxima

```sql
LAG(coluna, deslocamento, valor-padrao) OVER (
  PARTITION BY ...
  ORDER BY ...
)

LEAD(coluna, deslocamento, valorpadrao) OVER (
  PARTITION BY ...
  ORDER BY ...
)
```

Exemplos

```sql
-- Exemplo 1: comparar o total de vendas do mês com o mês anterior
WITH vendas_mensais AS (
  SELECT
    DATE_TRUNC('month', order_date) AS mes,
    SUM(total_amount) AS total_vendas
    FROM orders
    GROUP BY mes
  )
SELECT
  mes,
  total_vendas,
  LAG(total_vendas) OVER (ORDER BY mes) AS vendas_anterior, -- busca o total de vendas do mês anterior (por padrão o descolamento é sempre 1)
  total_vendas - LAG(total_vendas) OVER (ORDER BY mes) AS diferenca -- aqui uma diferença do total de vendas no mês menos o do mes anterior
  FROM vendas_mensais
  ORDER BY mes;

-- Exemplo 2: meses para frente
WITH vendas_mensais AS (
  SELECT
    DATE_TRUNC('month', order_date) AS mes,
    SUM(total_amount) AS total_vendas
    FROM orders
    GROUP BY mes
  )
SELECT
  mes,
  total_vendas,
  LEAD(total_vendas) OVER (ORDER BY mes) AS vendas_proximo_mes
  FROM vendas_mensais
  ORDER BY mes;
```
