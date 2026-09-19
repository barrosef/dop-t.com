---
title: "O convite que não podia carregar a própria chave"
translationKey: "decision-0019"
adr: "0019"
adr_title: "The invite has no secret: identity in place of a bearer"
adr_file: "0019-invite-without-token.md"
date: 2026-09-01
weight: 19
group: "identity"
description: "Um token que não podia aparecer em lugar nenhum em repouso, um e-mail que precisava de um link, e quatro lugares onde todo evento pousa. A saída foi parar de ter um segredo."
related: ["0002", "0004", "0018", "0020"]
---

## O que estava na mesa

O convite nasceu do jeito de sempre: um token opaco, gerado uma vez, devolvido
a quem chamou, guardado no banco só como hash. A aceitação checava o hash e
nada mais. Quem tivesse o token entrava na conta.

Isso é uma credencial *bearer*, e travou o produto de um jeito que não
tínhamos previsto. O token não podia aparecer em lugar nenhum onde ficasse em
repouso — e "lugar nenhum" aqui é grande, porque o payload de `invite.created`
viaja para quatro destinos: a tabela de eventos (particionada, indefinida), a
outbox (até ser drenada), o JetStream (trinta dias em disco) e a projeção da
linha do tempo, que guarda o payload inteiro e é *feita para ser exibida*.
Colocar o token no evento replicaria uma credencial em quatro lugares, um
deles uma tela. Deixá-lo fora significava que o notificador — que só vê o
evento — não tinha como montar um link de aceitação. O e-mail anunciava um
convite e apontava para uma lista que o convidado, que ainda não é usuário,
não conseguia abrir.

## Os caminhos que pesamos

**O token no evento.** Uma credencial em repouso, quatro vezes, sem revogação
que alcance o JetStream ou a linha do tempo.

**Um caminho lateral até o notificador.** Um segundo mecanismo de entrega,
existindo só para não usar o primeiro — e o segredo ainda existiria, em menos
lugares.

**Indexar o token por um UUID e enviar o índice.** A proposta do idealizador, e
apontava na direção certa — mas sozinha não muda nada: se ter o UUID basta
para aceitar, o UUID *é* a credencial, nos mesmos quatro lugares.

**Manter o token, e também exigir que o e-mail bata.** Funciona, e mantém o
hash de um segredo que ninguém mais checa: uma superfície sem dono.

## O que escolhemos, e por quê

A segunda metade do idealizador foi a chave: *"o e-mail tem que bater
também."* Quando a aceitação exige ser o convidado, o link deixa de ser uma
credencial e vira um **endereço**. Então o convite parou de ter segredo. A
coluna do token foi removida, os geradores sumiram, e criar um convite não
devolve nada — não há nada para devolver. O que endereça um convite é o id da
linha, que viaja em texto claro no evento, na linha do tempo, no e-mail e no
log, porque sozinho não concede nada.

A aceitação recusa quando não há sessão, quando o convite não está utilizável,
quando o e-mail de quem está logado **não está verificado**, e quando esse
e-mail verificado **não é o do convite**. A última mudou a natureza da coisa —
e fechou um buraco que não tinha nada a ver com o token: antes, *qualquer*
usuário autenticado com o link recebia o papel concedido a outra pessoa. As
duas recusas novas são separadas de propósito ("confirme o seu e-mail" e
"este convite não é seu" mandam a pessoa fazer coisas diferentes), e a segunda
nunca diz para quem era o convite — dizer transformaria o link num oráculo, e
o segredo voltaria pela porta dos fundos.

O link do e-mail continua sendo dado, não código: a regra de notificação
carrega `/invites/{invite_id}`, resolvido contra os mesmos dados que o template
recebe; um teste recusa um placeholder que não está nos dados, e em execução
um campo ausente apaga o link inteiro em vez de entregar meio link.

## O que custou

Alguém convidado num endereço que entra com outro — uma conta Google pessoal
contra uma corporativa — não consegue aceitar. Está correto, o convite é para
uma pessoa, mas é uma parede nova e a mensagem dela precisa ser boa. E um
provedor de identidade que não informa se um e-mail está verificado faz toda
aceitação falhar do jeito seguro: com barulho.

## Desde então

O padrão fixado aqui — nenhum *bearer* viajando numa caixa de entrada — foi o
que o segundo fator ([ADR-0020](../second-factor-in-the-core/)) usou no dia
seguinte para recusar *magic links*: um código precisa ser digitado na sessão
que o pediu. A tela de aceitação para a qual o link aponta não existia quando
isto foi decidido; foi construída no dia seguinte, junto com o segundo fator.
