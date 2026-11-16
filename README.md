# hr-skills-dev

Projeto da HR Tech Solucoes para versionar e distribuir o plugin `hr-skills-dev`
no ChatGPT, Codex e Claude, com uma unica copia das tres skills.
Versao atual: `0.1.5`.

## Skills incluidas

- `desenvolvimento-vfp`: padrao de desenvolvimento Visual FoxPro.
- `sqlserver-stored-procedure-padrao`: padrao de procedures SQL Server, incluindo referencias e metadados associados.
- `vfp9-hrfoxbin2prg`: conversao de artefatos binarios VFP9.

As pastas foram inicialmente copiadas integralmente das skills pessoais.
Os ajustes posteriores estao registrados no `CHANGELOG.md`.
O inventario de origem e os hashes iniciais estao em `docs/origem-skills.json`.
A versao `0.1.2` usa `Skills_Desenvolvimento.zip` como fonte; o inventario dessa
atualizacao e os ajustes de referencias estao em `docs/origem-skills-0.1.2.json`.
A skill `padrao-sql` foi substituida por `sqlserver-stored-procedure-padrao`.
Nao foram incluidos plugins oficiais, outras skills, configuracoes de conta ou credenciais.

## Estrutura

- `.agents/plugins/marketplace.json`: catalogo do marketplace para ChatGPT/Codex.
- `.claude-plugin/marketplace.json`: catalogo do marketplace para Claude.
- `plugins/hr-skills-dev/plugin.json`: manifesto portavel do plugin.
- `plugins/hr-skills-dev/.claude-plugin/plugin.json`: manifesto para Claude.
- `plugins/hr-skills-dev/skills/`: fontes das tres skills.
- `CHANGELOG.md`: historico das versoes publicadas.

## Versionamento

O projeto ja esta versionado no Git, na branch `main`, com o remoto `origin`
configurado para o repositorio privado da equipe:

- Repositorio: [HR-Tech-Solucoes/hr-skills-dev](https://github.com/HR-Tech-Solucoes/hr-skills-dev).
- Pasta local de trabalho: `C:\projetos\hr-skills-dev`.

Antes de cada release,
revise as alteracoes, atualize a versao nos dois manifestos do plugin e o `CHANGELOG.md`,
e teste as skills em uma nova conversa. Os hashes registram a importacao inicial;
nao devem impedir futuras edicoes das skills.

## Uso e distribuicao

O uso no ChatGPT foi confirmado pelo usuario. O suporte ao Claude foi preparado
nesta versao e ainda precisa ser testado no cliente de destino.

Os dois catalogos apontam para `plugins/hr-skills-dev`, compartilhando as mesmas
skills e referencias. Cada plataforma tem seus proprios controles de instalacao,
acesso ao GitHub privado e sincronizacao; configurar um cliente nao configura o outro.

### ChatGPT e Codex

No workspace corporativo, usar Admin > Plugins > Add > Import marketplace:

- Source: `https://github.com/HR-Tech-Solucoes/hr-skills-dev`.
- Path: vazio.
- Branch: `main`.

Autorizar o acesso ao GitHub privado, conferir o resultado e configurar a politica
de instalacao. Para atualizar sem esperar a sincronizacao diaria, usar Sync now.
Ver [gerenciamento corporativo de plugins](https://learn.chatgpt.com/docs/enterprise/plugin-management).

### Claude na organizacao

Nos planos Team e Enterprise, o responsavel pela organizacao pode conectar o
repositorio privado em Organization settings > Plugins & skills > Marketplaces,
pela opcao Sync from GitHub. Informar `HR-Tech-Solucoes/hr-skills-dev` e autorizar
o acesso da integracao Claude GitHub App ao repositorio. Cowork e Skills precisam
estar habilitados. Depois, configurar o acesso da equipe ao plugin.

Os plugins distribuidos pela organizacao aparecem no chat, Cowork e Claude Code
quando usados com a mesma conta, conforme os controles da organizacao.
Para atualizar, usar Re-sync; conferir as condicoes de sincronizacao automatica
na [documentacao do Claude](https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization).

### Claude Code: instalacao individual

Com Claude Code instalado e acesso Git ao repositorio privado, executar no terminal:

```powershell
claude plugin marketplace add https://github.com/HR-Tech-Solucoes/hr-skills-dev.git
claude plugin install hr-skills-dev@hr-skills-dev
```

Para um teste antes de publicar, executar na raiz deste projeto:

```powershell
claude plugin validate .
claude plugin validate ./plugins/hr-skills-dev
claude plugin marketplace add .
claude plugin install hr-skills-dev@hr-skills-dev
```

Em uma nova sessao, conferir as tres skills e testar, por exemplo,
`/hr-skills-dev:desenvolvimento-vfp`.
Ver [marketplaces do Claude Code](https://code.claude.com/docs/en/plugin-marketplaces).

A validacao deve ser feita em novas conversas. A atualizacao de instrucoes ja
carregadas em conversas abertas ainda nao foi confirmada.

## Dependencias e pontos a adaptar

A skill `vfp9-hrfoxbin2prg` referencia `C:\Desenvolve\HRFoxBin2prg` e runtime VFP9.
O conversor e o runtime nao estao incluidos: precisam existir nas maquinas de uso,
ou seus caminhos devem ser adaptados em uma alteracao posterior.

A skill `desenvolvimento-vfp` agora referencia a skill de conversao incluida,
`vfp9-hrfoxbin2prg`, em vez do nome antigo `foxbin2prg`. A referencia foi corrigida na versao
`0.1.1`; os hashes de origem permanecem como registro da importacao inicial.

Instalar as skills nao concede acesso ao disco local nem instala o conversor.
Conversao e reconstrucao de binarios dependem de um ambiente Windows com as
ferramentas e permissoes necessarias, mesmo ao usar um cliente de chat na nuvem.

## Documentacao oficial

- [Construcao de plugins](https://developers.openai.com/plugins/build/plugins).
- [Gerenciamento corporativo de plugins](https://learn.chatgpt.com/docs/enterprise/plugin-management).
- [Manifesto de plugins do Claude](https://code.claude.com/docs/en/plugins-reference).
- [Marketplaces do Claude Code](https://code.claude.com/docs/en/plugin-marketplaces).
- [Distribuicao corporativa no Claude](https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization).
