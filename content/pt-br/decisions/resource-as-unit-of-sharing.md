---
title: "Quatro coisas para compartilhar, um jeito de compartilhar"
translationKey: "decision-0009"
adr: "0009"
adr_title: "The resource as the account's unit of ownership and sharing"
adr_file: "0009-resource-as-unit-of-sharing.md"
date: 2026-08-30
weight: 9
group: "resources"
description: "O compartilhamento existia só para integrações. Então skills, workflows e git flows pediram a mesma coisa — e um mecanismo por tipo seria o erro que a decisão das contas tinha acabado de evitar."
related: ["0002", "0010", "0016"]
---

## O que estava na mesa

O modelo de compartilhamento existia para uma coisa: integrações, com grants
de `use` e `manage` compostos no convite. Então outras coisas que uma conta
possui e quer compartilhar sob autorização apareceram. **Skills** — as
capacidades reutilizáveis dos agentes. **Workflows humano-agente** — como um
desenvolvedor e os agentes colaboram numa demanda. **Git flows** — governança
de branches como artefato: uma taxonomia por tipo de card, base e direção,
composição de release, *back-merge* de hotfix, políticas — um conceito tirado
de governança real em produção, onde o tipo do card decide o prefixo, a base
e o fluxo do branch. E as próprias integrações ganharam uma terceira
categoria: **provedores de agente**, Claude e Codex entre eles.

Um mecanismo de compartilhamento por tipo repetiria o erro que a decisão das
contas tinha acabado de evitar: a mesma política implementada N vezes,
divergindo.

## Os caminhos que pesamos

**Um mecanismo de compartilhamento por tipo.** Rejeitado: a mesma política
escrita quatro vezes, com quatro telas e quatro bugs.

**Tudo como "integração".** Rejeitado: uma skill e um fluxo não têm
credencial; têm versão e conteúdo. Forçá-los na entidade errada cobraria por
isso a cada evolução.

## O que escolhemos, e por quê

**`Resource` é a unidade de posse e de compartilhamento** — id, conta, tipo,
nome, config e uma referência opcional a credencial. Os tipos iniciais:
`integration` (git, gerenciador de tarefas, e agora `agent`; o único com
credencial), `skill`, `workflow`, `git_flow`.

O grant passou a ser por recurso — `use`/`manage`, por usuário, composto no
convite, editável a qualquer momento — e as regras existentes não mudaram,
generalizaram: o recurso de uma conta pessoal é privado; só o de uma
organização é compartilhável; donos e admins têm um `manage` implícito;
revogar `use` não desmonta o que já está configurado.

A plataforma, no nível zero, oferece um catálogo de recursos globais —
provedores, skills, fluxos padrão — que uma conta *adota* e passa a governar
como seus. E um projeto consome os recursos da conta dona do seu workspace: o
git flow anexado a um projeto parametriza a fila de merge e a verificação; as
skills e o workflow parametrizam os agentes.

## O que custou

A tabela de grants generalizou, e o schema nasceu assim. Adotar um recurso
global exige uma decisão de versionamento — cópia ou referência — por tipo,
registrada na spec de recursos.

## Desde então

O recurso workflow ganhou forma no mesmo dia
([ADR-0010](../dynamic-workflow/)), que também refinou o padrão de acesso por
tipo: um recurso com credencial fica fechado; um recurso de conteúdo numa
organização fica aberto dentro da conta por padrão. Credencial é risco; fluxo
é conhecimento. E quando o runtime do agente foi redesenhado
([ADR-0016](../agent-provider-as-port/)), o fato de um provedor de agente já
ser um recurso — Claude e Codex como duas linhas, não duas versões do código —
foi o que tornou um port por fornecedor a forma natural, e não uma ideia nova.
