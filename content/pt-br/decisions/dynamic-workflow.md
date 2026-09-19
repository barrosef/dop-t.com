---
title: "O fluxo é dado, e a tela nunca conheceu estágios fixos"
translationKey: "decision-0010"
adr: "0010"
adr_title: "A dynamic, typed and inheritable workflow"
adr_file: "0010-dynamic-workflow.md"
date: 2026-08-30
weight: 10
group: "delivery"
description: "O Jira personaliza só status. O GitHub Actions é fluxo-como-código para uma máquina. O BPMN modela tudo e custa um analista. O meio — estágios tipados, composição livre — estava vago."
related: ["0004", "0009", "0021"]
---

## O que estava na mesa

O ciclo da demanda estava indefinido, e o produto decidiu como deveria ser:
**dinâmico**. A plataforma tem um padrão, mas cada conta, workspace, projeto
e até uma demanda específica pode trabalhar do seu jeito. A tela de chat e o
agente precisam entender qualquer fluxo sem conhecer nenhum — o que significa
que o fluxo é dado, não código.

As referências ocupam os extremos. O Jira personaliza só status — sem
artefatos, sem agente. O GitHub Actions é fluxo-como-código para uma máquina,
não para um humano acompanhar. O BPMN modela tudo e custa um analista. O meio
do caminho — estágios tipados mais composição livre — estava vago.

## Os caminhos que pesamos

**Estágios fixos da plataforma.** O desenho anterior, e o do PRD antigo.
Rejeitado: as contas trabalham diferente, e o custo do dinamismo caiu para
quase zero quando os estágios ficaram tipados — o caso estático vira o caso
particular de um fluxo só.

**Um motor de workflow completo** — BPMN, Temporal. Rejeitado para a v1:
compra condicionais e paralelismo que ninguém pediu, a um preço em
complexidade que todo mundo pagaria.

**Fluxo como código, um YAML por repositório.** Rejeitado como interface
primária: o público é o desenvolvedor-gestor numa tela, não um pipeline.

## O que escolhemos, e por quê

**Os estágios têm tipo semântico; os fluxos são composições.** O vocabulário
de tipos pertence à plataforma — contexto, spec, plano, implementação, teste,
validação humana, finalização, genérico — e o tipo decide o renderizador na
tela e o comportamento do agente: qual artefato produzir, onde parar. Um tipo
novo é uma evolução da plataforma; uma composição nova não é.

A estrutura da v1 é deliberadamente simples — um nome, uma versão, uma lista
de estágios com chave, tipo, artefatos e um portão — sem condicionais, sem
estágios paralelos, sem linguagem de regras.

**Uma cadeia de resolução com herança**: plataforma, conta, workspace,
projeto, demanda — o nível mais próximo vence, herdado por omissão,
sobrescrito por declaração, e a interface sempre mostra de onde veio o fluxo
efetivo. **A demanda congela a versão do fluxo quando começa**; o avanço de um
estágio é um evento, e a régua na tela é uma projeção.

E um refinamento da decisão de compartilhamento: um recurso com credencial
fica fechado por padrão; um recurso de conteúdo — um fluxo, uma skill — fica
aberto dentro de uma organização por padrão. Credencial é risco; fluxo é
conhecimento.

## O que custou

A cadeia de herança exige um rastro visível — "herdado de…" — ou vira um
chamado de suporte. O compartilhamento externo de fluxos entre contas ficou
fora da v1, registrado como estratégico: espera, não dorme. E a sintaxe dos
critérios executáveis dentro do artefato de spec ficou aberta.

## Desde então

Duas adições, as duas dentro da estrutura e as duas testadas contra a linha
"sem DSL de regras". Em 2026-09-03 um artefato ganhou endereço: um arquivo no
repositório raiz do projeto. Em 2026-09-06 um estágio ganhou *ações* — o que a
plataforma faz quando uma demanda entra nele e quando sai — tiradas de um
vocabulário fechado que a plataforma implementa, com parâmetros planos e
nenhuma condição a avaliar. Foi a menor mudança que responde "provisione a
bancada quando a implementação terminar" sem reabrir o que foi recusado: no
momento em que um estágio pudesse dizer *quando* agir em vez de só *o quê*, o
fluxo deixaria de ser dado que um carregador lê e viraria um programa. Um
limite caiu do portão de idempotência e não do desenho, e é recusado em voz
alta quando um fluxo é escrito: um nome de ação não pode se repetir no mesmo
momento de um estágio, porque o segundo seria pulado para sempre, em
silêncio.
