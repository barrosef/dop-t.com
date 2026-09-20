---
title: "Threads que se endereçam, achados que se publicam"
translationKey: "decision-0007"
adr: "0007"
adr_title: "Multi-agent per demand: addressable threads and published findings"
adr_file: "0007-multi-agent-per-demand.md"
date: 2026-08-29
weight: 7
group: "agents"
description: "A conversa de uma demanda é um conjunto de threads — o agente principal e uma por especialista — cada uma com histórico e card próprios. Os especialistas compartilham o que acharam como achados estruturados, não copiando o contexto inteiro uns nos outros."
related: ["0004", "0006", "0008", "0017"]
---

## O que estava na mesa

Uma demanda pode precisar de vários agentes ao mesmo tempo: um
implementando, um lendo um banco forensicamente, um vasculhando logs. O
desenvolvedor precisa falar com cada um sem misturar as linhas do tempo, e
cada um precisa usar o que os outros acharam. Dentro de uma demanda os
agentes dividem uma sandbox e cooperam; entre demandas a fronteira é dura.

## Os caminhos que pesamos

**Um único agente sequencial.** Rejeitado: sem especialização, sem
paralelismo.

**Uma sandbox por subagente.** Rejeitado: quebra o workspace compartilhado
de que os especialistas precisam, multiplica custo e isola agentes que
cooperam.

**Uma linha do tempo compartilhada.** Rejeitado: linhas do tempo separadas
são o requisito.

## O que escolhemos, e por quê

**A conversa da demanda é um conjunto de threads:** `#main` mais uma por
subagente, cada uma com o próprio histórico; o desenvolvedor endereça uma
thread de cada vez. **Todo subagente tem um card** — propósito, ferramentas
concedidas, o modelo que o roteador escolheu, uma fatia do orçamento da
demanda.

**Conhecimento entre threads é por consulta, não por despejo.** As threads
são legíveis pelas irmãs por ferramentas; nenhuma linha do tempo é copiada
no contexto de outra. **Uma conclusão é um achado publicado** — um resultado
estruturado no quadro da demanda que entra no contexto das irmãs, é um
evento, e é escrito na memória do projeto.

Os subagentes são lançados pelo humano, pelo chat, ou pelo agente principal
por iniciativa própria — uma premissa que o produto pode restringir. A
fronteira de segurança continua sendo a demanda: os subagentes dividem a
sandbox, o workspace, a credencial e a cota.

## O que custou

O runtime suporta várias sessões por sandbox, e a caixa de atenção vira
pré-requisito de escala em vez de opção.

## Desde então

A quarentena da saída crua de ferramentas na thread do especialista é também
a maior economia da [decisão de custo](../llm-cost-governance/): o agente
principal recebe o achado, nunca o despejo.
