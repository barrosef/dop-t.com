---
title: "Quem age, e quem assina o commit"
translationKey: "decision-0003"
adr: "0003"
adr_title: "An organization credential to act, human authorship on the commit"
adr_file: "0003-organization-credential-human-authorship.md"
date: 2026-08-29
weight: 3
group: "resources"
description: "O token de uma pessoa morre quando a pessoa sai. A credencial de uma organização apaga quem pediu a mudança. Dois problemas, uma escolha para cada — e um terceiro campo que o git sempre teve."
related: ["0002", "0004", "0021"]
---

## O que estava na mesa

O token OAuth de um usuário pertence à pessoa. Quando ela sai da empresa,
revoga o acesso ou troca a senha, a integração morre e leva junto todo projeto
que dependia dela. Numa conta pessoal, tudo bem — a pessoa *é* a conta. Numa
organização, é uma bomba-relógio.

A alternativa óbvia cria o problema oposto: se a plataforma age com a
credencial de uma organização, o repositório mostra a instalação do App, e o
histórico para de dizer quem pediu a mudança. Rastreabilidade é exatamente o
que o dossiê de uma demanda promete.

## Os caminhos que pesamos

**Tudo em nome da organização.** Mais simples e uniforme. Rejeitado: o
repositório para de registrar quem executou o trabalho, e a rastreabilidade
fica presa dentro da plataforma — inútil para quem lê o histórico meses
depois.

**A credencial da pessoa quando ela tem uma**, caindo para a da organização.
Atribuição perfeita. Rejeitada porque contradiz a regra de que a integração
de uma conta pessoal nunca é usada dentro de uma organização, e reintroduz
exatamente a fragilidade que esta decisão existe para remover.

## O que escolhemos, e por quê

Duas escolhas independentes, uma por problema.

**Para agir**, uma organização usa uma credencial da organização — um GitHub
App instalado na organização, um *group access token* no GitLab, um *service
principal* no Azure DevOps. Sobrevive a saídas. Um token pessoal é permitido
como saída de emergência, mas a interface mostra de quem a integração depende,
para que o risco seja visível em vez de descoberto no dia em que quebra.

**Para atribuir**, o push e o pull request usam a credencial da organização,
mas todo commit carrega um `author` com o nome e o e-mail do desenvolvedor que
executou a demanda, e o corpo do PR nomeia quem pediu.

E então o terceiro campo do git. **O `committer` é a thread** — o agente que
fez o commit, como identidade da plataforma, nunca uma pessoa. O git tem dois
campos para duas perguntas: `author` responde *para quem* o trabalho foi
feito, e o histórico do cliente guarda a pessoa; `committer` responde *quem
escreveu*, e a atribuição da plataforma guarda o agente. Um commit que a
plataforma faz sozinha — um índice regenerado, uma regra salva do cockpit —
leva a plataforma como committer e a pessoa que agiu como author.

## O que custou

Um ponto único de falha: se o App é desinstalado ou o token revogado, todo
projeto daquela conta para. Isso exige monitorar o status da integração e
alertar o dono. E o ator do push e o autor do commit são entidades diferentes,
o que pode confundir quem lê a interface do provedor sem o contexto.

## Desde então

Quando o conhecimento do projeto virou um repositório git hospedado pela
plataforma ([ADR-0021](../project-knowledge-as-a-git-repository/)), a
pergunta "como um commit é atribuído lá?" já tinha resposta: estas regras
governam todo repositório em que a plataforma escreve, e aquele registro
explicitamente se recusa a repeti-las. Uma regra, um lugar.
