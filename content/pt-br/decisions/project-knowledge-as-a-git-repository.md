---
title: "Em vez de compartilhar um volume, compartilhe um repositório"
translationKey: "decision-0021"
adr: "0021"
adr_title: "The project's knowledge is a git repository, hosted by the platform"
adr_file: "0021-project-knowledge-as-a-git-repository.md"
date: 2026-09-03
weight: 21
group: "knowledge"
description: "Duas tentativas eram do tipo errado — um config map, depois um pod carregador empurrando um tarball. Um volume compartilhado falhou em dois fatos. Então o idealizador nomeou a resposta: versionado em git."
related: ["0001", "0003", "0004", "0006", "0017"]
---

## O que estava na mesa

A expectativa do idealizador, dita com todas as letras em 2026-09-03: todo
documento que serve de conhecimento ao agente — memórias, contexto, specs,
planos — está disponível para *qualquer* demanda do projeto no momento em que
a sandbox sobe. Sem envios, sem requisições, sem config maps. Do ponto de
vista do agente o conteúdo simplesmente existe, compartilhado e colaborado
entre agentes.

Duas tentativas tinham precedido isto, e as duas eram do tipo errado. **Um
ConfigMap projetado como volume** — configuração, não dado; mora no etcd e
tem teto de um megabyte. Rejeitado na hora. **Um volume por demanda
preenchido por um pod carregador empurrando um tarball por WebSocket, montado
somente-leitura** — um transporte, por demanda, somente-leitura, reescrito a
cada retomada: o oposto de compartilhado.

O próximo passo óbvio, um volume por projeto, falhou em dois fatos: o cluster
local recusa volumes *read-write-many*, e um sistema de arquivos compartilhado
não faz ideia de *quem* mudou um arquivo — a única coisa que a decisão do log
de eventos exige que esta plataforma sempre saiba.

Então o idealizador nomeou: **versionado em git.** Em vez de compartilhar um
volume, compartilhe um repositório.

## Os caminhos que pesamos

**Um volume *read-write-many* por projeto.** O armazenamento local recusa;
*filers* gerenciados forneceriam; e ainda assim não tem atribuição. O git tem
atribuição embutida.

**Armazenamento de objetos montado como sistema de arquivos.** Zero cópias,
e o armazenamento que a plataforma já usa. Mas sem driver nos clusters
locais, semântica de último-que-escreve-ganha, e versionamento que dá
histórico sem autoria. Continua sendo a forma certa para binários grandes,
que um repositório faz mal.

**O provedor do próprio usuário como primário.** Exige integrar um provedor
*antes* de um projeto poder guardar conhecimento — contradizendo "nasce por
baixo do capô" — e põe a credencial do usuário no caminho da sandbox.
Sobrevive como espelho.

## O que escolhemos, e por quê

**O conhecimento mora no repositório raiz do projeto**, que nasce com o
projeto num servidor git que a plataforma roda. Regras, mapas de repositório,
memórias, e a spec, o plano e o contexto de cada demanda são arquivos nele, e
o manifesto no topo é gerado pela plataforma a cada push — um manifesto que
diverge da árvore é pior que nenhum.

**Toda sandbox o clona; os agentes escrevem commitando.** O clone é uma cópia
de trabalho, com escrita; compartilhar entre demandas é push e pull; um
conflito é o conflito do git, e um que o agente não consegue resolver vira um
item de atenção. Como um commit é atribuído é a regra da
[ADR-0003](../organization-credential-human-authorship/) e não é repetida.

**A sandbox se autentica com um token que abre exatamente um repositório** —
por demanda, de curta duração, restrito àquele projeto, entregue como arquivo
projetado e nunca como variável de ambiente, que todo processo filho herda. É
a única credencial que a sandbox tem, e é aceitável pelo que ela abre: a
própria bancada do agente, não a chave de um terceiro.

**O remoto do usuário é um espelho**, anexável a qualquer momento: o
repositório da plataforma continua primário e empurra adiante com a
credencial do usuário vinda do vault, que a sandbox nunca vê. Sem
sincronização bidirecional na v1 — isso é um motor de conflitos que ninguém
pediu.

**Todo push é um evento**, carregando o autor, os caminhos e o commit — o que
regenera o manifesto, alimenta a linha do tempo, e faz o ciclo das lições
existir: os achados de uma demanda viram a memória do projeto pelo agente
escrevê-los onde o próximo agente vai ler.

## O que custou

Um componente com estado para rodar — um servidor git com armazenamento e
backup — o preço de "nasce por baixo do capô", e a plataforma já roda
Postgres e NATS do mesmo jeito. `git` na imagem do devbox. As garantias do
contrato da sandbox reescritas: o caminho do conhecimento passou a ser
gravável, persistente entre retomadas e entre demandas, e cercado por
projeto. O mecanismo construído nos dois dias anteriores foi removido. E a
regra que caiu daí: texto no git, bytes no bucket.

## Desde então

Foi construído no mesmo dia e provado nos dois executores, Docker e
Kubernetes: o servidor git, tokens por demanda, o clone no provisionamento, o
*fan-out* de push e o espelho. A metade que falta é a superfície — a visão da
estante no cockpit.
