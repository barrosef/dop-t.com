---
title: "Um contrato, gerado em todo lugar"
translationKey: "decision-0013"
adr: "0013"
adr_title: "The .proto is the contract's source of truth"
adr_file: "0013-proto-as-source-of-truth.md"
date: 2026-08-30
weight: 13
group: "foundations"
description: "Um .proto por domínio é a única descrição da API. O servidor do core, o cliente da borda, a CLI e os hooks do cockpit são gerados dele, e uma mudança incompatível derruba o build em vez de aparecer em produção."
related: ["0012", "0022"]
---

## O que estava na mesa

Quatro componentes — o core, a borda, a CLI e o cockpit — dividem uma API.
Descrito mais de uma vez, um contrato diverge: um campo renomeado de um
lado, um tipo alargado do outro, e a diferença aparece em tempo de
execução. O contrato precisava de exatamente uma fonte.

## Os caminhos que pesamos

**OpenAPI como fonte, gRPC gerado dele.** Rejeitado: o contrato do core
passaria a depender do formato da borda.

**Um contrato escrito à mão em cada ponta.** Rejeitado: isso é divergência
por desenho.

## O que escolhemos, e por quê

**Um `.proto` por domínio, versionado, é a única fonte.** Dele são gerados o
servidor e os tipos do core em Go, o cliente da borda em Python, o cliente
da CLI e — pelo documento OpenAPI que a borda publica — os hooks do
react-query e os schemas Zod do cockpit. Código gerado nunca é editado à mão;
uma mudança incompatível é recusada no CI.

Cinco convenções viajam junto:

- tudo o que é ao vivo é streaming do lado do servidor, que a borda
  transforma em SSE para o navegador;
- identificadores são referências tipadas (`AccountRef{id}`), nunca strings
  soltas;
- toda escrita carrega um `idempotency_key`, delimitado pela conta dona das
  linhas — linhas de nível de plataforma usam um namespace próprio;
- **quem está chamando, em qual conta, viaja nos metadados da chamada**,
  nunca no corpo da mensagem — resolvido por um interceptor antes de
  qualquer caso de uso rodar, e verificado conforme [a decisão dos
  chamadores](../the-core-verifies-its-callers/);
- um campo nunca muda de número nem de tipo.

## O que custou

`buf` e geração de código estão no CI desde o primeiro dia, e uma mudança de
contrato é uma mudança versionada e aditiva. Quatro repositórios regeneram
quando o contrato se move.

## Desde então

Duas convenções foram tornadas precisas depois da primeira implementação: em
2026-09-02 as mensagens de requisição perderam um campo de contexto que os
metadados já tinham substituído (o número fica reservado), e em 2026-09-06 o
escopo da chave de idempotência foi fixado na conta dona.
