---
name: sqlserver-stored-procedure-padrao
description: Criar, revisar ou padronizar stored procedures SQL Server no estilo usado neste projeto. Use quando a tarefa envolver procedures, scripts de arquitetura SQL, padroes SPI/SPC/SPA/SP, retrocompatibilidade com SQL Server 2008, retorno ST/MSG, HRSplit, tratamento de NULL com ISNULL, datas default 1900-01-01, controle transacional com TRY/CATCH e formatacao com virgulas a esquerda.
---

# SQL Server Stored Procedure Padrao

Siga este skill ao criar ou revisar procedures SQL Server deste projeto.

## Fluxo

1. Ler primeiro [references/padroes-procedures.md](references/padroes-procedures.md).
2. Identificar o tipo da procedure:
   - `SPI`: inicializacao/insercao de controle
   - `SPC`: consulta/criacao de contexto para processamento
   - `SPA`: atualizacao/conclusao/interrupcao/processamento
   - `SP`: inclusao, alteracao, upsert ou carga final de entidade de negocio
3. Preservar compatibilidade com SQL Server 2008.
4. Preferir scripts com `IF OBJECT_ID(..., 'P') IS NOT NULL DROP PROCEDURE ...` seguido de `CREATE PROCEDURE`.
5. Manter nomes, schema e estilo ja adotados no projeto antes de propor qualquer variacao.

## Regras de execucao

- Usar virgulas a esquerda em parametros, colunas, `SELECT`, `INSERT` e `UPDATE`.
- Em `SELECT INTO`, manter o `INTO #TABELA` logo apos a lista de campos, alinhado no bloco do `SELECT`.
- Manter `INNER JOIN`, `LEFT JOIN`, `CROSS APPLY` e operadores equivalentes alinhados na mesma linha da tabela ou subconsulta anterior.
- Usar `ISNULL` para campos retornados em `SELECT` quando o consumo legado puder falhar com `NULL`.
- Usar data padrao `19000101` ao expor datas default em consultas.
- Em procedures de gravacao, preferir retorno centralizado `ST/MSG` para validacoes esperadas.
- Usar `SET NOCOUNT ON`.
- Quando houver escrita concorrente em controle, considerar `WITH (UPDLOCK, HOLDLOCK)` no padrao ja adotado.
- Envolver escrita em `BEGIN TRY / BEGIN CATCH`, com rollback quando `XACT_STATE() <> 0`.
- Para excecoes inesperadas, retornar `ST < 0` e `MSG = ERROR_MESSAGE()` quando a procedure seguir o padrao `ST/MSG`; repropagar com `RAISERROR` apenas em rotinas que nao usam esse padrao.
- Evitar recursos nao compativeis com SQL Server 2008.

## Saida esperada

Ao responder ou editar:

- entregar procedure pronta no padrao do projeto
- manter comentarios objetivos em SQL, sem excesso
- explicar rapidamente qualquer decisao que fuja do padrao
- quando houver mais de uma procedure relacionada, avaliar se faz sentido consolidar em um unico script

## Referencia

Para regras detalhadas e exemplos de estrutura, ler [references/padroes-procedures.md](references/padroes-procedures.md).
