# Atividade Prática – SQL com Oracle APEX

**Seções 7 a 9 – Oracle Academy: Database Programming with SQL**
**Tema:** Junções, funções de agregação, agrupamentos e operadores de conjunto.
**Ferramenta:** Oracle APEX – SQL Workshop
**Base de dados utilizada:** Sistema bancário com tabelas `agencia`, `cliente`, `conta`, `emprestimo`.

---

## Parte 1 – Junções e Produto Cartesiano (Seção 7)

**Objetivo:** Compreender e aplicar junções entre tabelas, além de trabalhar com o conceito de produto cartesiano.

1. Exiba o nome de cada cliente junto com o número de sua conta.

   ```sql
       SELECT c.cliente_nome, co.conta_numero
       FROM cliente c
       JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod;
   ```

2. Mostre todas as combinações possíveis de clientes e agências (produto cartesiano).

   ```sql
       SELECT c.cliente_nome, a.agencia_nome
       FROM cliente c, agencia a;
   ```

3. Usando aliases de tabela, exiba o nome dos clientes e a cidade da agência onde mantêm conta.

   ```sql
       SELECT c.cliente_nome, a.agencia_cidade
       FROM cliente c
       JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod
       JOIN agencia a ON co.agencia_agencia_cod = a.agencia_cod;
   ```

---

## Parte 2 – Funções de Grupo, COUNT, DISTINCT e NVL (Seção 8)

**Objetivo:** Utilizar funções de agregação, como `COUNT`, `SUM`, `AVG`, e `NVL` para realizar cálculos e tratativas de dados.

4. Exiba o saldo total de todas as contas cadastradas.

   ```sql
       SELECT SUM(saldo) AS saldo_total
       FROM conta;
   ```

5. Mostre o maior saldo e a média de saldo entre todas as contas.

   ```sql
       SELECT MAX(saldo) AS maior_saldo, AVG(saldo) AS media_saldo
       FROM conta;
   ```

6. Apresente a quantidade total de contas cadastradas.

   ```sql
       SELECT COUNT(*) AS total_contas
       FROM conta;
   ```

7. Liste o número de cidades distintas onde os clientes residem.

   ```sql
       SELECT COUNT(DISTINCT cidade) AS total_cidades_distintas
       FROM cliente;
   ```

8. Exiba o número da conta e o saldo, substituindo valores nulos por zero.

   ```sql
       SELECT conta_numero, NVL(saldo, 0) AS saldo
       FROM conta;
   ```

---

## Parte 3 – GROUP BY, HAVING, ROLLUP e Operadores de Conjunto (Seção 9)

**Objetivo:** Aprender a utilizar agrupamentos com `GROUP BY`, filtragem com `HAVING`, e a cláusula `ROLLUP` para análises mais detalhadas. Além disso, explorar operadores de conjunto como `UNION`.

9. Exiba a média de saldo por cidade dos clientes.

   ```sql
       SELECT c.cidade, AVG(co.saldo) AS media_saldo
       FROM cliente c
       JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod
       GROUP BY c.cidade;
   ```

10. Liste apenas as cidades com mais de 3 contas associadas a seus moradores.

    ```sql
        -- nao tem nenhuma cidade com mais de 3 contas
        SELECT c.cidade, COUNT(co.conta_numero) AS total_contas
        FROM cliente c
        JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod
        GROUP BY c.cidade
        HAVING COUNT(co.conta_numero) > 3;
    ```

11. Utilize a cláusula ROLLUP para exibir o total de saldos por cidade da agência e o total geral.

    ```sql
        SELECT a.agencia_cidade, SUM(co.saldo) AS total_saldo
        FROM agencia a
        JOIN conta co ON a.agencia_cod = co.agencia_agencia_cod
        GROUP BY ROLLUP(a.agencia_cidade);
    ```

12. Faça uma consulta com `UNION` que combine os nomes de cidades dos clientes e das agências, sem repetições.

    ```sql
        SELECT cidade FROM cliente
        UNION
        SELECT agencia_cidade FROM agencia;
    ```

---

## Atividade Prática – SQL com Oracle APEX

**Seção 10 – Subconsultas**
**Tema:** Subconsultas de linha única, multilinha, correlacionadas, com `EXISTS`, `NOT EXISTS` e a cláusula `WITH`.
**Ferramenta:** Oracle APEX – SQL Workshop
**Base de dados utilizada:** Sistema bancário com tabelas `agencia`, `cliente`, `conta`, `emprestimo`.

---

## Parte 1 – Subconsultas de Linha Única

**Objetivo:** Trabalhar com subconsultas simples, que retornam um único valor.

