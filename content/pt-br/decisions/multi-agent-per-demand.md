---
title: "Um subagente com quem se pode conversar"
translationKey: "decision-0007"
adr: "0007"
adr_title: "Multi-agent per demand: addressable threads and published findings"
adr_file: "0007-multi-agent-per-demand.md"
date: 2026-08-29
weight: 7
group: "agents"
description: "No mercado um subagente é uma caixa-preta: você despacha e espera. Queríamos um com thread própria, interrogável em voo — e um jeito de os especialistas compartilharem o que acharam sem despejar o contexto inteiro uns nos outros."
related: ["0004", "0006", "0008", "0017"]
---

## O que estava na mesa

Uma demanda real pode precisar de especialistas ao mesmo tempo. O caso que
deu forma a isto veio da própria história do produto: o agente principal
implementa; um subagente faz uma leitura forense do banco pela ferramenta
MySQL do workspace; outro vasculha os logs do servidor. O desenvolvedor
precisa falar com os três sem misturar as linhas do tempo, e os três precisam
usar as conversas uns dos outros como conhecimento.

Dois eixos tinham que ficar separados. Entre demandas, a fronteira é dura —
uma demanda, uma microVM. Dentro de uma demanda, N agentes dividem uma sandbox
e colaboram. Esta decisão é sobre o segundo. E não havia o que copiar: um
subagente endereçável, com thread própria e interrogável em voo, não existia
nas ferramentas da época.

## Os caminhos que pesamos

**Um único agente sequencial.** Rejeitado: perde a especialização e não
paraleliza nada.

**Uma sandbox por subagente.** Rejeitado: quebra o workspace compartilhado — o
agente forense precisa do mesmo banco que o agente principal sobe —,
multiplica custo, e não compra isolamento que importe, porque os agentes
cooperam.

**Uma única linha do tempo compartilhada.** Rejeitado: é o problema que o
requisito veio resolver.

## O que escolhemos, e por quê

A conversa da demanda é um **conjunto de threads**, não uma linha do tempo:
`#main` mais uma por subagente, cada uma com o próprio histórico; o
desenvolvedor entra numa e fala com aquele agente em isolamento. Todo
subagente nasce com um **card** — propósito, ferramentas concedidas, o modelo
que o roteador escolheu, uma fatia do orçamento da demanda.

**Conhecimento cruzado por consulta, não por despejo.** As threads são
legíveis pelas irmãs como uma ferramenta — ler uma thread, fazer uma
pergunta. Despejar linhas do tempo inteiras no contexto de todo agente não
escala nem é seguro; amplia a superfície para instruções injetadas.

**Uma conclusão vira um achado publicado** — um resultado estruturado no
quadro comum da demanda: "um deadlock na tabela X entre 14:02 e 14:07, causado
pela migration Y". Os achados entram automaticamente no contexto das irmãs,
são eventos, e alimentam a memória do projeto.

Tanto o humano quanto o agente principal podem lançar subagentes — o agente
por iniciativa própria quando julgar necessário, a thread aparecendo na hora
para o desenvolvedor acompanhar ou entrar. Essa iniciativa é uma premissa
registrada, adotada por coerência com a autonomia; o produto pode
restringi-la. E a fronteira de segurança continua sendo a demanda: os
subagentes dividem a microVM, o workspace, a credencial e a cota. São
colaboradores, não estranhos.

## O que custou

O runtime precisa suportar N sessões por sandbox. E mais threads significa
mais pontos de atenção: a caixa de atenção deixa de ser opcional e vira
pré-requisito de escala.

## Desde então

O achado se revelou mais que um recurso de UX. Quando a decisão de custo
([ADR-0008](../llm-cost-governance/)) procurou onde um loop de agente
desperdiça dinheiro, a maior economia já estava aqui: logs, dumps e leituras
volumosas ficam em quarentena na thread do especialista, e o agente principal
recebe o achado. Metade da economia de tokens veio de uma decisão tomada por
outra razão.
