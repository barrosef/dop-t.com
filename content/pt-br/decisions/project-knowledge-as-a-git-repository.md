---
title: "O conhecimento do projeto é um repositório git que a plataforma hospeda"
translationKey: "decision-0021"
adr: "0021"
adr_title: "The project's knowledge is a git repository, hosted by the platform"
adr_file: "0021-project-knowledge-as-a-git-repository.md"
date: 2026-09-03
weight: 21
group: "knowledge"
description: "Todo projeto tem um repositório raiz, que nasce com ele e é clonado em toda sandbox. Os agentes escrevem commitando; todo push é um evento; um token por demanda abre exatamente aquele repositório; o remoto do próprio usuário é um espelho."
related: ["0001", "0003", "0004", "0006", "0017"]
---

## O que estava na mesa

Todo documento que é o conhecimento do agente — regras, mapas, memórias,
specs e planos de demanda — precisa estar presente em toda sandbox de um
projeto desde o provisionamento, gravável pelos agentes, compartilhado entre
demandas e atribuído por mudança. Compartilhar arquivos não basta: a
plataforma precisa sempre saber quem mudou o quê.

## Os caminhos que pesamos

**Um ConfigMap.** Rejeitado: configuração, não dado; um megabyte no etcd.

**Um volume por demanda preenchido por um carregador.** Rejeitado: por
demanda, somente-leitura, sem compartilhamento.

**Um volume *read-write-many* por projeto.** Rejeitado: indisponível no
cluster local, e um sistema de arquivos não tem atribuição.

**Armazenamento de objetos montado como sistema de arquivos.** Rejeitado
para texto — último-que-escreve-ganha, histórico sem autoria; mantido para
binários.

**O provedor do próprio usuário como primário.** Rejeitado: exige uma
integração antes de um projeto poder guardar conhecimento, e põe a
credencial do usuário no caminho da sandbox. Sobrevive como espelho.

## O que escolhemos, e por quê

**Todo projeto tem um repositório raiz**, criado com ele num servidor git
que a plataforma roda, com um layout fixo — `rules/`, `index/`, `memory/`,
`demand/<id>/` — e um manifesto que a plataforma regenera a cada push.

**Toda sandbox o clona** em `/project`, com escrita. Os agentes leem arquivos
e escrevem com `git commit` e `git push`; compartilhar entre demandas é push
e pull; um conflito que o agente não resolve vira um item de atenção. Os
commits são atribuídos pela [decisão das
credenciais](../organization-credential-human-authorship/).

**A credencial da sandbox é um token que abre exatamente um repositório** —
por demanda, de curta duração, restrito ao projeto, entregue como arquivo
projetado e nunca como variável de ambiente. É a única credencial que uma
sandbox tem.

**O remoto do usuário é um espelho de push:** o repositório da plataforma
continua primário e empurra adiante com a credencial do usuário vinda do
vault, que a sandbox nunca vê. Sem sincronização bidirecional. Uma pessoa
também pode clonar o repositório da plataforma diretamente com a sua
identidade na plataforma.

**Todo push é um evento** — autor, caminhos, commit — regenerando o
manifesto, alimentando a linha do tempo, e fechando o ciclo das lições
quando um agente commita em `memory/`. O port, `ProjectRepository`, tem
dois adaptadores — o servidor git da plataforma no Kubernetes e
repositórios *bare* atrás de `git http-backend` localmente — e uma suíte de
contrato. **Texto no git, bytes no armazenamento de objetos.**

## O que custou

Um servidor git com estado para rodar, com armazenamento e backup. `git` na
imagem do devbox e um clone no provisionamento. As garantias do contrato da
sandbox para o caminho do conhecimento: legível, gravável, visível para a
próxima sandbox depois de um push, persistente entre retomadas e demandas,
cercado por projeto.

## Desde então

Construído e provado nos dois executores — Docker e Kubernetes — no dia em
que foi decidido. A visão da estante no cockpit é a superfície que falta.
