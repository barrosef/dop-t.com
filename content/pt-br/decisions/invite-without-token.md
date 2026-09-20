---
title: "Um convite é um endereço, não uma chave"
translationKey: "decision-0019"
adr: "0019"
adr_title: "The invite has no secret: identity in place of a bearer"
adr_file: "0019-invite-without-token.md"
date: 2026-09-01
weight: 19
group: "identity"
description: "Um convite não carrega segredo. O link é o id da linha, seguro para viajar em eventos, projeções e e-mails, porque aceitar exige entrar com o e-mail verificado para o qual o convite foi enviado."
related: ["0002", "0004", "0018", "0020"]
---

## O que estava na mesa

O payload de um evento é replicado em quatro lugares — a tabela de eventos,
a outbox, o stream do broker e a projeção da linha do tempo, que é feita
para ser exibida. Nada que conceda acesso sozinho pode viajar num evento. O
e-mail do convite, produzido por um consumidor que só vê o evento, ainda
precisa de um link que a pessoa convidada — que ainda não é usuária —
consiga abrir.

## Os caminhos que pesamos

**Um token secreto no evento.** Rejeitado: uma credencial em repouso,
replicada em quatro lugares, sem revogação que alcance o stream ou a linha
do tempo.

**Um canal lateral até o notificador**, fora do log. Rejeitado: um segundo
mecanismo de entrega, e o segredo ainda existiria.

**Um UUID indexando o token**, com só o índice no evento. Rejeitado
sozinho: se ter o índice basta para aceitar, o índice é a credencial.

**Manter o token e também exigir que o e-mail bata.** Rejeitado: mantém o
hash de um segredo que ninguém checa.

## O que escolhemos, e por quê

**O convite não tem segredo.** É endereçado pelo `id` da linha, que pode
aparecer em eventos, projeções, e-mails e logs, porque sozinho não concede
nada.

**Aceitar exige ser o convidado.** A aceitação recusa quando não há sessão,
quando o convite não está mais utilizável, quando o e-mail de quem está
logado não está verificado, e quando esse e-mail verificado não é o do
convite. As duas últimas são recusas separadas — "confirme o seu e-mail" e
"este convite não é seu" mandam a pessoa fazer coisas diferentes — e a
segunda nunca revela para quem era o convite.

O link do e-mail é `/invites/{invite_id}`, produzido pela regra de
notificação como dado: um placeholder que não pode ser resolvido remove o
link em vez de entregar meio link. O convite expira em catorze dias.

## O que custou

Alguém convidado num endereço que entra com outro — uma conta pessoal contra
uma corporativa — não consegue aceitar; a mensagem precisa dizer isso. E um
provedor de identidade que não informa se um e-mail está verificado faz toda
aceitação falhar do jeito seguro.

## Desde então

A tela de aceitação foi construída no dia seguinte, junto com o segundo
fator. O mesmo princípio — nenhum *bearer* viajando numa caixa de entrada — é
a razão de o segundo fator usar códigos digitados na sessão em vez de *magic
links*.
