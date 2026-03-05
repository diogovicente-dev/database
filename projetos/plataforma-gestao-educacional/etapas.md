# Etapas para criação da modelagem

## Modelo conceitual

1. Mapeamento das principais entidades e seus atributos principais;

- no modelo conceitual não 'detalhar' os atributos multivalorados
- mapear colocando as PK's de cada entidade

2. Mapeamento dos relacionamentos entre as entidades;

- definição das cardinalidades
- levantar se existe algum atributo derivado do relacionamento
  - exemplo: data de admissão do professor só existe com a relação de trabalho com a escola;
  - exemplo2: turma e disciplina, com atributos de data de início e término da disciplina;

## Modelo lógico

**Aplicação das Formas Normais - 1FN, 2FN e 3FN**

1. Transformação de entidades para tabelas - regra - cada entidade é convertida em uma tabela

- determinação dos atributos primeiramente (pode ser que nos atributos devam ser criadas novas tabelas)
- exemplo: endereço e telefone (depende do modelo de negócio, se telefone e endereço for mais de um, cria-se uma tabela nova),
- se mais de uma entidade, em teoria, usar uma tabela 'igual', o ideal é ter uma tabela para cada entidade, para que a tabela 'endereco', por exemplo, não fique com 2, 3, 4 chaves estrangeiras;
- não que uma tabela não possa ter 2, 3 FK's, mas neste caso 'nenhuma' FK seria obrigatória, pois poderia ser de aluno, escola ou professor

2. Mapear atributos multivalorados e transformá-los em novas tabelas, se aplicável

- criar novos relacionamentos, se aplicável
- ou detalhar o atributo multivalorado em atributos simples 'menores'
- exemplo: o atributo `endereco` => vira `rua`, `cep`, `cidade` e `estado`

3. Mapear atributos 'candidatos' a PK, se aplicável, e criar nova tabela

4. Determinação das FK's nas tabelas, conforme relacionamentos;

- 1:N => chave estrangeira do lado 'N';
- 1:1 => chave estrangeira em qualquer um dos lados;
- N:N => cria-se uma nova 'tabela de relacionamento' que junta as duas

5. Atributos dos relacionamentos

- se N:N, fica na nova tabela;
- se 1:N, fica na tabela 'N';
- se 1:1, fica junto com a que ficou com a FK;

6. Tipagens

- respeitar as tipagens principalmente das FK's
- se a PK referenciada na FK for `serial`, na tabela tem que ser `int`, por exemplo
- respeitar os tamanhos criados também; se a PK for `VARCHAR(20)`, a FK tem que ser `VARCHAR(20)`

## Modelo Físico

1. Criação das constraints e regras (NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, ... )

2. Criar primeiro as tabelas que não possuem FK's

3. Criar as tabelas que possuem as FK's
