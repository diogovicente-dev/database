# Otimização e Performance

## Índices - conceitos básicos

Conceito: é uma estrutura auxiliar que acelera buscar, ordenações e junções, montada a parir de uma ou mais colunas de uma tabela.

- _benefícios_:
  - reduz custo de varredura completa (seq scan) para buscar pontuais ou intervalos;
  - melhora performance de consultas **WHERE, JOIN ORDER BY e GROUP BY**
  - o uso deve ser com cautela:
- _custos_:
  - uso de espaço em disco/memória
  - sobrecarga em **INSER/UPDATE/DELETE** (cada operação de escrita também atualiza índices)

- banco com muitas consultas, poucos inserts, melhor com índices; ao contrário, melhor não usar os índices

- o índice sempre será associado a uma coluna de uma tabela
-

```sql
-- criar índice
CREATE INDEX [CONCURRENTLY] nome_do_indice -- criar um nome para o índice
  ON tabela [USING metodo] (coluna1[ASC|DESC], coluna2, ...); -- na tabela 'x'

-- índice único na base de dados
CREATE UNIQUE INDEX nome_indice_unico
  ON tabela (coluna);
```

```sql
EXPLAIN ANALYZE
-- ao executar esse comando, traz o SEQ SCAN, que é um detalhamento do custo, tempo, processamento, ... para executar a consulta (linhas removidas, tempo de consulta)
SELECT * FROM products
WHERE price BETWEEN 100 AND 200;

CREATE INDEX idx_products_price
-- nome do índice, como uma boa prática colocar o "idx" na frente
  ON products(price);

-- depois de curar o índice, executar com o EXPLAIN ANALYZE que mostrará que o tempo de execução é bem mais rápido
```

## Árvores B-Tree - Hash - GiST - GIN

forma de criação dos índices

- **B-Tree** (padrão utilizado ao criar um índice)
  - equilíbrio ótimo para busca de igualdade e intervalo (`BETWEEN, >, <`)
- **Hash**
  - otimizado apenas para igualdade (=)
  - não suporta buscas por intervalo
- **GiST** (Generalized Search Tree)
  - permite índices sobre tipos geométricos, arrays, texto completo;
  - suporta operações "aproximadas" (p.ex. índice de similaridade)
  - onde uso: textos longos, banco de dados espaciais, ...
- **GIN** (Generalized Inverted Index)
  - ideal para colunas com múltiplos valores (arrays, JSONB);
  - mantém um _inverted list_ de valores -> rápida para a existência de elemento;

- armazenamento
  - cada índice é uma tabela interna (chamada de _pg_class - postgre class_) com páginas de dados organizadas em nós folha e não-folha;
  - organiza os dados para tornar a consulta mais eficiente
- pesquisa
  - usa o planner para escolher entre _seq scan_ e _index scan_
  - em _index scan_, busca-se nas páginas raiz -> páginas folhas -> retorna as tuplas correspondentes

```sql
CREATE INDEX idx_products_price_hash
  ON products USING HASH(price); -- usando o modelo 'hash' nesse caso

EXPLAIN ANALYZE
SELECT * FROM products
WHERE price = 249.37; -- hash suportado para este tipo de comparação
```

```sql
ALTER TABLE products
  ADD COLUMN tags TEXT[];
UPDATE products
  SET tags = ARRAY['promo', 'novo']
  WHERE product_id % 5 = 0;

-- index GIN
CREATE INDEX idx_products_tags_gin
  ON products USING GIN(tags);

EXPLAIN ANALYZE
SELECT * FROM products
WHERE tags @> ARRAY['promo'];
```

## Escolha e Criação de Índices

- custo de leitura x escrita
  - leitura: índices diminuem input/output de leitura para buscar seletivas, gerando melhor performance na busca, mais eficiência;
  - escrita: todo INSERT / UPDATE / DELETE que afete a coluna indexada gera manutenção extra no índice, impactando _throughput_, ou seja, caso uma ação dessa seja efetuada, é necessário um 'recálculo' dos índices considerando o novo registro inserido, o registro atualizado ou o registro deletado;
  - trade-off: quanto mais índices, mais 'penalidade' na escrita

- índices únicos e não únicos
  - não único: **default** => _CREATE INDEX_
  - único: garante unicidade em colunas => _CREATE UNIQUE INDEX_

