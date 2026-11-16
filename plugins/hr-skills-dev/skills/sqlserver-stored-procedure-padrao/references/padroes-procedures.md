# Padroes de Procedures

## Objetivo

Aplicar o padrao de stored procedures SQL Server usado neste projeto, em especial nas rotinas de `HRCTE`.

## Convencoes de nome

- `SPI_`: inicializa ou garante existencia de registro de controle
- `SPC_`: consulta, cria contexto de processamento ou retorna dados para a aplicacao
- `SPA_`: atualiza, conclui, interrompe ou processa etapas intermediarias
- `SP_`: inclui, altera, faz upsert, grava entidade final ou executa carga de negocio

## Compatibilidade

- Manter retrocompatibilidade com SQL Server 2008.
- Nao usar `THROW`, `CONCAT`, `IIF`, `FORMAT`, `SEQUENCE`, `MERGE` ou sintaxes posteriores sem necessidade extrema.
- Preferir `RAISERROR` apenas em rotinas que nao usam retorno `ST/MSG`.

## Formatacao

- Usar virgulas a esquerda:

```sql
SELECT
      CAMPO_A = T.CAMPO_A
    , CAMPO_B = ISNULL(T.CAMPO_B, '')
FROM
    TABELA T
WHERE
        T.ID = @ID
```

- Em `SELECT INTO`, manter o `INTO #TABELA` logo apos a lista de campos, alinhado no bloco do `SELECT`:

```sql
SELECT
      T.*
    , CAMPO_NOVO = CAST(0 AS NUMERIC(14,3))
    INTO #TEMP
FROM
    TABELA T
```

- Manter `INNER JOIN`, `LEFT JOIN`, `CROSS APPLY` e operadores equivalentes alinhados na mesma linha da tabela ou subconsulta anterior:

```sql
FROM
    TABELA_A A
    INNER JOIN TABELA_B B
        ON B.ID = A.ID
    CROSS APPLY
    (
        SELECT TOP 1
              C.CAMPO
        FROM
            TABELA_C C
        WHERE
            C.ID = A.ID
    ) C
```

- Indentar blocos `BEGIN/END`.
- Manter `GO` entre procedures em scripts consolidados.
- Em scripts de publicacao, usar `DROP` seguido de `CREATE`, nao `ALTER`, para agilizar atualizacao manual em banco.

## Validacoes e retorno

- Validar obrigatoriedade de parametros logo no inicio.
- Validar valores de dominio, por exemplo ambiente `IN (1, 2)`.
- Normalizar entradas quando necessario com `LTRIM`, `RTRIM`, `RIGHT`, `REPLICATE`.
- Quando a procedure de gravacao seguir o padrao de retorno do projeto, usar `ST` e `MSG` em vez de `RAISERROR` para validacoes esperadas.
- Usar `ST = 0` para sucesso, `ST < 0` para excecoes inesperadas e `ST > 0` para avisos/validacoes que devem ser exibidos no frontend.
- Centralizar o retorno em `FIM:` e direcionar validacoes com `GOTO FIM`.

Exemplo:

```sql
DECLARE @ST  INT
DECLARE @MSG VARCHAR(4000)

SET @ST = 0
SET @MSG = 'Processamento realizado com sucesso.'

IF LTRIM(RTRIM(ISNULL(@CNPJ, ''))) = ''
    BEGIN
        SET @ST = 1
        SET @MSG = 'CNPJ nao informado.'
        GOTO FIM
    END

FIM:
    SELECT
          ST  = @ST
        , MSG = @MSG
```

## NULL e valores default

- Preferir `ISNULL` em retornos consumidos pelo legado.
- Para datas retornadas em consulta, usar `19000101`.
- Para strings de controle, devolver valor padrao consistente em vez de `NULL` quando o legado exigir.

Exemplo:

```sql
SELECT
      DATA_CONSULTA = ISNULL(T.DATA_CONSULTA, '19000101')
    , MAXIMO_NSU    = ISNULL(T.MAXIMO_NSU, '000000000000000')
FROM
    ALGUMA_TABELA T
```

## Controle transacional

- Usar `SET NOCOUNT ON`.
- Em escrita, usar `BEGIN TRY / BEGIN CATCH`.
- Iniciar transacao apenas quando necessario.
- Fazer validacoes de parametros, dominios, listas e temporarias antes do `BEGIN TRAN`, quando nao houver necessidade de bloqueio transacional.
- Manter dentro da transacao apenas `INSERT`, `UPDATE`, `DELETE` e leituras concorrentes necessarias para proteger a escrita.
- Em `CATCH`, fazer rollback quando `XACT_STATE() <> 0`.
- Em procedures com retorno `ST/MSG`, excecoes inesperadas devem retornar `ST < 0` e `MSG = ERROR_MESSAGE()`.
- Repropagar o erro original com severidade e estado apenas em rotinas que nao usam o padrao `ST/MSG`.

