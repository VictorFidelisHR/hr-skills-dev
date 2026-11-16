---
name: desenvolvimento-vfp
description: Aplica o padrão pessoal de desenvolvimento do usuário ao criar, alterar ou revisar código Visual FoxPro (VFP), incluindo formulários, controles, métodos, variáveis, cursores e integração com banco de dados. Para conversão de objetos binários, combine com vfp9-hrfoxbin2prg.
---

# Desenvolvimento em Visual FoxPro

Baseado no documento pessoal “Padrão de Desenvolvimento”, de 36 páginas. Trate o cadastro de setores do HR Auto Atendimento como exemplo, sem impor seus campos ou regras de negócio a outros sistemas. Respeite o pedido atual e o escopo do projeto.

## Nomenclatura

- Use `snake_case` para arquivos, métodos, propriedades e controles. Como VFP é case insensitive, os underscores preservam a legibilidade quando a capitalização muda.
- Nomeie tabelas, cursores e campos em `MAIUSCULAS_COM_UNDERSCORE`.
- O documento não determina snake_case ou camelCase para variáveis: mantenha consistência no mesmo método, classe ou módulo.
- Prefixe variáveis pelo escopo e depois pelo tipo: local `l`, pública `p`; Character `c`, Numeric `n`, Date `d`, Datetime `t`, lógico `l`. Exemplos: `lcLoja`, `pnNumeroAleatorio`. Esses prefixos não incentivam criar variáveis públicas.
- Use nomes claros, evite abreviações desnecessárias e nomeie todos os controles que serão referenciados.
- Combine os prefixos abaixo com snake_case: `txt_descricao`, `cmd_salvar`, `cnt_detalhes`.

| Controle | Prefixo |
|---|---|
| Container | cnt |
| PageFrame | pgf |
| Page | pg |
| Grid | grd |
| Column | col |
| CommandButton | cmd |
| TextBox | txt |
| Label | lbl |
| OptionGroup | opt |
| ComboBox | cbo |
| CheckBox | chk |
| EditBox | edt |
| Image | img |

## Preparação

Para novas funcionalidades com interface, desenhe as telas e defina o fluxo antes de codificar. Nomeie os controles nessa etapa e derive a estrutura de dados dos campos e operações necessários. Uma correção pontual não exige redesenhar a aplicação.

Inspecione o código e os contratos existentes. Aplique o padrão ao trecho criado ou alterado sem renomear indiscriminadamente componentes ou quebrar referências. PRG já é texto. Para ler, editar ou reconstruir SCX/VCX e outros objetos binários, use a skill `vfp9-hrfoxbin2prg` deste plugin e preserve os arquivos associados.

## Comentários de autoria e solicitação nas rotinas

Ao criar um arquivo inteiramente novo (formulário, classe, PRG, relatório ou outro artefato VFP), dispense os comentários de autoria e solicitação descritos nesta seção, inclusive nos métodos e rotinas que compõem esse arquivo. Não pergunte autor e solicitação apenas para preencher esses comentários quando o trabalho envolver somente arquivos novos. Comentários funcionais úteis continuam permitidos. Essa dispensa não altera a regra de versão da tela nem o histórico na propriedade `Comment` do label.

As regras abaixo se aplicam ao código de arquivos já existentes. Em trabalhos que criem arquivos e também alterem arquivos existentes, aplique a dispensa apenas aos novos; criar uma nova representação textual por conversão de um binário existente não torna o artefato novo.

Antes de criar ou alterar código VFP em arquivos já existentes, pergunte ao usuário o nome do autor e o número da solicitação, caso ainda não tenham sido informados para o trabalho atual. Pergunte os dois dados juntos e aguarde a resposta antes de editar esse código; a inspeção pode continuar. Não deduza o autor pelo Git, pela conta ou por comentários antigos e não invente número de solicitação. Reutilize os dados já fornecidos para a mesma solicitação, sem perguntar novamente em cada rotina. Use a data local efetiva da alteração no formato `DD/MM/AAAA`, nunca a data do exemplo.

Em rotinas já existentes, delimite cada trecho alterado com comentários de início e fim, incluindo em ambos o autor, a data, a solicitação e a expressão literal `alterado via IA`. Aplique também a inclusões de trechos dentro de rotinas existentes. Preserve comentários históricos e limite os delimitadores ao trecho afetado, sem abranger código inalterado desnecessariamente. Modelo, substituindo os campos entre `< >` pelos dados reais:

