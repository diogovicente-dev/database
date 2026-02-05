# Manipulação básica de dados

## Operações básicas

- criação de tabelas simples (sem relacionamento com outras tabelas)

- boas práticas
  -- estrutura clara e consistente
  --- se tenho clientes, preciso ter uma tabela clientes
  -- tipos corretos para cada campo
  --- tipos de dados do postgre, definição dos tipos de dados para cada situação
  -- restrições bem aplicadas
  --- não permitir dado diferente do configurado, dado 'null', entre outras regras
  -- facilidade de manutenção e expansão

## Estrutura do comando CREATE TABLE

- comando `CREATE`
- comando `DROP` apaga toda a tabela e suas informações

- `serial`=> cada registro criado incrementa automaticamente e de forma crescente
- `primary key` => chave primária = identificador único da tabela
- boa prática: semrpe usar os comandos em maiúsculo
- manter padrão nas tabelas quanto aos nomes das colunas (snake, underline, etc..)

```sql
CREATE TABLE clientes (
  id SERIAL PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  email VARCHAR(150) UNIQUE NOT NULL,
  data_nascimento DATE,
  saldo NUMERIC(10,2),
  ativo BOOLEAN DEFAULT TRUE,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- no processo de modelagem de banco de dados entenderemos como os dados serão tratados (null, not null, date, numeric, etc..)

## Tipos de Dados no Postgre

- entender os tipos para definir corretamente na criação das tabelas
- principais utilizados no dia-a-dia

### Números

- `Integer`
  -- INT, INTEGER, SMALLINT, BIGINT
  -- números inteiros com diferentes tamanhos
  -- `SMALLINT` - números menores, até 32.767, ex.: idade
  -- `INTEGER` ou `INT` => valor padrão para inteiros ex.: quantidade de produtos
  -- `BIGINT` => números muito grandes, ex: população de um país

  -- exemplo de declaração:

  ```SQL
  CREATE TABLE exemplos_inteiros (
   id SERIAL PRIMARY KEY,
   idade SMALLINT,
   quantidade INT,
   populacao BIGINT
  )
  ```

- `Decimal`
  -- NUMERIC, DECIMAL, REAL, DOUBLE PRECISION
  -- números com ponto flutuante
  -- NUMERIC e DECIMAL são mais precisos
  --`NUMERIC(10,2)` => para valores monetários, por exemplo: salário
  --`DECIMAL(5,3)` => para percentuais precisos
  --`REAL` => para medições menos críticas (32bits)
  --`DOUBLE PRECISION` => para medições de ala escala (64bits) = geolocalização, números com muitas casas decimais

  -- exemplo declaração:

  ```sql
  CREATE TABLE exemplos_decimais (
  id SERIAL PRIMARY KEY,
  salario NUMERIC(10,2),
  percentual DECIMAL(5,3),
  temperatura REAL,
  distancia DOUBLE PRECISION
  )
  ```

### Textos

Principais tipos

- `CHAR(n)`: texto com tamanho fixo - campos que eu sei quantos caracteres vai ter (CPF, CNPJ, TELEFONE, etc..)
  -- exemplo UFs, `CHAR(2)`
- `VARCHAR(n)`: texto com tamanho variável mas com o limite (nomes, emails, descrições, observações)
  -- uso: 0 a 255 caracteres
- `TEXT`: texto com tamanho ilimitado (declaração, artigo, etc..)

```SQL
CREATE TABLE exemplos_textuais (
  id SERIAL PRIMARY KEY,
  sigla CHAR(2),
  nome VARCHAR(150),
  texto TEXT
)
```

### Boolean e Datas/Horas

- `BOOLEAN`: valores lógicos
  -- valores: TRUE, FALSE, ou NULL (em alguns cenários que possam ser)
  -- uso em verificações binárias (ativo, status, etc..)

```sql
CREATE TABLE exemplo_boolean (
  id SERIAL PRIMARY KEY,
  nome VARCHAR(150),
  ativo BOOLEAN
);
```

- Data e Hora
  -- Tipos:
  -- `DATE`: apenas a data (dia, mês e ano)
  --- exemplo: data de nascimento
  -- `TIME`: apenas o horário (hora, minuto, segundo)
  -- `TIMESTAMP`: data e hora (sem fuso horário)
  -- `TIMESTAMPTZ`: data e hora com fuso horário
  --- ideal para aplicações globais

```SQL
CREATE TABLE exemplos_data_hora (
  id SERIAL PRIMARY KEY,
  nascimento DATE,
  hora_abertura TIME,
  criado_em TIMESTAMP,
  evento_inicio TIMESTAMPTZ
);
```

### Serial e Dados UUID

- `SERIAL`: auto incremento (exemplo: id da tabela)
  -- numeração crescente a partir do "1"
  -- sequencial
  -- se deletar o registro, não 'preenche' o buraco na tabela
  -- geralmente (ou sempre) a `primary key`
- `UUID`: identificador único universal
  -- `Universally Unique Identifier`
  -- conjunto de caracteres e letras que representam um id (uma grande `string`)
  -- identificador único de 128bits
  -- id's únicos globais em diferentes bancos

```SQL
CREATE TABLE exemplo_serial (
  id SERIAL PRIMARY KEY,
);

