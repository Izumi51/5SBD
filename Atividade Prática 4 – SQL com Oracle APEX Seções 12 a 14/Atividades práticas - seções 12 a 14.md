# Lista de Exercícios – SQL com Oracle APEX

**Seções 12 a 14 – Oracle Academy: Database Programming with SQL**
**Tema:** Conversões, Tipos de Dados Avançados e Constraints.
**Ferramenta:** Oracle APEX – SQL Workshop
**Base de dados utilizada:** Sistema bancário com tabelas `agencia`, `cliente`, `conta`, `emprestimo` (e novas tabelas `cartao_credito`, `transacao`).

---

## Parte 1 – Seção 12: Conversões Implícitas e Explícitas

**Objetivo:** Aplicar funções de conversão de dados, como `TO_CHAR`, para formatar números e datas, e realizar concatenação de strings.

1.  Exiba o número da conta e o saldo formatado com separador de milhar e vírgula como separador decimal.

    ```sql
    SELECT
        conta_numero,
        TO_CHAR(saldo, '999G999G990D00', 'NLS_NUMERIC_CHARACTERS = '',.''') AS saldo_formatado
    FROM conta;
    ```

2.  Mostre a data atual no formato 'DD/MM/YYYY'.

    ```sql
    SELECT TO_CHAR(SYSDATE, 'DD/MM/YYYY') AS data_atual
    FROM dual;
    ```

3.  Exiba o nome do cliente concatenado com a cidade onde ele reside.

    ```sql
    SELECT cliente_nome || ' (' || cidade || ')' AS cliente_localidade
    FROM cliente;
    ```

4.  Liste os empréstimos com valor superior a R$ 5000, formatando o valor com símbolo monetário.

    ```sql
    SELECT
        emprestimo_cod,
        TO_CHAR(valor, 'L99G999D00', 'NLS_CURRENCY = ''R$''') AS valor_formatado
    FROM emprestimo
    WHERE valor > 5000;
    ```

---

## Parte 2 – Seção 13: Tipos de Dados Avançados

**Objetivo:** Trabalhar com os tipos de dados `TIMESTAMP` e `INTERVAL`, e realizar operações com datas.

5.  Adicione uma coluna do tipo `TIMESTAMP` na tabela `cliente` chamada `data_cadastro`.

    ```sql
    ALTER TABLE cliente
    ADD data_cadastro TIMESTAMP;
    ```

6.  Mostre quantos dias se passaram desde o cadastro de cada cliente.

    ```sql
    SELECT
        cliente_nome,
        TRUNC(SYSDATE - CAST(data_cadastro AS DATE)) AS dias_desde_cadastro
    FROM cliente;
    ```

7.  Adicione uma coluna do tipo `INTERVAL YEAR TO MONTH` na tabela `cliente` chamada `tempo_fidelidade`.

    ```sql
    ALTER TABLE cliente
    ADD tempo_fidelidade INTERVAL YEAR TO MONTH;
    ```

8.  Exiba para cada cliente a data de cadastro e uma previsão de renovação de cadastro adicionando 3 meses.

    ```sql
    SELECT
        cliente_nome,
        data_cadastro,
        ADD_MONTHS(data_cadastro, 3) AS previsao_renovacao
    FROM cliente;
    ```

---

## Parte 3 – Seção 14: Constraints

**Objetivo:** Criar e gerenciar tabelas com restrições (constraints) como `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL` e `CHECK`.

9.  Crie uma tabela chamada `cartao_credito` com as seguintes restrições:
    * `cartao_numero` (PK)
    * `cliente_cod` (FK)
    * `limite_credito` (NOT NULL)
    * `status` (CHECK: 'ATIVO', 'BLOQUEADO', 'CANCELADO')

    ```sql
    CREATE TABLE cartao_credito (
        cartao_numero   NUMBER(16) PRIMARY KEY,
        cliente_cod     NUMBER,
        limite_credito  NUMBER(10, 2) NOT NULL,
        status          VARCHAR2(10) CHECK (status IN ('ATIVO', 'BLOQUEADO', 'CANCELADO')),
        
        CONSTRAINT fk_cliente_cartao
            FOREIGN KEY (cliente_cod)
            REFERENCES cliente(cliente_cod)
    );
    ```

10. Tente inserir um cartão de crédito com campo `limite_credito` nulo (explique o erro).

    ```sql
    -- Tentativa de inserção
    INSERT INTO cartao_credito (cartao_numero, cliente_cod, limite_credito, status)
    VALUES (1234567890123456, 1, NULL, 'ATIVO');

    -- Explicação do Erro:
    -- O erro ocorre porque a coluna 'limite_credito' foi definida com a
    -- restrição NOT NULL, que proíbe explicitamente a inserção de valores nulos.
    ```

11. Insira ao menos três registros válidos na tabela `cartao_credito`.

    ```sql
    INSERT INTO cartao_credito (cartao_numero, cliente_cod, limite_credito, status)
    VALUES (1111222233334444, 1, 5000.00, 'ATIVO');
    
    INSERT INTO cartao_credito (cartao_numero, cliente_cod, limite_credito, status)
    VALUES (5555666677778888, 2, 2500.00, 'BLOQUEADO');
    
    INSERT INTO cartao_credito (cartao_numero, cliente_cod, limite_credito, status)
    VALUES (9999888877776666, 3, 10000.00, 'ATIVO');
    ```

12. Crie uma tabela `transacao` com restrição `CHECK` para valores positivos.

    ```sql
    CREATE TABLE transacao (
        transacao_id    NUMBER PRIMARY KEY,
        cartao_numero   NUMBER(16) REFERENCES cartao_credito(cartao_numero),
        valor           NUMBER(10, 2),
        data_transacao  DATE DEFAULT SYSDATE,
        
        CONSTRAINT chk_valor_positivo
            CHECK (valor > 0)
    );
    ```

13. Tente inserir uma transação com valor negativo (explique o erro).

    ```sql
    INSERT INTO transacao (transacao_id, cartao_numero, valor)
    VALUES (1001, 1111222233334444, -50.00);

    -- O erro ocorre porque o valor -50.00 viola a restrição CHECK 'chk_valor_positivo',
    -- que exige que todos os valores inseridos na coluna 'valor' sejam maiores que zero.
    ```

14. Crie uma consulta que relacione clientes ativos com seus respectivos limites de crédito acima de 3000.

    ```sql
    SELECT
        c.cliente_nome,
        cc.cartao_numero,
        cc.limite_credito,
        cc.status
    FROM cliente c
    JOIN cartao_credito cc ON c.cliente_cod = cc.cliente_cod
    WHERE
        cc.status = 'ATIVO'
        AND cc.limite_credito > 3000;
    ```

15. Crie uma `VIEW` chamada `vw_clientes_com_cartao` que exiba nome, cidade e status do cartão.

    ```sql
    CREATE OR REPLACE VIEW vw_clientes_com_cartao AS
    SELECT
        c.cliente_nome,
        c.cidade,
        cc.status AS status_cartao,
        cc.limite_credito
    FROM cliente c
    JOIN cartao_credito cc ON c.cliente_cod = cc.cliente_cod;