```foxpro
&& <autor> - <DD/MM/AAAA> - Sol. [<solicitação>] - alterado via IA - Início
* Código alterado ou incluído na rotina existente.
&& <autor> - <DD/MM/AAAA> - Sol. [<solicitação>] - alterado via IA - Fim
```

Ao incluir uma rotina inteira nova, como método, FUNCTION ou PROCEDURE, adicione apenas um comentário de autoria no início do corpo, antes da primeira instrução executável. Identifique o tipo e o nome da rotina, seguidos de `criado`, autor, data, solicitação e `alterado via IA`. Não acrescente delimitador de fim nem envolva todo o corpo no par Início/Fim. Em PRG, mantenha a declaração da rotina antes desse comentário; no corpo de método de formulário ou classe, coloque-o no topo. Exemplo de cabeçalho:

```foxpro
&& Método <nome_do_metodo> criado - <autor> - <DD/MM/AAAA> - Sol. [<solicitação>] - alterado via IA
```

Para FUNCTION e PROCEDURE, substitua `Método` pelo tipo correspondente. Alterar uma rotina já existente, mesmo reescrevendo todo o corpo, não a torna uma rotina nova: use os delimitadores de alteração. Se uma rotina nova for chamada por uma rotina existente, aplique o cabeçalho de criação na nova e os delimitadores no trecho alterado da chamadora. Confira esses comentários também no texto obtido após reconstruir um artefato binário.

## Métodos e responsabilidades

Prefira métodos e funções reutilizáveis. Separe operações distintas quando agrupá-las provocar execução desnecessária ou impedir reutilização. O documento apresenta essa separação como orientação, não como regra absoluta para fragmentar toda rotina.

Adote, quando pertinentes, métodos como `criar_cursores`, `apagar_cursores`, `configurar_grids`, consulta, atualização dos detalhes selecionados, validação e gravação. Adapte os nomes ao domínio.

- A consulta deve carregar dados e permitir uso em diferentes contextos, sem atualizar obrigatoriamente a interface.
- Faça o `Refresh` do grid no chamador, separado do carregamento.
- Concentre na configuração inicial dos grids propriedades como `RecordSource`, `ControlSource` e `DynamicBackColor`.
- Separe validação de entrada e persistência. O chamador valida antes de salvar; o método de gravação envia os valores à rotina de persistência. Preserve tratamento de erros e restrições de integridade do banco.

## Cursores e eventos

- Prefira criar cursores do formulário no `Load` e encerrá-los no `Destroy`, usando métodos próprios.
- Mantenha os nomes dos campos do cursor iguais aos da tabela ou resultado correspondente, quando isso permitir transferência sem mapeamento campo a campo e loops desnecessários. Preserve compatibilidade de tipos e estrutura.
- Libere os cursores auxiliares ao terminar seu uso, inclusive em caso de erro, sem fechar recursos de outros componentes.
- Configure os grids no `Init` e carregue os dados iniciais nesse evento quando o fluxo exigir. Trate falhas nas operações que acessam o banco.
- Quando a seleção de linha precisar atualizar detalhes, use um evento apropriado, como `AfterRowColChange`, delegando a atualização a um método.

## Estado e comunicação entre formulários

Passe por parâmetro as informações essenciais para abrir um formulário. Use propriedades do formulário para o estado que pertence a ele, evitando variáveis públicas.

No exemplo, `id_setor = 0` identifica inclusão; outro identificador identifica alteração. Aplique essa convenção apenas quando compatível com o contrato do projeto.

Guarde estado exclusivo de um controle em propriedade do próprio controle. O exemplo usa `oldValue` para preservar o valor ao receber foco. Esse é um nome do exemplo; novas propriedades devem seguir snake_case, como `old_value`, sem renomear contratos existentes sem necessidade.

Defina retorno padrão para cancelamento ou ausência de gravação. Sinalize sucesso somente depois de confirmar a persistência. O exemplo usa `gravacao_efetuada` e devolve o resultado à tela anterior sem variável pública. Preserve o mecanismo de retorno efetivamente adotado; não presuma que uma propriedade estará acessível após destruir o formulário.

## Validação e reutilização

Concentre as condições de negócio no método de validação. A descrição é o único campo obrigatório no exemplo de setores; não generalize isso para outros cadastros. Valide cores e outras entradas conforme os requisitos reais.