CREATE TABLE exemplo_uuid (
  id UUID PRIMARY KEY,
);
```

### Array e JSON

- `ARRAY`: lista de valores
  -- lista de valores iguais (inteiros ou textos)
  -- [] = representa o array
- `JSON`: dados em formato 'json'
  -- JSON e JSONB
  -- JSON armazena texto simples
  -- JSONB armazena como binário otimizado (mais rápido para busca e filtragem)

```SQL
CREATE TABLE exemplo_array (
  id ID PRIMARY KEY,
  notas INTEGER[],
  tags TEXT[]
);

CREATE TABLE exemplo_json (
  id SERIA PRIMARY KEY,
  dados JSONB,
);
```

## Insert

- como os dados devem ser escritos, enviados
- comando INSERT para adicionar dados em uma tabela existente
- Sintaxe:

```SQL
INSERT INTO nome_tabela (coluna1, coluna2, ...)
VALUES (valor1, valor2, ...)
```

- Importante:
  -- ao criar a tabela, é essencial definir corretamente os tipos de dados em cada coluna
  --- se a coluna é número, enviar número; se é texto, enviar texto;
  -- isso garante integridade, desempenho e economia de espaço

- Exemplo:

```SQL
--- inserindo dados na tabela 'alunos'
INSERT INTO alunos (nome, idade, email)
VALUES ('João Silva', 21, 'joao@emmail.com');
```

- não preciso enviar o 'id' da chave primária;
- o tipo `SERIAL` já define isso automaticamente
- o banco já entende que é automático => `ID` ou `UUID`
- inserção de vários itens:

```sql
INSERT INTO alunos (nome, idade, email)
VALUES
  ('João Silva', 21, 'joao@emmail.com'),
  ('Maria Santos', 22, 'maria@emmail.com');
```

_Criação de Tabela_

```SQL
CREATE TABLE clientes_novo (
  id SERIAL PRIMARY KEY,
  codigo_uuid UUID DEFAULT uuid_generate_v4(), --- UUID automático (função)
  idade SMALLINT,
  quantidade_compras INTEGER,
  pontos_acumulados BIGINT,
  ticket_medio NUMERIC(10,2),
  desconto_medio DECIMAL(5,2),
  estado CHAR(2),
  nome VARCHAR(100),
  observacoes TEXT,
  ativo BOOLEAN,
  data_nascimento DATE,
  hora_cadastro TIME,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  atualizado_em TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
  notas INTEGER[],
  tags TEXT[],
  informacoes_extras JSONB
);
```

- comando para habilitar/aplicar a geração de uuid automático
- tem relação com a função `uuid_generate_v4()`

```SQL
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

## Select

- buscar informações armazenadas em uma ou mais tabelas
- Sintaxe simples (uma tabela):

```SQL
SELECT coluna1, coluna2 FROM nome_tabela;

--- exemplo
SELECT nome, idade FROM alunos;

--- trazendo todos os dados de uma vez (todas as colunas)
SELECT * FROM alunos;

--- filtros
SELECT coluna1, coluna2 FROM nome_tabela WHERE condicao ;
```

- usar `SELECT * FROM` com cautela, principalmente nas implementações;
- cada consulta buscar apenas o que eu preciso;

## Update

- Atualização dos registros
- Sintaxe:

```SQL
UPDATE nome_tabela
SET coluna1 = novo_valor1, coluna2 = novovalor2
WHERE condição --- para atualizar apenas um registro ---
```

- `where` => em qual condição o update será feito
- sem o where, todos registros serão atualizados
- precisa atualizar todos os valores de uma coluna? atualiza sem o `WHERE`
- geralmente vai usar o `where`
- Exemplo>

```SQL
UPDATE alunos
SET idade = 23
WHERE nome = 'Maria Santos';
```

- no `where` não usar filtro que possa existir valores iguais, geralmente usar os identificadores únicos
- `WHERE` tem mais operadores que serão descritos posteriormente

## Delete

- deletar o registro
- Sintaxe:

```SQL
DELETE FROM nome_tabela
WHERE condição
```

- o delete remove a 'linha inteira', ou seja, o registro completo;
- Exemplo:

```sql
DELETE FROM alunos
WHERE idade < 18;
```

- sem o `where` apaga tudo! todos os registros são apagados!
- `where` trabalhar com identificadores precisos (id, uuid), não trabalhar com registros que podem ter valores duplicados
