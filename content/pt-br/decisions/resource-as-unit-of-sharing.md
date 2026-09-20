---
title: "Um tipo de recurso, um jeito de compartilhar"
translationKey: "decision-0009"
adr: "0009"
adr_title: "The resource as the account's unit of ownership and sharing"
adr_file: "0009-resource-as-unit-of-sharing.md"
date: 2026-08-30
weight: 9
group: "resources"
description: "Integrações, skills, workflows e git flows são todos recursos de uma conta, compartilhados pelos mesmos grants e pelo mesmo convite. Um tipo novo de recurso nunca toca o mecanismo de compartilhamento."
related: ["0002", "0010", "0016"]
---

## O que estava na mesa

Uma conta é dona de mais que integrações: as skills reutilizáveis dos
agentes, os workflows que um desenvolvedor e os agentes seguem numa demanda,
e os git flows — governança de branches como artefato: uma taxonomia por
tipo de card, base e direção, composição de release, *back-merge* de hotfix.
E as próprias integrações têm três categorias: hospedagens git,
gerenciadores de tarefa e provedores de agente. Tudo isso tem dono, é
versionado e é compartilhado sob autorização.

## Os caminhos que pesamos

**Um mecanismo de compartilhamento por tipo.** Rejeitado: a mesma política
escrita quatro vezes, com quatro telas e quatro conjuntos de bugs.

**Tudo como integração.** Rejeitado: uma skill e um fluxo têm conteúdo e
versão, não credencial; a entidade errada cobra a cada evolução.

## O que escolhemos, e por quê

**`Resource` é a unidade de posse e de compartilhamento** — id, conta, tipo,
nome, config e uma referência opcional a credencial. Quatro tipos:
`integration` (git, gerenciador de tarefas, agente — o único com
credencial), `skill`, `workflow`, `git_flow`.

**Os grants são por recurso:** `use` e `manage`, por usuário, compostos no
convite e editáveis a qualquer momento. O recurso de uma conta pessoal é
privado; só o de uma organização é compartilhável; donos e admins têm um
`manage` implícito; revogar `use` não desmonta o que já está configurado.

**A plataforma publica recursos globais** — provedores, skills, fluxos
padrão — que uma conta adota, por cópia ou por referência versionada, e
passa a governar como seus. **Um projeto consome os recursos da conta dona
do seu workspace:** o git flow parametriza a fila de merge e a verificação;
as skills e o workflow parametrizam os cards dos agentes.

## O que custou

A tabela de grants é genérica desde o nascimento. Adotar um recurso global
exige uma decisão de versionamento por tipo, registrada na spec de recursos.

## Desde então

O tipo workflow ganhou forma no mesmo dia, junto com um refinamento dos
padrões: um recurso com credencial fica fechado por padrão; um recurso de
conteúdo numa organização fica aberto dentro da conta por padrão. Um
provedor de agente ser um recurso é o que fez de [um port por
fornecedor](../agent-provider-as-port/) o desenho natural quando o runtime
foi decidido.
