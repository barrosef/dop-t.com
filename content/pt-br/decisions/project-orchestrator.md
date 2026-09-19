---
title: "O techlead é um agente, e nunca pausa uma demanda"
translationKey: "decision-0011"
adr: "0011"
adr_title: "The project orchestrator: the techlead agent"
adr_file: "0011-project-orchestrator.md"
date: 2026-08-30
weight: 11
group: "agents"
description: "Duas demandas tocando os mesmos arquivos, uma dependendo do resultado da outra — situações que o agente de uma demanda sozinho não enxerga. Alguém precisava enxergar, e a resposta errada mais barata era parar tudo até um humano decidir."
related: ["0004", "0005", "0007"]
---

## O que estava na mesa

Com demandas paralelas num projeto, aparecem situações transversais que o
agente de nenhuma demanda enxerga sozinho: duas demandas tocando os mesmos
arquivos, uma dependendo do resultado da outra, uma mudança de comportamento
numa que quebra a premissa da outra. A decisão de entrega tinha previsto que
"o orquestrador enxerga a sobreposição" sem dizer quem era o orquestrador.

## Os caminhos que pesamos

**Regras estáticas de detecção** — um diff de caminhos mais um grafo de
dependências declarado. Ficam como sensores, mas não bastam sozinhas:
interferência de comportamento não aparece num caminho; precisa de uma
leitura semântica das specs e dos diffs.

**Pausar as demandas em risco até uma decisão.** Rejeitado com ênfase. Mata o
paralelismo que é requisito e transforma a detecção, que é barata, num
bloqueio, que é caro.

**Um techlead humano.** É o jeito de todo mundo hoje — e é exatamente a
atenção escassa que a plataforma existe para poupar.

## O que escolhemos, e por quê

Todo projeto tem um **agente orquestrador, o techlead dos agentes de
demanda**. Ele acorda quando o projeto tem duas ou mais demandas ativas e
dorme no resto do tempo; vive na plataforma, não na sandbox de nenhuma
demanda, e observa pelo log de eventos, pelo estado dos branches e pelos
fluxos das demandas.

**Autonomia primeiro.** Ao identificar uma situação transversal, ele planeja
soluções e leva à caixa de atenção uma proposta de decisão — opções prontas,
com uma recomendação — nunca um alarme cru. A escolha do desenvolvedor vira
uma **diretiva de coordenação**, um evento que aparece nas threads envolvidas.
O exemplo canônico: "a demanda 1 depende da demanda 0" — então, quando a 0
commitar o que a 1 precisa, a 1 faz cherry-pick do branch da 0 e segue.

**A regra de ouro: uma situação transversal identificada nunca pausa uma
demanda.** A demanda 1 vai até onde consegue; quando a condição da diretiva
é atendida, aplica a coordenação e continua. Um bloqueio só existe quando a
própria demanda esgota o que dá para fazer sem a condição — e então é um
bloqueio dela, visível na caixa.

O vocabulário inicial: sequenciamento com cherry-pick ou rebase, uma ordem
preferida na fila de merge, partição de arquivos ("a 2 não toca o módulo X até
a 1 mergear"), verificação cruzada. Extensível — o techlead propõe, o
vocabulário só nomeia.

## O que custou

O custo de modelo do techlead: observar é barato, planejar é caro e é roteado
como investigação; ele acorda por evento, não por polling. E uma diretiva de
coordenação é estado novo entre demandas — precisa aparecer na linha do tempo
e nas duas threads, ou vira mágica invisível.

## Desde então

Nada a contradisse, e nada a construiu ainda: o techlead pertence à parte da
plataforma que executa em vez de modelar, que o roadmap coloca depois das
ferramentas do agente. O que foi construído desde então — fluxos como dado,
reações como dado — é o vocabulário em que uma diretiva será escrita.
