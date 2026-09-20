---
title: "Um segundo fator, três verificadores, no core"
translationKey: "decision-0020"
adr: "0020"
adr_title: "The second factor is the platform's, with three verifiers"
adr_file: "0020-second-factor-in-the-core.md"
date: 2026-09-02
weight: 20
group: "identity"
description: "O provedor de identidade faz só o primeiro fator. O segundo — aplicativo autenticador, e-mail ou SMS — é um domínio do core, então é igual para todo provedor de identidade, roda localmente e guarda os segredos no vault."
related: ["0001", "0016", "0018", "0019"]
---

## O que estava na mesa

O produto oferece três segundos fatores: um aplicativo autenticador (TOTP),
e-mail e SMS. O segundo fator tinha que se comportar igual seja qual for o
provedor de identidade ligado, ser exercitável no ambiente local, e guardar
os segredos onde a plataforma guarda segredos.

## Os caminhos que pesamos

**O MFA do provedor de identidade.** Rejeitado: não oferece fator por
e-mail, a semântica muda por provedor — inscrição, recuperação, o que o token
afirma — e o emulador local não consegue exercitá-lo.

**Aceitar um fator que o provedor afirma** (um token cujo `amr` diz `mfa`)
como equivalente ao nosso. Adiado: duas réguas para uma decisão; aberto para
o dia em que o port de identidade conseguir normalizar a afirmação.

**Só TOTP.** Rejeitado: exclui quem não tem aplicativo autenticador.

***Magic links*** em vez de código por e-mail. Rejeitado: um *bearer*
viajando numa caixa de entrada. Um código é digitado na sessão que o pediu.

## O que escolhemos, e por quê

**O segundo fator é um domínio do core.** Um fator nasce `pending` e só vira
`active` quando a pessoa devolve um código válido — a inscrição prova a
posse. A semente TOTP é uma referência ao vault e nunca é devolvida depois
da inscrição; um fator por e-mail exige e-mail verificado.

**Três verificadores:** TOTP pela RFC 6238 (passo de 30 segundos, seis
dígitos, janela de ±1); um código de seis dígitos válido por dez minutos por
e-mail; o mesmo código por SMS. Um código morre na primeira resposta
correta.

**Um código é um desafio, não uma notificação.** Usa os ports de canal
diretamente e nunca o notificador — sem digest, sem atraso, sem
destinatário resolvido por membership. O SMS precisou de um port próprio,
`SMSer`, estreito de propósito (160 caracteres, destino em E.164), com dois
adaptadores e uma suíte de contrato como todos os outros.

**O *step-up* é registrado por usuário e sessão**, com validade. É exigido
para entrar quando há fator ativo, para gravar uma credencial no vault, para
mudar um papel, convidar ou revogar, e para apagar uma conta. Leituras não
são bloqueadas. Revogar um fator derruba todo *step-up* daquela pessoa em
toda sessão. Dez códigos de recuperação, mostrados uma vez, guardados com
hash.

**Limites:** cinco falhas consecutivas esfriam um fator; como um SMS custa
dinheiro, o envio tem tetos próprios — uma mensagem por minuto e cinco por
hora, por fator. Uma organização pode exigir segundo fator dos membros; a
exigência bloqueia operar naquela conta, não a conta pessoal da pessoa.

## O que custou

Quem tem MFA no provedor de identidade verifica duas vezes até o item adiado
ser decidido. O SMS é o mais fraco dos três e o único que custa por
tentativa, o que faz dele a superfície de abuso; a política da conta pode
desativá-lo. Localmente não há entrega de SMS — o adaptador imprime o
código.

## Desde então

Entregue de ponta a ponta no dia em que foi decidido. O identificador de
sessão em que o *step-up* se apoia é verificado pelo core desde [a decisão
dos chamadores](../the-core-verifies-its-callers/), no dia seguinte.
