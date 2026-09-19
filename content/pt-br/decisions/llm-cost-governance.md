---
title: "A maior linha de custo que ninguém tinha escrito"
translationKey: "decision-0008"
adr: "0008"
adr_title: "The LLM's cost: measuring it, capping it, routing it, and spending less"
adr_file: "0008-llm-cost-governance.md"
date: 2026-08-29
weight: 8
group: "agents"
description: "N agentes, sessões longas, muitos tenants — e nenhuma linha sobre quanto custa. Medir, limitar, rotear; e perceber que metade da economia veio de decisões já tomadas por outras razões."
related: ["0004", "0005", "0007", "0016"]
---

## O que estava na mesa

N agentes autônomos, vezes sessões longas, vezes muitos tenants: o maior
custo variável do produto — e nenhum documento tinha uma linha sobre ele. Sem
medição por conta não há modelo de negócio. Sem orçamento por demanda, uma
demanda patológica queima dinheiro em loop. Sem roteamento, tudo roda no
modelo mais caro.

Por baixo dos três está a mecânica de um loop de agente: **a conversa inteira
é reenviada a cada turno.** Um agente de quarenta turnos paga a transcrição
quarenta vezes. A API oferece descontos de até noventa por cento com cache e
cinquenta com lote, mas nenhum é uma flag; todos exigem disciplina de
engenharia.

## Os caminhos que pesamos

**Um modelo forte para tudo.** Simples e caro; vira um teto na margem.

**Rotear pelo tamanho do prompt.** Rejeitado: o que importa é a natureza da
tarefa, não o comprimento.

**Só um roteador de modelos.** Insuficiente: o maior desperdício é reenviar a
transcrição, e o roteador não toca nisso.

**Compactação como jeito de retomar uma demanda.** Rejeitado: reconstruir o
contexto a partir dos eventos é mais barato, mais limpo, e o material já
existia.

## O que escolhemos, e por quê

**Medir desde o primeiro dia.** Todo uso de modelo emite um evento de custo —
tokens, modelo, demanda, thread, conta — no log da demanda; a medição é uma
projeção, não um sistema paralelo. O evento carrega leituras e escritas de
cache, então um *miss* recorrente num prefixo estável é um alerta, não um
mistério.

**Um orçamento por demanda com corte suave.** No estouro a demanda pausa e
pergunta, pela caixa de atenção; nunca morre no meio e nunca continua
queimando. O agente enxerga o teto e se regula.

**Um roteador que escolhe modelo e esforço por tipo de trabalho** — trabalho
mecânico na classe barata, investigação na média, planejamento e
implementação na forte — com a tabela marcada como *rascunho*, a calibrar com
telemetria. Uma regra limita todas as outras: **não se economiza no
crítico.** Modelo forte, esforço máximo. Economizar no freio devolve o custo
como um pull request rejeitado, o retrabalho mais caro do fluxo.

**Cache primeiro.** O prompt é disposto para um prefixo estável — sistema,
ferramentas, pacote de contexto, depois a conversa — e o pacote é
serializado deterministicamente: ordem estável, sem timestamps, sem ids
voláteis, porque um byte mudado invalida tudo depois dele. A intervenção de um
operador entra como mensagem no meio, nunca editando o topo.

**Contexto sujo não entra no agente principal.** A thread do especialista
guarda os logs e os dumps; o agente principal recebe o achado. Onde cabe, o
filtro roda como código na sandbox e só o resultado passa pelo modelo.

**Retomar por reconstrução, não por replay.** Uma demanda que retoma dias
depois não reenvia a transcrição — o cache já expirou de qualquer jeito. O
contexto é reconstruído do pacote, dos achados e de um resumo do rastro.

**Não usar modelo onde código resolve.** O dossiê, as métricas, a auditoria
e a caixa de atenção são projeções do log, computadas em código, a custo zero
de tokens. Trabalho assíncrono — o índice depois de um merge, métricas
noturnas — vai para a API em lote.

## O que custou

A tabela de roteamento é um palpite até haver dados — daí o status de
rascunho. A serialização determinística do pacote é uma restrição permanente
sobre o subsistema de contexto. E a reconstrução na retomada precisa ser
comprovadamente suficiente: se o agente "esquece" o que importava, o resumo
do rastro é que está fraco.

## Desde então

O registro foi escrito como dois — um para medir e limitar, outro para gastar
menos — e fundido em um em 2026-09-04, porque o segundo sempre se declarou
complemento do primeiro. Quando o runtime do agente virou um port por
fornecedor ([ADR-0016](../agent-provider-as-port/)), a separação entre
*política* (qual classe) e *catálogo* (qual modelo concreto) foi o que
permitiu às regras de custo continuarem valendo para todo fornecedor sem
conhecer nenhum — e o mesmo registro avisou que a semântica de cache de
prefixo difere entre fornecedores, que é a divergência mais cara de que esta
decisão depende.