- **índice não único**: uma coluna que pode ter valores iguais
  - exemplo: preço / cidade
- **índice único**: coluna onde precisamos garantir que os valores precisem ser diferentes
  - exemplo: email, cpf, cnpj,

**Índices Parciais e Expressões Indexadas**

- parciais: indexa apenas _subset_ de linhas, reduzindo tamanho e custo de manutenção
  - índice mais específico, mais restritos
  - no exemplo abaixo, em vez de criar um índice para toda a coluna _oder_date_, o índice será criado apenas para as _orders_ com status PENDING.

```sql
CREATE INDEX idx_order_pending
ON orders(order_date)
WHERE status = 'PENDING'
```

- expressão indexada: indexa o resultado de uma expressão qualquer
  - como se agregasse uma regra de negócio nesse índice
  - no exemplo, para trabalhar apenas com letras minúsculas para o e-mail

```sql
CREATE INDEX idx_customers_lower_email
ON customers (LOWER(email));
```

## Demais Recursos dos Índices

- índices concorrentes
  - no momento da criação dos índices, a tabela é bloqueada temporariamente para que o índice seja contruído;
  - uma tabela com muitos dados o tempo de construção é alto e, se tiver alto índice de insert/update/delete, o tempo é maior e mais custoso para fazer a criação do índice;
  - palavra _CONCURRENTLY_ - operação paralela
  - em bancos de dados em produção é praticamente impossível parar a operação para criar o índice;

```sql
CREATE INDEX CONCURRENTLY index_name
ON table_name(column);
```

- índices com mais de uma coluna
  - consultas que envolvem múltiplas colunas
  - otimização de consultas (indexação em ambas colunas)

```sql
CREATE INDEX index_name
ON table_name(column1, column2);
```

- recalcular índices
  - corrigir, ajustar, corrigir fragmentos

```sql
REINDEX INDEX index_name
```

- limpeza

```sql
VACUUM (ANALYZE) products;
```

## Explain e Análise de Planos de Execução

- EXPLAIN ANALYZE é um comando para analisar o plano de execução de consultas SQL;
  - análise de desempenho de uma consulta

## Leituras Avançadas - Index Scan / Bitmap Scan

- Index Scan => quando trabalhar com índices em tabelas
  - para forçar o 'explain analyze' precisa desabilitar o 'seq scan', pois com poucos registros o próprio postgre entende que o 'índice' não é necessário

```sql
EXPLAIN
SELECT * FROM products WHERE price > 300;

EXPLAIN ANALYZE SELECT * FROM products WHERE price > 300;

-------

CREATE INDEX idx_price ON products(price);

SET enable_seqscan = off;
EXPLAIN ANALYZE
SELECT * FROM products
WHERE price > 300;

SET enable_seqscan = on;

EXPLAIN ANALYZE
SELECT o.order_id, c.first_name, c.last_name
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
WHERE o.status = 'SHIPPED';
```

- HASH JOIN => quando existe a consulta com duas ou mais tabelas
  - faz uma análise de cada tabela (seq scan de cada tabela)

## Otimizando Consultas Complexas - Reescrita de Consultas e Refatoração

- subquery escalar: subconsulta que retorna um único valor

```sql

-- esse primeiro caso a subquery será executada para cada linha da tabela 'orders', problema de performance na consulta --
-- esse primeiro caso deve ser evitado quando a consulta for percorrer muitos registros --
SELECT
  o.order_id,
  (SELECT COUNT (*)
    FROM order_items oi
    WHERE oi.order_id = o.order_id) AS itens
FROM orders o
WHERE o.status = 'DELIVERED';

-- no segundo caso a subquery é substituída por um 'count' direto e um group by para retornar corretamente
-- melhor utilizar joins com group-by com tabelas grandes, melhor do que usar as subqueries --
SELECT
  o.order_id,
  COUNT (oi.order_id) AS itens
FROM orders o
LEFT JOIN order_items oi
  ON o.order_id = oi.order_id
WHERE o.status = 'DELIVERED'
GROUP BY o.order_id;
```

- Otimização com pré-agregação
  - JOINs com grande volume de dados
  - se fazer o JOIN antes da agregação, precisa de mais recursos,

