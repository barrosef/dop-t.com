---
title: "Toda escrita é um evento; todo o resto é projeção"
translationKey: "decision-0004"
adr: "0004"
adr_title: "The demand is an event log; everything else is a projection"
adr_file: "0004-demand-as-event-log.md"
date: 2026-08-29
weight: 4
group: "events"
description: "Toda escrita no core emite um evento imutável que carrega quem, o quê, quando e em qual contexto. O dossiê, a linha do tempo, a auditoria, o replay e as métricas são lidos desse log — nenhum deles é escrito à parte."
related: ["0003", "0014", "0008"]
---

## O que estava na mesa

Cinco necessidades pedem, cada uma, um registro do que aconteceu numa
demanda: depuração (reproduzir uma demanda passo a passo), auditoria (quem
autorizou este push, com qual credencial), o dossiê gerado estágio a
estágio, perícia de segurança, e métricas — intervenções, retrabalho, tempo
até o verde. Um registro precisa servir às cinco.

## Os caminhos que pesamos

**Um armazenamento por consumidor** — uma tabela de dossiê, uma trilha de
auditoria, um pipeline de métricas. Rejeitado: várias escritas, divergência
garantida, e nenhum replay.

**Um log textual sem estrutura.** Rejeitado: auditoria num sistema
multi-tenant precisa de campos, não de grep.

## O que escolhemos, e por quê

**Toda escrita no core emite um evento imutável** num log só de acréscimo,
na mesma transação da mudança de estado. O envelope carrega o id do evento,
a conta, o agregado a que pertence e uma chave legível do agregado
(`account-created`, `pr-delivered`), o tipo, o payload, o horário — e o
contexto da chamada: quem agiu (usuário, agente ou plataforma), a
requisição, a sessão, o chamador.

**Todo o resto é projeção.** O dossiê, a linha do tempo, a auditoria, o
replay, as métricas e a caixa de atenção são leituras do log, nunca
escritas próprias. Uma ação sem evento é um defeito.

## O que custou

A disciplina de emitir em todo lugar. Um volume que cresce com a frota,
tratado com particionamento e retenção na camada de persistência.

## Desde então

O envelope ganhou os campos de contexto — chave do agregado, ator,
requisição, sessão, chamador — em 2026-09-13, junto com o caminho de dead
letters que carrega o envelope inteiro quando um consumidor falha. Achados,
medição de custo e notificações são tipos de evento em vez de mecanismos, e
[a verificação de chamadores](../the-core-verifies-its-callers/) existe para
que a autoria de um evento valha alguma coisa.
