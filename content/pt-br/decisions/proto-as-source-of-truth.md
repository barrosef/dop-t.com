---
title: "Um contrato que nasceu triplicado"
translationKey: "decision-0013"
adr: "0013"
adr_title: "The .proto is the contract's source of truth"
adr_file: "0013-proto-as-source-of-truth.md"
date: 2026-08-30
weight: 13
group: "foundations"
description: "Três descrições da mesma API, nenhuma com autoridade, divergindo antes de haver produto. Um arquivo virou a verdade — e um campo nele que não fazia nada nos ensinou algo."
related: ["0012", "0022"]
---

## O que estava na mesa

O contrato já existia em três lugares antes de qualquer um deles funcionar: um
`types.ts` escrito à mão no frontend, um `openapi.yaml` vazio com geração de
código rodando sobre nada, e a promessa de um `.proto` para o core. Três
fontes, nenhuma delas a autoridade. A divergência tinha começado antes de
existir um produto para divergir.

## Os caminhos que pesamos

**OpenAPI como fonte, gRPC gerado dele.** Inverte a dependência: o contrato
interno — o do core — passaria a depender do formato da borda. Rejeitado.

**Contratos escritos à mão nas duas pontas.** Era o estado atual, e era o
problema.

## O que escolhemos, e por quê

Um `.proto` por domínio, versionado, é a única fonte. Dele geramos o servidor
e os tipos do core em Go, o cliente da borda em Python, o cliente da CLI e os
tipos do frontend. O documento OpenAPI da borda é ele mesmo um produto da borda,
que é um produto do proto.

Cinco convenções vieram junto, e duas importam mais que as outras. Tudo o que
é ao vivo é streaming do lado do servidor — a borda transforma em SSE para o
navegador. Toda escrita carrega um `idempotency_key`, porque com eventos e
retentativas isso é requisito, não luxo. E **quem está chamando, em qual conta,
viaja nos metadados, não no corpo** — resolvido por um interceptor antes de
qualquer caso de uso rodar. Contexto é transversal: no corpo, cada RPC teria
que lembrar de checar, e o que esquecesse seria um buraco de isolamento. A
compatibilidade é verificada no CI: um campo nunca muda de número nem de tipo.

## O que custou

`buf` e geração de código entram no CI no primeiro dia. Uma mudança de contrato
exige disciplina — aditiva por padrão — e um build agora falha onde uma
divergência antes seria uma descoberta em produção.

## Desde então

Duas coisas aconteceram que o texto original não poderia prever, e as duas
agora fazem parte da decisão.

A primeira foi uma dívida. As mensagens de requisição carregavam um
`CallContext ctx = 1` de uma tentativa anterior, e o servidor o ignorava. Um
contrato que declara um campo sem efeito ensina a coisa errada: o cliente
acredita que está delimitando a chamada quando não está. Ele já tinha se
copiado sozinho para todo proto novo, o do segundo fator incluído. Em
2026-09-02 o campo saiu dos doze protos, o número 1 foi reservado em toda
mensagem que o carregava, o tipo sumiu do `common.proto`, e dois testes que
afirmavam que o campo estava no corpo passaram a afirmar o contrário — que é o
que o impede de voltar.

A segunda foi sutil, sobre o *namespace* da chave de idempotência. Uma tabela
tornou a chave única entre todas as linhas; outra a delimitou por conta. A
chave é fornecida pelo cliente e guardada literalmente, então um namespace da
tabela inteira deixa a conta B enviar uma chave que a conta A já usou, colidir
com uma linha que B não pode ver, e receber essa colisão de volta. Uma tabela
entregou isso uma vez. A regra que saiu daí: delimite a chave pelo que é dono
das linhas, e onde um nível não tem dono, dê a ele um namespace próprio — uma
decisão que o autor da próxima tabela deve tomar de propósito, não copiando o
vizinho que abriu primeiro.
