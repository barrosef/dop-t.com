---
title: "Privacidade"
translationKey: "privacy"
layout: "privacy"
seo_title: "Política de privacidade — dop-t.com e a plataforma DOP"
description: "O que este site e a plataforma DOP coletam, por quê, onde fica guardado e como pedir que seja apagado."
---

Esta política cobre duas coisas: **este site** (dop-t.com) e **a plataforma
DOP** — o produto em que você entra. Foi escrita pela pessoa que mantém os
dois, em linguagem simples, e diz apenas o que é verdade hoje. Quando algo
mudar, a data no topo muda junto.

## Quem responde

O DOP é idealizado e mantido por **[Ed Barros](https://barrosef.com)**, pessoa
física, no Brasil. Para qualquer assunto desta política, escreva para
**[{{< param "legal.contactEmail" >}}](mailto:{{< param "legal.contactEmail" >}})**.

## Este site

O site é estático, servido pelo GitHub Pages. Não tem contas, formulários nem
comentários.

**Analytics.** O site usa o Google Analytics 4 para contar visitas e ver quais
páginas são lidas. Ele grava cookies chamados `_ga` e `_ga_*` no seu navegador,
registra a página, a região aproximada, o navegador e o tipo de dispositivo, e
trunca o seu endereço IP antes de guardar qualquer coisa. Se o seu navegador
envia o sinal *Do Not Track*, o script nem é carregado. Você também pode
bloqueá-lo com qualquer bloqueador de conteúdo; o site funciona igual sem ele.
Os termos do próprio Google para os dados que ele processa em nosso nome estão
em [policies.google.com/privacy](https://policies.google.com/privacy).

**Fontes.** As tipografias são servidas deste domínio, não do Google Fonts.
Renderizar uma página não faz nenhuma requisição a terceiros.

**Logs do servidor.** O GitHub Pages mantém os logs de acesso comuns de um
servidor web (endereço IP, URL pedida, horário, user agent) sob a política do
próprio GitHub. Nós não os recebemos.

**Nada mais.** Sem publicidade, sem pixels, sem fingerprinting, sem corretores
de dados.

## A plataforma DOP

A plataforma está em pré-lançamento. Entrar só é possível para pessoas que
convidamos; esta seção descreve o que acontece quando você entra.

**O que coletamos no login.** Você entra com um endereço de e-mail e uma senha,
ou pelo Google ou pelo GitHub. Do provedor recebemos o seu **nome, endereço de
e-mail e foto de perfil** — nada mais; não pedimos acesso ao seu calendário,
aos seus contatos ou aos seus repositórios pelo login. A autenticação é feita
pelo Google Cloud Identity Platform em nosso nome.

**O que coletamos enquanto você usa.** A plataforma registra o que você faz
dentro dela como um log de eventos — uma conta criada, uma demanda aberta, uma
especificação aprovada, um pull request entregue — com quem fez e quando. Esse
log **é** o produto: é como você e o agente enxergam a mesma história. Ele fica
dentro da sua conta.

**Credenciais que você dá à plataforma** (um token do GitHub ou do GitLab, por
exemplo) ficam guardadas cifradas no Google Cloud Secret Manager, são usadas
só para fazer o que você pediu, e nunca são mostradas de volta por inteiro.

**Onde fica.** No Google Cloud, nos Estados Unidos (`us-central1`), num projeto
nosso. Não vendemos dados, não os compartilhamos com ninguém além dos
provedores nomeados aqui, e não os usamos para treinar modelos.

**Dados de usuário do Google.** Os dados recebidos pelo *Entrar com o Google*
são usados apenas para criar e identificar a sua conta, e o uso deles pela
plataforma segue a [Política de Dados de Usuário dos Serviços de API do
Google](https://developers.google.com/terms/api-services-user-data-policy),
incluindo os requisitos de Uso Limitado.

**Apagar.** Escreva para o endereço acima a partir do e-mail que a sua conta
usa, e a conta e tudo o que está sob ela são apagados. Enquanto a plataforma
está em pré-lançamento isso é feito à mão; vai ser um botão.

## Os seus direitos

Pela Lei Geral de Proteção de Dados (LGPD) e, onde se aplica, pelo GDPR, você
pode perguntar o que temos sobre você, pedir correção ou exclusão, e retirar
um consentimento que deu. O endereço acima é o canal; a resposta vem de uma
pessoa.

## Crianças

Nem o site nem a plataforma se destinam a menores de 18 anos.