Modelo com `ST/MSG`:

```sql
DECLARE @MENSAGEM_ERRO   VARCHAR(4000)
DECLARE @ST              INT
DECLARE @MSG             VARCHAR(4000)

SET @ST = 0
SET @MSG = 'Processamento realizado com sucesso.'

BEGIN TRY
    BEGIN
        -- validacoes e preparacao fora da transacao

        BEGIN TRAN

        -- processamento

        COMMIT TRAN
    END
END TRY
BEGIN CATCH
    BEGIN
        IF XACT_STATE() <> 0
            BEGIN
                ROLLBACK TRAN
            END

        SET @MENSAGEM_ERRO = ERROR_MESSAGE()

        SET @ST = -1
        SET @MSG = @MENSAGEM_ERRO
        GOTO FIM
    END
END CATCH

FIM:
    SELECT
          ST  = @ST
        , MSG = @MSG
```

## Concorrencia

- Em tabelas de controle como ultimo NSU, considerar `WITH (UPDLOCK, HOLDLOCK)` para evitar corrida entre consultas simultaneas.
- Ler o estado atual dentro da mesma transacao da atualizacao.
- Em upserts em lote, atualizar apenas quando algum valor realmente mudou para reduzir locks, escrita e tempo de execucao.

## Chaves e parametros

- Toda tabela pode usar `ID` como surrogate key.
- Chaves estrangeiras devem seguir `ID_NOME_TABELA`.
- Nao exigir `ID` como parametro de gravacao quando existir chave funcional, chave natural ou registro singleton.
- Em tabelas singleton, verificar existencia do registro para decidir entre `INSERT` e `UPDATE`.
- Em tabelas com chave funcional, fazer upsert pela chave de negocio validada.

## Parametros em lote com HRSplit

- Para listas delimitadas por `|`, usar `DBO.HRSplit(@PARAMETRO, '|')`.
- A funcao retorna colunas `ID` e `VALOR`; relacionar listas pela coluna `ID`.
- Usar `INNER JOIN` quando todas as listas forem obrigatorias e cada posicao precisar existir em todas elas.
- Materializar cada split uma unica vez em tabela temporaria e reutilizar nas validacoes.
- Evitar chamar `HRSplit` repetidas vezes para a mesma lista.
- Validar que todas as listas possuem a mesma quantidade de itens antes de gravar.
- Montar uma tabela temporaria final para o upsert depois de validar quantidade, tipos e dominios.

Exemplo:

```sql
SELECT
      ID_LINHA = S.ID
    , VALOR    = LTRIM(RTRIM(ISNULL(S.VALOR, '')))
    INTO #CAMPO_A
FROM
    DBO.HRSplit(@CAMPO_A, '|') S

SELECT
      ID_LINHA = S.ID
    , VALOR    = LTRIM(RTRIM(ISNULL(S.VALOR, '')))
    INTO #CAMPO_B
FROM
    DBO.HRSplit(@CAMPO_B, '|') S

SELECT
      A.ID_LINHA
    , CAMPO_A = A.VALOR
    , CAMPO_B = B.VALOR
    INTO #DADOS
FROM
    #CAMPO_A A
    INNER JOIN #CAMPO_B B
        ON B.ID_LINHA = A.ID_LINHA
```

## Regras especificas deste projeto

- Respeitar a resposta oficial da API ao atualizar ultimo NSU.
- Nao avancar NSU em processamento manual de arquivo apenas para "sincronizar" banco.
- Em consultas, devolver campos sem `NULL` quando o legado puder falhar com cursores VPF.
- Ao trabalhar com scripts da pasta `docs\Script SQL`, manter coerencia entre `Arquitetura.sql` e `Procedures.sql`.

## Checklist final

- Nome da procedure segue SPI/SPC/SPA/SP?
- Script usa `DROP` + `CREATE`?
- Compativel com SQL Server 2008?
- Validacoes iniciais presentes?
- Procedures de gravacao usam `ST/MSG` quando este for o padrao do projeto?
- Retorno esta centralizado em `FIM:` quando usa `ST/MSG`?
- `SET NOCOUNT ON` presente?
- `ISNULL` aplicado onde o legado exige?
- Datas default em `19000101` quando retornadas?
- `TRY/CATCH` e rollback corretos?
- Transacao comeca apenas no trecho necessario de escrita?
- Parametros em lote com `HRSplit` sao materializados uma unica vez?
- Virgulas a esquerda em todo o script?
