---
title: "Um agente techlead por projeto, e ele nunca pausa uma demanda"
translationKey: "decision-0011"
adr: "0011"
adr_title: "The project orchestrator: the techlead agent"
adr_file: "0011-project-orchestrator.md"
date: 2026-08-30
weight: 11
group: "agents"
description: "Quando um projeto tem duas ou mais demandas ativas, um agente orquestrador observa sobreposição, dependências e interferência, planeja uma solução e leva ao desenvolvedor uma decisão — sem parar nenhuma demanda enquanto espera."
related: ["0004", "0005", "0007"]
---

## O que estava na mesa

Demandas paralelas num projeto produzem situações que o agente de nenhuma
demanda sozinho observa: dois branches tocando os mesmos arquivos, uma
demanda dependendo do resultado de outra, uma mudança de comportamento numa
que quebra a premissa da outra. Algo precisa enxergar através das demandas,
e detectar não pode virar bloquear.

## Os caminhos que pesamos

**Só regras estáticas** — um diff de caminhos mais um grafo de
dependências declarado. Mantidas como sensores; insuficientes para
interferência de comportamento, que precisa de uma leitura semântica de
specs e diffs.

**Pausar as demandas em risco até uma decisão.** Rejeitado: serializa o
paralelismo para o qual a plataforma existe.

**Um techlead humano.** Rejeitado: é a atenção que a plataforma existe para
poupar.

## O que escolhemos, e por quê

**Todo projeto tem um agente orquestrador** — o techlead dos agentes de
demanda. Ele acorda quando o projeto tem duas ou mais demandas ativas, roda
na plataforma em vez de numa sandbox, e observa o log de eventos, o estado
dos branches e os fluxos das demandas.

**Ele planeja antes de perguntar.** Ao detectar uma situação transversal,
produz opções com uma recomendação e abre um item de atenção, nunca um
alarme cru. A escolha do desenvolvedor vira uma **diretiva de
coordenação**, um evento visível nas threads envolvidas: sequenciamento com
cherry-pick ou rebase entre branches, uma ordem preferida na fila de merge,
partição de arquivos, verificação cruzada. O vocabulário é extensível.

**Uma situação detectada nunca pausa uma demanda.** A demanda segue até onde
consegue; quando a condição da diretiva é atendida, aplica a coordenação e
continua. Um bloqueio só existe quando a própria demanda esgotou o que dá
para fazer, e então é um item de atenção dela.

## O que custou

Observar é barato e dirigido por eventos; planejar é roteado como
investigação. As diretivas são estado novo entre demandas e precisam
aparecer na linha do tempo e nas duas threads.

## Desde então

O techlead pertence à parte da plataforma que executa em vez de modelar,
que o roadmap coloca depois das ferramentas do agente; fluxos e reações
como dado são o vocabulário em que uma diretiva será escrita.