Considere métodos ou classes para reduzir repetição quando viável. O documento apresenta classes como possibilidade de evolução, sem exigir nova hierarquia em toda alteração.

Se a tarefa incluir SQL, aplique também a skill `sqlserver-stored-procedure-padrao` deste plugin, respeitando o dialeto e as interfaces existentes. Os nomes de procedures, tabelas e campos do exemplo não definem um esquema universal.

## Versão da tela ao finalizar alterações

Ao concluir uma alteração em uma tela VFP, inclusive em seus métodos ou comportamento, atualize o label de versão da tela antes da entrega. Em pedidos somente de análise, não altere a versão.

- Identifique o label pela aparência e posição na tela, pelo texto de versão exibido e pela relação com o título/cabeçalho. O nome do componente não é padronizado: não dependa de nomes como `lb_versao` ou `lbl_versao`, nem renomeie um controle existente apenas para padronizá-lo. Confira o visual no designer, na execução ou em uma imagem atual e relacione-o ao controle na representação textual convertida. Se não conseguir identificar o controle com segurança, informe a limitação em vez de escolher um label arbitrário.
- Antes de modificar, guarde o `Caption` anterior e o `Comment` existente. Atualize o `Caption` com a data e a hora locais da conclusão da alteração, no formato já adotado pela tela. O exemplo fornecido usa `v.DD.MM.AA HHmm`, como `v.13.11.26 1615`. Não copie a data do exemplo como versão nova; use a data e a hora efetivas.
- Atualize também a propriedade `Comment` desse mesmo componente com a versão que acabou de ser substituída, preservando o histórico anterior. No padrão do exemplo, use `Ant.: <versão substituída>` e mantenha depois dela as versões mais antigas, com o separador existente. Não registre a nova versão como se fosse a anterior e não duplique entradas se refizer a finalização da mesma alteração.
- Se o label não existir, crie um Label visível na região de versão do cabeçalho, seguindo a aparência e o alinhamento da tela, sem sobrepor outros controles. Para um novo componente, use um nome disponível conforme a nomenclatura da skill, como `lbl_versao`. Preencha o `Caption` com a versão atual e o `Comment` com `Ant.: sem versão anterior`; não invente uma versão anterior.
- Verifique se métodos ou propriedades herdadas sobrescrevem o `Caption` em execução e ajuste a origem efetiva quando necessário. Confira o label e o `Comment` após a reconversão para SCX/SCT e, quando o ambiente permitir, confira a versão visível na tela.

Referência fornecida pelo usuário e conferida por conversão de uma cópia de `hromni_notificacao_pedido.scx`, do projeto Omnichannel: o componente é `Container9.lb_versao`, aparece em azul abaixo do título, usa fonte 8 e fundo transparente. Seu `Caption` é `v.13.11.26 1615` e seu `Comment` é `Ant.: v.13.07.26 1320#v.05.02.25 1000`. Ao substituir essa versão, o histórico deve ficar `Ant.: v.13.11.26 1615#v.13.07.26 1320#v.05.02.25 1000`, enquanto o `Caption` recebe a nova data e hora. Esse nome, essas versões e esse caminho de projeto são exemplos, não requisitos para localizar o controle em outras máquinas.

## Conferência e entrega

Confira nomenclatura do trecho alterado, referências de controles, parâmetros, propriedades, vínculos do grid, tratamento de erros e liberação dos recursos usados. Exercite os fluxos afetados, como inclusão, alteração, cancelamento, falha de validação e falha de persistência, conforme o escopo.

Use as verificações e o runtime disponíveis. Sem VFP disponível, faça revisão estática e informe que compilação e execução não foram verificadas. Relate a mudança, a versão anterior e a nova versão da tela, a atualização do `Comment`, a validação realizada e as limitações concretas.

## Origem das orientações

- Páginas 3–5: nomenclatura e prefixos.
- Páginas 7–11: requisitos do exemplo, telas, fluxo e estrutura.
- Páginas 12–21: métodos, cursores, grids, consulta, erros e Refresh separado.
- Páginas 23–29: seleção do grid, parâmetros, propriedades e validações de controles.
- Páginas 30–36: validação, gravação, sinalização de sucesso e retorno padrão.

Preservar contratos existentes, limitar renomeações e declarar verificações indisponíveis são orientações operacionais acrescentadas nesta skill, não transcrições do documento.