1. Liste os nomes dos clientes cujas contas possuem saldo acima da média geral de todas as contas registradas.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       WHERE c.cliente_cod IN (
           SELECT cliente_cliente_cod
           FROM conta
           WHERE saldo > (SELECT AVG(saldo) FROM conta)
       );
   ```

2. Exiba os nomes dos clientes cujos saldos são iguais ao maior saldo encontrado no banco.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       WHERE c.cliente_cod IN (
           SELECT cliente_cliente_cod
           FROM conta
           WHERE saldo = (SELECT MAX(saldo) FROM conta)
       );
   ```

3. Liste as cidades onde a quantidade de clientes é maior que a quantidade média de clientes por cidade.

   ```sql
       -- toda cidade tem a mesma quantidade de clientes
       SELECT cidade
       FROM cliente c
       GROUP BY cidade
       HAVING COUNT(*) > (
           SELECT AVG(qtd) 
           FROM (
               SELECT cidade, COUNT(*) AS qtd
               FROM cliente
               GROUP BY cidade
           )
       );
   ```

---

## Parte 2 – Subconsultas Multilinha

**Objetivo:** Trabalhar com subconsultas que retornam múltiplos valores.

4. Liste os nomes dos clientes com saldo igual a qualquer um dos dez maiores saldos registrados.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       WHERE c.cliente_cod IN (
           SELECT cliente_cliente_cod
           FROM conta
           ORDER BY saldo DESC
           FETCH FIRST 10 ROWS ONLY
       );
   ```

5. Liste os clientes que possuem saldo menor que todos os saldos dos clientes da cidade de Niterói.

   ```sql
        SELECT c.cliente_nome, co.saldo
        FROM cliente c
        JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod
        WHERE co.saldo < ALL (
            SELECT co2.saldo
            FROM cliente c2
            JOIN conta co2 ON c2.cliente_cod = co2.cliente_cliente_cod
            WHERE c2.cidade = 'Niterói'
        );
   ```

6. Liste os clientes cujos saldos estão entre os saldos de clientes de Volta Redonda.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       JOIN conta co ON c.cliente_cod = co.cliente_cliente_cod
       WHERE co.saldo BETWEEN (
           SELECT MIN(saldo)
           FROM cliente c2
           JOIN conta co2 ON c2.cliente_cod = co2.cliente_cliente_cod
           WHERE c2.cidade = 'Volta Redonda'
       ) AND (
           SELECT MAX(saldo)
           FROM cliente c2
           JOIN conta co2 ON c2.cliente_cod = co2.cliente_cliente_cod
           WHERE c2.cidade = 'Volta Redonda'
       );
   ```

---

## Parte 3 – Subconsultas Correlacionadas

**Objetivo:** Explorar subconsultas que dependem da linha externa para avaliação.

7. Exiba os nomes dos clientes cujos saldos são maiores que a média de saldo das contas da mesma agência.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       WHERE EXISTS (
           SELECT 1
           FROM conta co
           WHERE co.cliente_cliente_cod = c.cliente_cod
           AND co.saldo > (
               SELECT AVG(saldo)
               FROM conta co2
               WHERE co2.agencia_agencia_cod = co.agencia_agencia_cod
           )
       );
   ```

8. Liste os nomes e cidades dos clientes que têm saldo inferior à média de sua própria cidade.

   ```sql
        SELECT c.cliente_nome,
        c.cidade,
        co.saldo
        FROM cliente c
        JOIN conta co 
            ON c.cliente_cod = co.cliente_cliente_cod
        WHERE co.saldo < (
            SELECT AVG(co2.saldo)
            FROM cliente c2
            JOIN conta co2 
                ON c2.cliente_cod = co2.cliente_cliente_cod
            WHERE c2.cidade = c.cidade
        );
   ```

---

## Parte 4 – Subconsultas com EXISTS e NOT EXISTS

**Objetivo:** Trabalhar com as cláusulas `EXISTS` e `NOT EXISTS`.

9. Liste os nomes dos clientes que possuem pelo menos uma conta registrada no banco.

   ```sql
       SELECT cliente_nome
       FROM cliente c
       WHERE EXISTS (
           SELECT 1
           FROM conta co
           WHERE co.cliente_cliente_cod = c.cliente_cod
       );
   ```

10. Liste os nomes dos clientes que ainda não possuem conta registrada no banco.

    ```sql
        SELECT c.cliente_nome
        FROM cliente c
        WHERE NOT EXISTS (
            SELECT 1
            FROM conta co
            WHERE co.cliente_cliente_cod = c.cliente_cod
        );
    ```