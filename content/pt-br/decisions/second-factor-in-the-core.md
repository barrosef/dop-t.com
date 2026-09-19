---
title: "Onde o segundo fator mora é a decisão"
translationKey: "decision-0020"
adr: "0020"
adr_title: "The second factor is the platform's, with three verifiers"
adr_file: "0020-second-factor-in-the-core.md"
date: 2026-09-02
weight: 20
group: "identity"
description: "O provedor de identidade oferecia MFA de graça. Construímos o nosso mesmo assim — porque o provedor não cobria o requisito, não sobreviveria a uma troca, e nunca poderia ser exercitado localmente."
related: ["0001", "0016", "0018", "0019"]
---

## O que estava na mesa

O requisito do produto fixou os três segundos fatores que uma pessoa pode
escolher: um aplicativo autenticador, e-mail e SMS. A pergunta não era *se* — a
plataforma nasce com segundo fator — mas *onde ele mora*, e havia dois
candidatos.

**O do provedor de identidade.** O Firebase, nosso primeiro adaptador de
identidade, tem MFA. Três coisas o tornavam uma casa ruim. Não cobre o
requisito: o segundo fator dele é SMS e TOTP, e e-mail não existe lá — então
metade da funcionalidade seria nossa de qualquer jeito, e dois mecanismos
decidiriam a mesma coisa. Não sobrevive a uma troca de adaptador: a semântica
de MFA é diferente em cada provedor — inscrição, recuperação, o que o token
afirma e como — então a segurança da conta dependeria de qual fornecedor está
ligado. E o ambiente local não consegue exercitá-lo: o emulador não faz
inscrição TOTP, o que significa entregar um caminho de segurança que nunca
roda localmente — a divergência que já tinha custado a esta plataforma duas
falhas de autenticação.

**O da plataforma.** A semente TOTP é uma credencial, então pertence ao vault,
que mora no core. O canal de e-mail já existia. O log de eventos já existia. O
que faltava era um domínio e um canal.

## Os caminhos que pesamos

**Delegar o MFA ao Firebase Identity Platform.** A resposta padrão, e a certa
num produto com um único provedor de identidade e sem fator por e-mail.
Nenhuma das duas coisas é verdade aqui.

**Aceitar um fator afirmado pelo provedor** — um token cujo `amr` diz `mfa` —
como equivalente ao nosso. Conveniente: quem tem 2FA na conta Google não
faria o nosso. Rejeitado para a v1 porque cria duas réguas para uma decisão,
o que esta plataforma insiste em recusar; registrado como item aberto para o
dia em que o port conseguir normalizar a afirmação.

**Só TOTP.** O mais forte e o mais barato. Rejeitado porque exclui quem não
usa aplicativo autenticador — exatamente da proteção.

**Um *magic link* em vez de um código por e-mail.** Um link é um *bearer*
viajando numa caixa de entrada, exatamente o que [a decisão do
convite](../invite-without-token/) tinha acabado de remover. Um código precisa
ser digitado na sessão que o pediu.

## O que escolhemos, e por quê

O segundo fator é um conceito de domínio do core, com um mecanismo e três
verificadores; o provedor de identidade faz o primeiro fator e nada mais. Um
fator nasce `pending` e só vira `active` quando a pessoa devolve um código —
uma fechadura que ninguém testou é descoberta no dia do login que falha. Um
fator por e-mail exige e-mail verificado. Os códigos são de uso único e morrem
na primeira resposta correta.

O código é um **desafio, não uma notificação**. O teste da própria tabela de
notificações — "isso merece interromper a pessoa fora da plataforma?" — falha
nas duas direções: ninguém está sendo interrompido, e um código de 2FA não
pode passar por uma política com digests, atrasos e destinatários resolvidos
por membership. Ele usa o canal e não o gatilho, que é o vocabulário da
[ADR-0018](../communication-trigger-and-channel/) aplicado ao contrário.

O SMS precisou de um port novo, `SMSer`, nascido com dois adaptadores e uma
suíte de contrato como todos os outros — estreito de propósito: sem assunto,
sem HTML, 160 caracteres, um destino em E.164.

O que exige um *step-up* novo: entrar, gravar uma credencial no vault, mudar
um papel, convidar, revogar, apagar uma conta. Leitura não é bloqueada — um
desafio a cada requisição seria teatro e ensinaria as pessoas a responder sem
ler. Revogar um fator derruba todo *step-up* daquela pessoa em toda sessão.
Dez códigos de recuperação, mostrados uma vez, guardados com hash: sem eles um
celular perdido vira chamado de suporte, e o suporte vira o desvio. Cinco
falhas esfriam um fator; e como um SMS custa dinheiro, o *envio* tem os
próprios dois tetos — sessenta segundos entre mensagens e cinco por hora —
porque um loop que nunca responde não custaria nada a quem o roda.

## O que custou

Quem tem 2FA no provedor de identidade faz duas vezes até o item aberto ser
decidido. O SMS é o mais fraco dos três — o NIST o desencoraja — e o único
que custa por tentativa, o que também faz dele a superfície de abuso.
Localmente não há entrega de SMS: o adaptador imprime o código. E o core
confiar no identificador de sessão da borda deixou de ser uma preocupação de
fundo, que é o fio que a [ADR-0022](../the-core-verifies-its-callers/) puxou
no dia seguinte.

## Desde então

Foi entregue de ponta a ponta no dia em que foi decidido — o verificador TOTP
escrito sobre a biblioteca padrão, os dois canais, o gate de *step-up* em
quatro operações, o contrato, a borda e as telas do cockpit.
