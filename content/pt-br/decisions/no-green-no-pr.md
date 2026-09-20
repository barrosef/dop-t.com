---
title: "Sem verde, sem PR — e uma fila de merge depois do verde"
translationKey: "decision-0005"
adr: "0005"
adr_title: "No green, no PR: native verification, and a merge queue per repository"
adr_file: "0005-no-green-no-pr.md"
date: 2026-08-29
weight: 5
group: "delivery"
description: "Um pull request só abre quando a aceitação executável passa e um crítico independente revisou, carregando a evidência. Depois do verde, uma fila por repositório reverifica cada PR contra o main atual e mergeia um de cada vez."
related: ["0004", "0008", "0011", "0023"]
---

## O que estava na mesa

Os agentes entregam pull requests mais rápido do que humanos conseguem
revisar, e demandas paralelas num repositório produzem PRs verificados
contra um `main` desatualizado — o primeiro merge pode quebrar os outros em
silêncio. Um merge é decisão humana, por regra do produto; o que acontece
antes de o humano ser chamado, e depois de o PR ficar verde, cabe à
plataforma decidir.

## Os caminhos que pesamos

**Revisão só por humanos.** Rejeitada: a capacidade de revisão é o gargalo.

**Merge automático no verde.** Rejeitado: o portão humano é regra do
produto.

**Merge otimista, na ordem de chegada.** Rejeitado: quebras semânticas
chegam ao `main`.

**Um arquivo, um dono.** Rejeitado: serializa o paralelismo exigido.

**Só a fila de merge do provedor.** Rejeitada como rota única: não é
universal, e não tem visão entre demandas.

## O que escolhemos, e por quê

**Antes do pull request, quatro regras.** Os critérios de aceitação são
executáveis e vivem na spec da demanda — suítes de teste e checagens
derivadas dela. Nenhum PR abre com a aceitação falhando; o agente itera até
o verde, e uma falha persistente vira uma pergunta ao humano. Um crítico
independente — modelo forte, contexto limpo, esforço máximo — revisa o
diff, a spec e a evidência antes do humano. O PR carrega o pacote de
evidências, para o humano revisar a exceção, não a regra.

**Depois do verde, uma fila de merge por repositório**, como conceito de
domínio. Um PR verde entra na fila; a fila reaplica cada PR sobre o `main`
atual, reexecuta a verificação e mergeia um de cada vez. Um conflito é
tarefa do agente da demanda primeiro, depois do humano. A sobreposição
entre demandas é detectada antes do PR pelo orquestrador do projeto. A fila
nativa do provedor é usada onde existe; a da plataforma orquestra por cima.

## O que custou

O crítico custa tokens, sem economia permitida. Os merges são serializados
por repositório, então a posição na fila e a previsão aparecem no cockpit.
Reverificar a cada posição custa computação. A sintaxe dos critérios
executáveis é definida na spec de workflow.

## Desde então

Dois registros — as regras antes do PR e a fila depois dele — foram
consolidados em 2026-09-04. O artefato de spec ganhou endereço no
repositório de conhecimento do projeto, e a verificação ganhou o seu runner
na [decisão de verificação](../verification-runs-from-source/).
