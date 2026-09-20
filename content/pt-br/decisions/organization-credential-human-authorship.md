---
title: "A organização age; a pessoa é a autora; o agente é o committer"
translationKey: "decision-0003"
adr: "0003"
adr_title: "An organization credential to act, human authorship on the commit"
adr_file: "0003-organization-credential-human-authorship.md"
date: 2026-08-29
weight: 3
group: "resources"
description: "Pushes e pull requests usam a credencial da organização, então nada quebra quando uma pessoa sai. Todo commit nomeia o desenvolvedor como author e a thread do agente como committer, então o histórico continua dizendo quem pediu e quem escreveu."
related: ["0002", "0004", "0021"]
---

## O que estava na mesa

Duas perguntas que puxam em direções opostas. O token OAuth de uma pessoa
pertence à pessoa e morre com o acesso dela — numa organização, todo projeto
que dependesse dele pararia. Uma credencial da organização sobrevive a
saídas mas, usada sozinha, apaga do histórico do repositório quem pediu a
mudança.

## Os caminhos que pesamos

**Tudo em nome da organização.** Rejeitado: nenhum rastro humano no
repositório; a rastreabilidade presa dentro da plataforma.

**A credencial da pessoa quando disponível, a da organização caso
contrário.** Rejeitada: mantém a dependência do token de uma pessoa e
contradiz a regra de que a integração de uma conta pessoal nunca é usada
numa organização.

## O que escolhemos, e por quê

Duas escolhas independentes, uma por pergunta — e um terceiro campo que o
git já tinha.

**Para agir**, uma organização usa uma credencial da organização: uma
instalação de GitHub App, um *group access token* do GitLab, um *service
principal* do Azure DevOps. Um token pessoal é permitido como saída de
emergência, e a interface mostra de quem a integração depende.

**Para atribuir**, o `author` de todo commit é o desenvolvedor que executou
a demanda, e o corpo do pull request nomeia quem pediu.

**E o `committer` é a thread** — o agente que fez o commit, como identidade
da plataforma, nunca uma pessoa. Os dois campos do git respondem a duas
perguntas: para quem o trabalho foi feito, e quem o escreveu. Um commit que
a plataforma faz sozinha leva a plataforma como committer e a pessoa que
agiu como author.

Estas regras governam todo repositório em que a plataforma escreve — os
repositórios de código do cliente e o repositório de conhecimento do
projeto igualmente — e estão enunciadas num lugar só.

## O que custou

Um ponto único de falha por organização: um App ou token revogado para todo
projeto daquela conta, então o status da integração é monitorado e o dono é
alertado. E o ator do push e o autor do commit são identidades diferentes, o
que pode surpreender quem lê a interface do provedor sem o contexto.

## Desde então

Quando o conhecimento do projeto virou um repositório git hospedado pela
plataforma, os commits dele seguiram estas regras sem mudança.
