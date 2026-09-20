---
title: "O fluxo é dado: estágios tipados, composição livre, herança"
translationKey: "decision-0010"
adr: "0010"
adr_title: "A dynamic, typed and inheritable workflow"
adr_file: "0010-dynamic-workflow.md"
date: 2026-08-30
weight: 10
group: "delivery"
description: "Os estágios têm um tipo que a plataforma sabe renderizar e executar; os fluxos os compõem livremente e são resolvidos por uma cadeia — plataforma, conta, workspace, projeto, demanda. Sem condicionais, sem linguagem de regras: um fluxo é dado que um carregador lê."
related: ["0004", "0009", "0021"]
---

## O que estava na mesa

O fluxo de desenvolvimento humano↔agente é configurável por conta,
workspace, projeto e até demanda. O cockpit e o agente precisam operar
qualquer fluxo sem conhecer um específico. As referências ficam nos
extremos: gerenciadores de tarefa personalizam só status; sistemas de CI
são fluxo-como-código para uma máquina; o BPMN modela tudo ao custo de um
analista.

## Os caminhos que pesamos

**Estágios fixos da plataforma.** Rejeitado: as contas trabalham diferente,
e estágios tipados fazem do caso fixo um fluxo particular.

**Um motor de workflow** — BPMN, Temporal. Rejeitado para a v1: condicionais
e paralelismo que ninguém pediu.

**Fluxo como código, um YAML por repositório.** Rejeitado como interface
primária; o público é um desenvolvedor numa tela.

## O que escolhemos, e por quê

**Os estágios têm tipo semântico; os fluxos são composições.** O
vocabulário de tipos pertence à plataforma — contexto, spec, plano,
implementação, teste, validação humana, finalização, genérico — e o tipo
decide o renderizador no cockpit e o comportamento do agente. Um tipo novo
é mudança na plataforma; uma composição nova não é.

**A estrutura é deliberadamente pequena:** um nome, uma versão, estágios
com chave, tipo, artefatos, um portão e, opcionalmente, ações. Um artefato é
um arquivo no repositório de conhecimento do projeto. **Ações de estágio** —
o que a plataforma faz quando uma demanda entra ou sai de um estágio — vêm
de um vocabulário fechado (`open_attention`, `close_attention`,
`send_email`, `provision_bench`) com parâmetros planos: uma declaração, não
um programa.

**Resolução com herança:** plataforma, conta, workspace, projeto, demanda —
o nível declarado mais próximo vence, e a interface mostra de onde veio o
fluxo efetivo. **Uma demanda congela a versão do fluxo quando começa**; o
avanço de estágio é um evento. Um fluxo pode ser promovido a um nível acima
por quem tem `manage`. Recursos de conteúdo como fluxos são abertos dentro
de uma organização por padrão; recursos com credencial são fechados.

## O que custou

A cadeia de herança exige o rastro "herdado de". O compartilhamento externo
de fluxos entre contas ficou fora da v1, registrado como estratégico. A
sintaxe dos critérios executáveis dentro do artefato de spec é definida na
spec de workflow.

## Desde então

Os artefatos ganharam endereço no repositório de conhecimento em 2026-09-03,
e as ações de estágio foram acrescentadas em 2026-09-06 — as duas dentro da
estrutura, as duas mantendo o fluxo como dado em vez de código.
