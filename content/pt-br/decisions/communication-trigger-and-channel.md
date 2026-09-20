---
title: "Um notificador decide; um canal entrega"
translationKey: "decision-0018"
adr: "0018"
adr_title: "Communication: the trigger and the channel are born together"
adr_file: "0018-communication-trigger-and-channel.md"
date: 2026-08-31
weight: 18
group: "events"
description: "Um consumidor da espinha de eventos decide o que notificar e para quem, a partir de uma tabela de regras. Um port por canal entrega, e cada adaptador — OneSignal, SendGrid, SMTP — resolve os próprios templates. Os casos de uso publicam eventos e nada mais."
related: ["0004", "0014", "0019", "0020"]
---

## O que estava na mesa

A plataforma avisa as pessoas fora do cockpit: um convite, uma verificação
de conta, uma integração quebrada, um orçamento estourado, uma thread
esperando resposta. A espinha de eventos já existe; comunicação é um
consumidor dela. Duas coisas precisavam ser decididas: quem decide que um
aviso sai, e que forma a entrega toma em canais que diferem — e-mail tem
assunto e HTML, SMS tem 160 caracteres.

## Os caminhos que pesamos

**Casos de uso chamando um mailer diretamente.** Rejeitado: a decisão "o
que notificar" ficaria espalhada por todo caso de uso que envia e-mail, sem
nome e sem lugar.

**Um port para todos os canais.** Rejeitado: carregaria a união dos campos
de todos os canais, ou o menor denominador comum.

**Renderizar templates no domínio.** Rejeitado: o port carregaria HTML, e o
editor de templates de um provedor nunca poderia ser usado.

**Um e-mail por item de atenção.** Rejeitado: ruído.

## O que escolhemos, e por quê

**Dois componentes.** O **Notifier**, um consumidor no worker, decide o que
notificar e para quem a partir de uma tabela de regras — tipo de evento,
tipo de aviso, resolução do destinatário, dados, caminho do link — e emite
um comando. Um **port de canal** entrega: `Mailer` para e-mail, `SMSer`
para SMS, `Pusher` quando existir push. Os casos de uso só publicam
eventos.

**O índice de templates, a resolução e a renderização vivem no
adaptador.** O port fala intenção — um tipo e os seus dados — e cada
adaptador o mapeia para um template: o SendGrid para um id de template, o
SMTP e o OneSignal para templates no repositório. A suíte de contrato faz
todo adaptador resolver todo tipo que o domínio emite, então um template
ausente é um teste falhando em vez de uma não-entrega silenciosa. Sem
credencial configurada, o adaptador imprime em vez de enviar.

**Avisos transacionais** — um convite, uma verificação — saem na hora, um
por evento. **Avisos de atenção** — uma thread bloqueada, um PR esperando
revisão, um orçamento estourado — se engancham na caixa de atenção e saem
como um **digest com atraso**, quinze minutos por padrão, só se o item ainda
estiver aberto.

**A idempotência é por (evento, regra, ação)**, então um evento pode
disparar várias ações. **Caminhos de link são dado:** o link de uma regra
aceita placeholders resolvidos contra os dados da notificação; um
placeholder sem resolução remove o link. O consumidor roda no core, onde
estão as credenciais dos canais.

## O que custou

Existe um template por adaptador, e a suíte de contrato é o que os mantém
completos. Os quinze minutos de atraso são um valor inicial a calibrar com
telemetria.

## Desde então

Os caminhos de link como dado foram acrescentados em 2026-09-01 para o
convite; o adaptador OneSignal se juntou ao SendGrid e ao SMTP em setembro.
O segundo fator usa os canais e deliberadamente não o notificador: um
código é um desafio, não uma notificação.