```sql
-- exemplo mais 'custoso'
SELECT
  o.order_id,
  o.total_amount,
  SUM(oi.quantity * oi.unit_price)
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status = 'DELIVERED'
GROUP BY o.order_id, o.total_amount;

-- CTE (commom table expression) - pré-agregação antes do JOIN --
WITH soma_itens AS (
  SELECT
    order_id,
  SUM(quantity * unit_price) AS subtotal
  FROM order_items
  GROUP BY order_id
)
SELECT
  o.order_id,
  o.total_amount,
  si.subtotal
FROM orders o
JOIN soma_itens si
  ON si.order_id = o.order_id
WHERE o.status = 'DELIVERED';

```

## Particionamento de Tabelas e Dados no PostgreSQL

- recurso no qual o administrador do banco de dados poderá ter o domínio dos locais onde seis dados são armazenados
  - _filegroups_
- **PARTITION BY RANGE**
  - divide dados em faixas contínuas de valores (datas, numéricos);
  - quando usar: séries temporais, logs, ordens por data;
- **PARTITION**
  - técnica de divisão física de uma tabela grande em partes menores (partições)
  - partições são fisicamente separadas (tabelas separadas, independetes), porém logicamente é uma única tabela **PAI**
  - _by range, list, default_

- usa partition para melhorar a performance;
- organização de dados por 'tempo', com muitos dados históricos;
- melhor manutenção (dados históricos mais fácil para serem removidos);
- mais facilidade para fazer agregações - _CTEs_

```sql
-- criando a tabela 'part' PAI de orders;
CREATE TABLE orders_part (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE NOT NULL,
    status VARCHAR(20),
    total_amount NUMERIC(12, 2),
    PRIMARY KEY (order_date, order_id)
) PARTITION BY RANGE (order_date);

-- após criada a tabela pai, é possível criar as tabelas 'filhos'; no caso, para 2 meses considerando que o partition é 'by range' = order_date
CREATE TABLE orders_2024_01 PARTITION OF orders_part
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders_part
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- criando os registros para testes; inserção 'direto' na tabela 'pai'
INSERT INTO orders_part (order_id, customer_id, order_date, status, total_amount)
VALUES (250, 123, '2024-01-15', 'SHIPPED', 250.00);

INSERT INTO orders_part (order_id, customer_id, order_date, status, total_amount)
VALUES (350, 123, '2024-02-15', 'SHIPPED', 250.00);

-- seleção direto da tabela pai também
SELECT * FROM orders_part
WHERE order_date
BETWEEN '2024-01-01' AND '2024-01-31';

SELECT * FROM orders_part
WHERE order_date
BETWEEN '2024-02-01' AND '2024-02-28';

-- ao executar a consulta, em vez de acessar a tabela 'inteira' de orders, busca apenas onde está alocada a partição com os dados correspondentes
```

## Demais Tipos de Particionamento de Tabelas

- **LIST**
- **HASH**
- **DEFAULT**

- list => trabalhar com conjuntos discretos de valores
  - ideal para categorização (status, por exemplo)
- hash => valor hash da chave, agrupamento com base no hash
- default => se não conseguiu categorizar os dados, cria-se um 'default' do que não conseguiu fazer

