---
name: vfp9-hrfoxbin2prg
description: Analisar e alterar forms, classes, projetos, reports e outros artefatos do Visual FoxPro 9 (VFP9), usando o projeto local HRFoxBin2prg para converter binarios em texto e reconstruir o formato original apos alteracoes.
---

# VFP9 com HRFoxBin2prg

Sempre que analisar ou alterar artefatos do VFP9, usar o projeto `C:\Desenvolve\HRFoxBin2prg` como ferramenta de conversao.

## Fluxo obrigatorio

1. Identificar os arquivos solicitados e seus arquivos acompanhantes (por exemplo, SCX/SCT, VCX/VCT, FRX/FRT e PJX/PJT). Preservar as alteracoes existentes do usuario e uma copia recuperavel dos originais antes de reconstruir binarios.
2. Consultar `LEIAME_HR.md`, `README.md` e a configuracao aplicavel no projeto HRFoxBin2prg para determinar a chamada e as extensoes corretas. Usar o conversor desse projeto; nao substituir por edicao direta das tabelas binarias.
3. Converter os arquivos binarios solicitados para texto antes da analise. Trabalhar sobre o texto gerado a partir da versao atual dos arquivos, evitando sobrescrever textos com alteracoes ainda nao reconvertidas.
4. Quando houver uma alteracao solicitada, editar a representacao textual, preservando a estrutura do conversor, a codificacao e as quebras de linha. O LEIAME_HR informa cp1252 e CRLF para os PRGs locais; verificar os arquivos de destino antes de grava-los.
5. Converter o texto alterado de volta ao formato original com o mesmo projeto e manter os arquivos acompanhantes consistentes. A entrega de uma alteracao em artefato binario deve incluir a reconversao, nao apenas o texto editado.
6. Verificar os logs, a existencia dos arquivos reconstruidos e, quando viavel, reconverter o resultado para texto em local separado para conferir se as alteracoes foram preservadas. Informar os arquivos alterados e os limites da validacao; conversao bem-sucedida nao comprova execucao funcional no VFP9.

Em pedidos somente de analise, converter e inspecionar o texto sem modificar nem reconstruir os originais desnecessariamente. Arquivos que ja sao texto, como PRG e H, podem ser lidos e editados diretamente; nao inventar uma etapa de conversao para formatos nao suportados.

## Ferramentas locais

- Conversor: `C:\Desenvolve\HRFoxBin2prg\foxbin2prg.exe` e `foxbin2prg.prg`.
- Entrada binario para texto: `Convert_VFP9_BIN_2_PRG.vbs`.
- Entrada texto para binario: `Convert_VFP9_PRG_2_BIN.vbs`.
- Os VBS usam `VisualFoxPro.Application.9` e podem exibir janelas e mensagens. Inspecionar as chamadas antes de automatizar; preferir uma invocacao documentada adequada ao ambiente.
- Exemplo documentado no VFP: `DO C:\Desenvolve\HRFoxBin2prg\foxbin2prg.prg WITH "<caminho>\arquivo.scx"` gera SC2; a mesma chamada com SC2 reconstroi SCX. As extensoes sao configuraveis: consultar a configuracao em vez de assumir nomes.

Limitar a conversao aos arquivos necessarios para o pedido. Se o projeto ou o runtime estiver indisponivel, informar o impedimento e nao afirmar que a conversao ou reconversao ocorreu.
