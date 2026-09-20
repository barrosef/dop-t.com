---
title: "Um core em Go, um BFF em Python, e uma regra entre eles"
translationKey: "decision-0012"
adr: "0012"
adr_title: "A core in Go, a BFF in Python, and the boundary between them"
adr_file: "0012-stack-go-core-python-bff.md"
date: 2026-08-30
weight: 12
group: "foundations"
description: "Go para estado, transações, eventos e o runtime do agente; Python para a borda que fala com o cockpit e a CLI. A fronteira cabe numa frase: o BFF não tem banco nem segredo."
related: ["0001", "0013", "0016"]
---

## O que estava na mesa

Duas naturezas de trabalho dividem o backend. Uma é domínio, estado e
transações: um servidor gRPC, consumidores de eventos, um daemon que fala
com o Kubernetes e — desde que o runtime passou para lá — conversas longas
com provedores de modelo. A outra é a borda: autenticar a pessoa, traduzir
protocolos para o cockpit e a CLI, agregar. Cada uma fica confortável numa
linguagem diferente.

## Os caminhos que pesamos

**Tudo em Python.** Uma linguagem; rejeitado para o core, onde um binário
único que sobe na hora em *scale-to-zero* e concorrência barata importam
mais.

**Tudo em Go.** Coerente; rejeitado para a borda, onde o ecossistema Python
de SDKs de agente e embeddings é o mais rico.

**TypeScript no backend todo**, uma linguagem com o frontend. Rejeitado pela
mesma razão do Go.

## O que escolhemos, e por quê

`dop-core` em Go: domínio, estado, transações, eventos, orquestração e o
runtime do agente; um binário com quatro modos — `serve`, `worker`, `sched`,
`launcher`. `dop-api` em Python: REST com SSE para o cockpit, gRPC para a
CLI, e um cliente gRPC para o core.

A fronteira é uma regra, e é uma das cinco invariantes sobre as quais todo
contribuidor é instruído: **o BFF não tem banco nem segredo.** Nenhuma
conexão da borda com o Postgres; nenhuma credencial no seu processo ou na
sua configuração. Quando a borda precisa registrar algo, chama o core, que
grava o estado e o evento numa transação só. A sandbox, por sua vez, fala só
com a borda e com o servidor git da plataforma.

Toda chamada da borda para o core é uma chamada de rede, então carrega um
deadline, uma política de retentativa e — nas escritas — uma chave de
idempotência.

## O que custou

Duas toolchains, dois setups de teste e lint, dois pipelines de imagem. Os
tipos existem nas duas pontas, o que só é suportável porque os dois são
gerados do mesmo contrato.

## Desde então

O runtime do agente foi colocado no core em 2026-08-31, que é também quando
"nem segredo" se juntou a "sem banco" na regra da fronteira. O raciocínio
está na [decisão do provedor de agente](../agent-provider-as-port/).
