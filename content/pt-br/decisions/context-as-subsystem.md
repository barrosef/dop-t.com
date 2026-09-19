---
title: "Armazenar é a metade fácil"
translationKey: "decision-0006"
adr: "0006"
adr_title: "Context is a subsystem: a knowledge base and a package per demand"
adr_file: "0006-context-as-subsystem.md"
date: 2026-08-29
weight: 6
group: "knowledge"
description: "Os requisitos diziam 'contexto criado pelo Claude' e 'as regras do workspace' sem entidade, sem port, sem mecanismo. O que separa um agente útil de um inútil é o que entra e como é montado — não onde é guardado."
related: ["0001", "0004", "0007", "0021"]
---

## O que estava na mesa

O que separa um agente útil de um inútil é contexto: as regras do projeto, o
mapa do código, a memória do que já foi tentado. Nos requisitos isso aparecia
como "contexto criado pelo Claude" e "as regras do workspace" — sem entidade,
sem port, sem mecanismo. A diretriz do produto era explícita sobre o
armazenamento: seguro, disponível, com permissões, alcançável das microVMs,
"para que os agentes trabalhem de forma genuinamente inteligente".

Armazenar é a metade fácil. A metade que gera inteligência é *o que* está lá
e *como* é montado por demanda.

## Os caminhos que pesamos

**Uma pasta crua de documentos na sandbox.** Rejeitada: sem curadoria e sem
montagem, o agente cava, e cavar é o que um pacote existe para eliminar.

**Tudo embutido no prompt.** Rejeitado: estoura a janela de contexto e cresce
com o projeto, não com a demanda.

**Um serviço de RAG externo por cliente.** Adiado: o port permite plugar um
depois; começar por aí é comprar infraestrutura antes de ter conteúdo.

## O que escolhemos, e por quê

**Uma base de conhecimento por projeto**, versionada, em três camadas.
*Regras*: as convenções que o agente obedece. *O índice*: o mapa do código —
o que mora onde, como construir, como testar; sem ele, toda demanda gasta a
primeira meia hora redescobrindo o repositório. *Memória*: achados e lições
de demandas passadas, decisões, leituras forenses.

**Um pacote de contexto por demanda**, montado quando a sandbox é
provisionada: a spec, as regras, o índice dos repositórios envolvidos, as
memórias relevantes. A bagagem de mão do agente — curada, não despejada. **Um
retorno no fechamento**: os achados e as lições da demanda vão para a camada
de memória. Contexto é um ciclo, não um arquivo. E **o índice atualiza no
evento de merge**, não por agenda: o mapa segue o `main` real.

## O que custou

Curadoria é trabalho de verdade: montar o pacote e julgar a relevância de uma
memória cabe ao orquestrador. E armazenamento por conta com permissão fina é
mais uma superfície para os testes de contrato cobrirem.

## Desde então

A "pasta crua" rejeitada voltou, e o registro explica por que isso não é
contradição. Em 2026-09-03, a [ADR-0021](../project-knowledge-as-a-git-repository/)
deu às três camadas uma casa: o repositório raiz do projeto, um servidor git
que a plataforma roda, clonado em toda sandbox. O que tinha sido rejeitado era
uma pasta *como substituta* do pacote. O pacote ficou exatamente como definido
aqui — a bagagem curada, com orçamento, que vai para o prompt — e o
repositório foi *adicionado* como a estante: completo, com um manifesto
gerado para o agente não cavar. O pacote é pago a cada turno; a estante não
custa nada até um arquivo ser aberto. E "sobre o armazenamento de objetos"
mudou: texto mora no git, que dá atribuição e histórico; o bucket guarda
bytes — diagramas, exportações, o que um repositório faz mal.
