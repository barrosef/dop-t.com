---
title: "Cinco necessidades, um log"
translationKey: "decision-0004"
adr: "0004"
adr_title: "The demand is an event log; everything else is a projection"
adr_file: "0004-demand-as-event-log.md"
date: 2026-08-29
weight: 4
group: "events"
description: "Depuração, auditoria, o dossiê, segurança, métricas — cinco coisas que queriam, cada uma, um registro do que aconteceu. Escrever os mesmos dados cinco vezes garantiria que discordassem."
related: ["0003", "0014", "0008"]
---

## O que estava na mesa

Cinco necessidades distintas pediam, cada uma, um registro do que aconteceu
numa demanda. **Depuração**: reproduzir, passo a passo, uma demanda que deu
errado. **Auditoria**: numa organização, responder "quem autorizou este push,
com qual credencial?" — a pergunta que a decisão das credenciais tinha acabado
de tornar possível. **O dossiê**: os requisitos pediam que fosse "gerado em
tempo de execução, estágio a estágio", não montado no fim. **Segurança**:
perícia e detecção quando conteúdo malicioso tenta desviar o agente.
**Métricas**: intervenções humanas, retrabalho, tempo até o verde.

Construir cinco mecanismos é escrever os mesmos dados cinco vezes e assistir
a eles divergirem.

## Os caminhos que pesamos

**Um armazenamento por consumidor** — uma tabela de dossiê, uma trilha de
auditoria, um pipeline de métricas. Rejeitado: escrita tripla, divergência
garantida, e o replay nunca chega.

**Um log textual sem estrutura.** Rejeitado: não é consultável nem
projetável. Auditoria num sistema multi-tenant precisa de campos, não de grep.

## O que escolhemos, e por quê

**Toda ação numa demanda emite um evento imutável** num log só de acréscimo:
quando, quem — humano, agente ou subagente —, o quê, qual credencial, um
resumo da entrada, o resultado. O log é a espinha da demanda, e o dossiê, a
linha do tempo, a auditoria, o replay e as métricas são **projeções** dele:
leituras, nunca escritas próprias.

Fixou uma regra para a decisão de persistência antes de ela ser tomada: o log
de eventos é cidadão de primeira classe do armazenamento, e o que guarda
documentos serve às projeções; não substitui o log.

## O que custou

A disciplina de emitir em todo lugar. Uma ação sem evento é um bug, não um
detalhe. E volume: o log cresce com a frota, então retenção e compactação
viraram uma pergunta para a decisão de persistência.

## Desde então

Esta é a decisão que o resto da plataforma continuou sacando. Os achados dos
subagentes e a medição de custo viraram dois tipos de evento a mais, em vez de
dois mecanismos. A caixa de atenção virou uma projeção. A análise de segurança
do convite ([ADR-0019](../invite-without-token/)) foi possível por sabermos
exatamente os quatro lugares onde um evento pousa. E a decisão de verificação
de chamadores ([ADR-0022](../the-core-verifies-its-callers/)) foi argumentada
daqui: a autoria de um evento vale o que vale o ator por trás dela.

Em setembro os próprios eventos cresceram. O envelope ganhou contexto — a
chave do agregado, o ator, a requisição, a sessão, o chamador — e a fila de
dead letters que a [ADR-0014](../postgres-persistence/) descrevia como
existente foi finalmente construída, depois de descobrirmos que um evento
esgotado tinha sido descartado sob uma linha de log dizendo que estava salvo.
