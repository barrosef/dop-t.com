---
title: "Três camadas de conhecimento, um pacote curado por demanda"
translationKey: "decision-0006"
adr: "0006"
adr_title: "Context is a subsystem: a knowledge base and a package per demand"
adr_file: "0006-context-as-subsystem.md"
date: 2026-08-29
weight: 6
group: "knowledge"
description: "O conhecimento de um projeto são regras, um índice do seu código e uma memória de demandas passadas. Cada demanda recebe um pacote curado montado a partir deles, e a estante inteira é clonada na sandbox para o agente abrir o que precisar."
related: ["0001", "0004", "0007", "0021"]
---

## O que estava na mesa

A utilidade de um agente depende das regras do projeto, do mapa do seu
código e da memória do que demandas anteriores acharam. Esse conhecimento
precisa ter permissão por conta e projeto, ser alcançável da sandbox e — a
parte que decide a qualidade — ser montado por demanda em vez de despejado.

## Os caminhos que pesamos

**Uma pasta crua de documentos como único mecanismo.** Rejeitada: sem
curadoria o agente cava.

**Tudo no prompt.** Rejeitado: cresce com o projeto, não com a demanda.

**Um serviço de RAG externo por cliente.** Adiado; o port permite depois.

## O que escolhemos, e por quê

**Uma base de conhecimento por projeto, em três camadas:** *regras* — as
convenções que o agente obedece; *índice* — um mapa por repositório: o que
mora onde, como construir, como testar; *memória* — achados e lições de
demandas passadas. O texto mora no repositório raiz do projeto, em `rules/`,
`index/`, `memory/` e `demand/<id>/`; artefatos binários moram no
armazenamento de objetos, referenciados do repositório.

**Um pacote de contexto por demanda** é montado quando a sandbox é
provisionada — a spec, as regras, o índice dos repositórios envolvidos, as
memórias relevantes — serializado deterministicamente para o prefixo do
prompt continuar cacheável. **O pacote é curado; o repositório é a
estante.** O repositório raiz inteiro é clonado na sandbox com um manifesto
gerado, para o agente abrir o que precisa sem pagar por isso a cada turno.

**Retorno no fechamento:** os achados e as lições de uma demanda são
commitados em `memory/`. **O índice é regenerado no evento de merge**, não
por agenda.

## O que custou

Montar o pacote e julgar a relevância de uma memória é trabalho do
orquestrador. Permissões por conta e projeto no armazenamento são cobertas
pelas suítes de contrato do repositório e do armazenamento de objetos.

## Desde então

O texto mudou do armazenamento de objetos para o repositório raiz do projeto
em 2026-09-03, quando [o repositório de
conhecimento](../project-knowledge-as-a-git-repository/) foi decidido; as
três camadas e o pacote não mudaram.
