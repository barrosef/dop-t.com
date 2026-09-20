---
title: "Medir, limitar, rotear — e desenhar o prompt para o cache"
translationKey: "decision-0008"
adr: "0008"
adr_title: "The LLM's cost: measuring it, capping it, routing it, and spending less"
adr_file: "0008-llm-cost-governance.md"
date: 2026-08-29
weight: 8
group: "agents"
description: "Toda chamada a um modelo emite um evento de custo; toda demanda tem um orçamento que pausa em vez de matar; um roteador escolhe a classe de modelo por tipo de trabalho — nunca mais barato para o crítico. E o prompt é disposto para que a parte cara fique em cache."
related: ["0004", "0005", "0007", "0016"]
---

## O que estava na mesa

O uso de modelos é o maior custo variável do produto. Sem medição por conta
não há modelo de negócio; sem orçamento por demanda, uma demanda pode
queimar dinheiro em loop; sem roteamento, tudo roda no modelo mais caro.
Por baixo: um loop de agente reenvia a conversa a cada turno, e os
descontos que as APIs oferecem — cache de prefixo, lote — exigem que o
prompt seja desenhado para eles.

## Os caminhos que pesamos

**Um modelo forte para tudo.** Rejeitado: um teto na margem.

**Rotear pelo tamanho do prompt.** Rejeitado: a natureza da tarefa decide.

**Só um roteador.** Rejeitado: não resolve o reenvio da transcrição.

**Compactação como jeito de retomar uma demanda.** Rejeitado: reconstruir
o contexto a partir dos eventos é mais barato.

## O que escolhemos, e por quê

**Medir.** Todo uso de modelo emite um evento de custo — tokens, leituras e
escritas de cache, modelo, demanda, thread, conta — no log da demanda; um
*miss* recorrente num prefixo estável é um alerta.

**Limitar.** Um orçamento por demanda com corte suave: no estouro a demanda
pausa e abre um item de atenção; o agente é informado do teto e se regula.
O card de um subagente carrega a sua fatia.

**Rotear.** Um roteador escolhe uma classe e um esforço por tipo de
trabalho — barato para trabalho mecânico, médio para investigação, forte
para planejamento e implementação — e o catálogo do provedor ativo resolve
o modelo concreto. **O crítico nunca é roteado mais barato**: modelo forte
com esforço máximo, porque economizar no freio volta como um pull request
rejeitado.

**Desenhar para o cache.** O prompt é disposto como sistema, ferramentas,
pacote de contexto e depois a conversa; o pacote é serializado
deterministicamente para o prefixo continuar idêntico byte a byte. A saída
crua de ferramentas fica na thread do especialista; filtros rodam como
código onde possível; achados usam saídas estruturadas. Uma demanda
retomada reconstrói o contexto a partir do pacote, dos achados e de um
resumo do rastro, em vez de reenviar a transcrição. Projeções são computadas
em código, nunca por um modelo; trabalho não interativo vai para a API em
lote; ferramentas são carregadas sob demanda.

## O que custou

A tabela de roteamento é um ponto de partida até a telemetria calibrá-la. A
serialização determinística do pacote é uma restrição permanente sobre o
subsistema de contexto. A reconstrução na retomada precisa se mostrar
suficiente.

## Desde então

Dois registros foram consolidados neste em 2026-09-04. A separação entre
política (qual classe) e catálogo (qual modelo) é o que deixa as regras
valerem para todo fornecedor atrás do [port de provedor de
agente](../agent-provider-as-port/).
