---
title: "Um canal sem gatilho é um gatilho difuso"
translationKey: "decision-0018"
adr: "0018"
adr_title: "Communication: the trigger and the channel are born together"
adr_file: "0018-communication-trigger-and-channel.md"
date: 2026-08-31
weight: 18
group: "events"
description: "O risco nunca foi um mailer sem nada que o disparasse. Era o oposto: se o gatilho não é desenhado, todo caso de uso que manda e-mail vira um, sem nome e sem lugar."
related: ["0004", "0014", "0019", "0020"]
---

## O que estava na mesa

A plataforma precisa avisar as pessoas fora do cockpit: um convite, uma
verificação de conta, uma integração quebrada, um orçamento estourado, uma
thread esperando resposta.

Um projeto irmão resolve isso gravando um documento numa coleção, que dispara
uma função, que envia pelo SendGrid. Três coisas de lá se transferem: o ensaio
local sem chave (imprime em vez de enviar), o estado guardado no registro, e
templates versionados no repositório. O gatilho não se transfere — aqui a
espinha de eventos já existe, e a spec diz que comunicação é *um consumidor da
espinha de eventos, não um sistema à parte*.

## O que escolhemos, e por quê

O idealizador resumiu numa frase: *"Um Mailer vivendo sem o Notifier seria
como uma bala que pudesse ser disparada sem o gatilho."* O ponto não é que o
canal *pode* viver sozinho. É que o gatilho existe de qualquer jeito: se
ninguém o desenha, o caso de uso do convite chama o mailer direto e *vira* o
gatilho, sem nome e sem lugar, espalhado por tantos casos de uso quantos
enviem e-mail. O risco nunca foi um canal sem gatilho. Era um gatilho
**difuso**.

Então: um consumidor de eventos, o **Notifier**, decide *o que* notificar e
*para quem*; produz um comando — tipo, destinatário, dados — e o **Mailer**,
um port de canal, dispara. O caso de uso do convite não conhece nenhum dos
dois; publica um evento e só.

**O port é por canal, não um para tudo.** E-mail tem assunto, HTML e anexos;
push tem título, badge e deep link; SMS tem 160 caracteres e nenhuma
formatação. Um port só carregaria a união de tudo ou o menor denominador
comum. Então `Mailer` agora, `Pusher` e `SMSer` quando existirem push e SMS;
um fornecedor pode implementar vários.

**O adaptador é grande: índice, resolução e envio.** Isso corrigiu a
proposta inicial, que renderizava templates no domínio. O port fala intenção
— "um convite foi criado, para este endereço, com estes dados" — e cada
adaptador decide no que isso vira: o SendGrid mapeia o tipo para um id de
template; o SMTP renderiza localmente dos arquivos do repositório. Renderizar
no domínio pareceria mais limpo e seria pior: a plataforma nunca poderia usar
o editor de templates de um provedor, e o port carregaria um bloco de HTML. É
a mesma separação que o roteador de custo faz — política aqui, catálogo ali.
Uma consequência precisou de teste: um tipo pode existir na política e não
ter template no fornecedor, e isso falha em silêncio. A suíte de contrato faz
todo adaptador resolver todo tipo que o domínio consegue emitir.

**Dois adaptadores reais, SendGrid e SMTP.** O SMTP é o caminho
auto-hospedado, e é o que *força* a resolução local de templates — prova de
que o port fala intenção e não o id de template de um fornecedor.

**Transacional e atenção são diferentes.** Um convite dispara na hora,
sempre, um por evento. Uma thread bloqueada ou um PR esperando revisão já tem
um mapa — a caixa de atenção — e o e-mail se engancha na caixa, não nos
eventos crus. E então o risco é spam: um e-mail por item torna uma caixa de
entrada inútil. Então um **digest com atraso**, quinze minutos por padrão: o
item abre, espera, e só vira e-mail se ainda estiver aberto. Quinze é um
palpite informado, a calibrar com telemetria.

**A idempotência é por (evento, regra, ação)**, não por evento. Com uma ação
por evento, uma chave só no evento funciona; no dia em que um evento disparar
várias ações, ela descartaria a segunda como duplicata — em silêncio, por
desenho. A chave nasceu composta para um futuro que já estava planejado.

## O que custou

O editor visual do SendGrid se perde para os templates SMTP, que são
arquivos. O template do mesmo aviso vive em dois lugares enquanto os dois
adaptadores existirem. E os quinze minutos são um palpite: um aviso urgente
demais espera, um trivial demais irrita, e só os dados resolvem.

## Desde então

O vocabulário se pagou duas vezes. A decisão do convite
([ADR-0019](../invite-without-token/)) mudou o link que a regra carrega, e
nada mais. O segundo fator ([ADR-0020](../second-factor-in-the-core/)) usou o
*canal* e deliberadamente não o *gatilho*: um código é um desafio, não uma
notificação, e não pode passar por digests e atrasos. Um terceiro adaptador —
o OneSignal, o que o registro tinha anotado como "futuro" — foi escrito em
setembro, e o fluxo de verificação de e-mail foi provado até a porta do
provedor.
