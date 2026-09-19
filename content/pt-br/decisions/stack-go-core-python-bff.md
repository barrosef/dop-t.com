---
title: "Duas linguagens, um dono da verdade"
translationKey: "decision-0012"
adr: "0012"
adr_title: "A core in Go, a BFF in Python, and the boundary between them"
adr_file: "0012-stack-go-core-python-bff.md"
date: 2026-08-30
weight: 12
group: "foundations"
description: "Go para o estado e as transações, Python para a borda — e uma fronteira numa frase: o BFF não tem banco. Uma das cláusulas não sobreviveu ao contato com o código."
related: ["0001", "0013", "0016"]
---

## O que estava na mesa

Duas naturezas de trabalho pediam para morar no mesmo backend. Uma é domínio,
estado e transações: identidade, recursos, fluxos, demandas, entrega, custo —
um servidor gRPC, consumidores de eventos, um daemon conversando com o
Kubernetes. A outra é conversa com modelos de IA: sessões longas, streaming de
tokens, ferramentas, embeddings. Forçar as duas numa linguagem só cobra em
algum lugar. gRPC e concorrência em Python são desconfortáveis; o ecossistema
de agentes e embeddings em Go é raso.

## Os caminhos que pesamos

**Tudo em Python.** Continuidade com o que já existia, uma linguagem só.
Rejeitado para o core: um binário único que sobe na hora em *scale-to-zero*,
concorrência barata, um daemon do Kubernetes — é trabalho em que Go é
materialmente melhor.

**Tudo em Go.** Coerente no backend, mas a borda perderia o ecossistema de IA
— os SDKs de agente, os embeddings, os tokenizadores — que é o ponto inteiro do
produto.

**TypeScript no backend todo**, uma linguagem com o frontend. Rejeitado pela
mesma razão do Go: as bibliotecas de agente mais completas são em Python.

## O que escolhemos, e por quê

`dop-core` em Go: domínio, estado, transações, eventos, orquestração; fala gRPC
com quem o chama e Postgres, NATS e a API do Kubernetes com o ambiente; um
binário com quatro modos. `dop-api` em Python: protocolo, a sessão do agente,
a conversa com os modelos; REST com SSE para o cockpit, gRPC para a CLI, e um
cliente gRPC para o core.

A fronteira cabe numa regra, e a regra é a decisão: **o BFF não tem banco.**
Nem "só uma consulta rápida". Dois donos de um schema é como uma fronteira
morre. Quando a borda precisa registrar algo, ela chama o core, que grava o
estado e o evento na mesma transação.

## O que custou

Duas toolchains, dois setups de lint e teste, dois pipelines de imagem. Toda
chamada da borda para o core é uma chamada de rede — precisa de deadline,
retentativa e idempotência, então o contrato carrega um `idempotency_key` em
toda escrita. Os tipos existem nas duas pontas, o que só é suportável porque
os dois são gerados da mesma fonte ([a decisão seguinte](../proto-as-source-of-truth/)).

## Desde então

Uma cláusula desta decisão estava errada e foi substituída. O texto original
punha o runtime do agente — a peça que fala com o modelo — na borda em Python:
*"o core decide o quê; o BFF roda a conversa"*. Parecia limpo no papel. Na
implementação, descobriu-se que o runtime precisa da credencial do provedor do
modelo, e a credencial mora no vault, no core, que nunca entrega um segredo. O
runtime mudou para o core, e a fronteira ganhou a sua segunda metade: o BFF
não tem banco **nem segredo**. Essa história é a da
[ADR-0016](../agent-provider-as-port/). O que ficou intacto aqui é a divisão
de linguagens e a frase sobre o banco.