```sql
-- exemplo list: criação pelo status do pedido
CREATE TABLE orders_part (
  order_id INT GENERATED ALWAYS AS IDENTITY,
  customer_id INT,
  status VARCHAR (20),
  order_date DATE,
  total_amount NUMERIC(12,2))
PARTITION BY LIST (status);

-- criação das partições
CREATE TABLE orders_shipped PARTITION OF orders_part
  FOR VALUES IN ('SHIPPED');

CREATE TABLE orders_delivered PARTITION OF orders_part
  FOR VALUES IN ('DELIVERED');

INSERT INTO orders_part (customer_id, status, order_date, total_amount)
  VALUES (123, 'SHIPPED', '2024-06-10', 150.00);

-- ao executar busca direto da partição shipped
SELECT * FROM orders_part WHERE stauts = 'SHIPPED'



-- HASH
CREATE TABLE customers_part (
  customer_id INT PRIMARY KEY,
  name TEXT)
PARTITION BY HASH (customer_id);
-- dados serão particionados automaticamente nas tabelas a serem criadas;
-- a hash, por ser inteiro, a regra é separar 'matematicamente'

CREATE TABLE customers_part_0 PARTITION OF customers_part
  FOR VALUES WITH (MODULUS 4, REMAINDER 0);
  -- modulus 4, remainder 0 => valor do hash dividido por 4 igual a 0;

CREATE TABLE customers_part_1 PARTITION OF customers_part
  FOR VALUES WITH (MODULUS 4, REMAINDER 1);
  -- modulus 4, remainder 0 => valor do hash dividido por 4 igual a 1;

CREATE TABLE customers_part_2 PARTITION OF customers_part
  FOR VALUES WITH (MODULUS 4, REMAINDER 2);
  -- modulus 4, remainder 0 => valor do hash dividido por 4 igual a 2;

CREATE TABLE customers_part_3 PARTITION OF customers_part
  FOR VALUES WITH (MODULUS 4, REMAINDER 3);
  -- modulus 4, remainder 0 => valor do hash dividido por 4 igual a 3;

INSERT INTO customers_part (customer_id, name)
  VALUES (101, 'Alice');

SELECT * FROM customers_part;


-- DEFAULT => partição que 'abrange' as inserções que não se enquadram nas outras partições criadas
CREATE TABLE orders_part (
  order_id INT GENERATED ALWAYS AS IDENTITY,
  status VARCHAR (20),
  customer_id INT)
PARTITION BY LIST (status);

CREATE TABLE orders_pending PARTITION OF orders_part
  FOR VALUES IN ('PENDING');

CREATE TABLE orders_failed PARTITION OF orders_part
  FOR VALUES IN ('FAILED');

CREATE TABLE orders_default
  PARTITION OF orders_part DEFAULT;

INSERT INTO orders_part (status, customer_id) VALUES ('CANCELLED', 321);

SELECT * FROM orders_part;
```

## Normalização e Denormalização

- desvantagens em cenários analíticos
  - múltiplos _JOINS_: consultores OLAP que agregam dados em grandes volumes sofrem com o custo de junções
  - latência: leitura de várias tabelas pode ser mais lenta que leitura de uma única tabela ampla

- cuidado com vários 'JOINS'
  - pensar sempre em uma maneira para não 'pesar' as consultas
  - CTEs, Funções Janelas, ...

- denormalização
  - sacrificar parte da normalização para ganhar rapidez de leitura (analisar em cada contexto)

- colunas computadas e agregados pré-computados
  - coluna calculada (computed/generated column) no próprio registro
  - do ponto de vista de normalização não é adequado, mas ganha performance
  - desvantagem: custo extra na escrita do registro

```sql
-- adiciona uma coluna calculada (o valor dela é um select)
ALTER TABLE pedidos
  ADD COLUMN valor_total NUMERIC(12,2)
GENERATED ALWAYS AS (
  SELECT SUM(quantidade * preco_unit)
  FROM itens_pedido i WHERE i.pedido_id = pedidos.pedido_id
) STORED;
```

- tabelas de resumo (summary tables)
  - tabela dedicada que armazena agregados
  - utilizada em relatórios

```sql
-- essa tabela é apenas para armazenar agregados, ve de outra tabela (select)
CREATE TABLE resumo_vendas_mensal (
  ano INT,
  mes INT,
  valor_venda NUMERIC(14,2),
  PRIMARY KEY (ano, mes)
)

INSERT INTO resumo_vendas_mensal (ano, mes, valor_venda)
SELECT
  EXTRACT(YEAR FROM p.data_pedido)::INT AS ano,
  EXTRACT(MONTH FROM p.data_pedido)::INT AS mes,
  SUM(i.quantidade * i.preco_unit)
FROM pedidos p
JOIN itens_pedido i ON i.pedido_id = p.pedido_id
GROUP BY 1,2;
```

- definir sempre critérios para definir normalização e denormalização (estratégias) mas **não** perder as ideias da normalização

- ferramentas de sincronização
  - triggers para updates, inserts, deletes e tabelas fonte;
  - jobs periódicos (cron, pg_cron, ferramentas de ETL): para refresh de materialized views o rebuild de sumarry tables durante janelas de baixa carga
  - streaming/cdc (logical replication, debezium): manter réplicas analíticas quase em tempo real
