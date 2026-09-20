---
title: "Arquitetura"
translationKey: "architecture"
seo_title: "Arquitetura do DOP — core em Go, BFF em Python, cockpit em React, eventos e uma sandbox por demanda"
layout: "architecture"
description: "Como a plataforma DOP é construída: os componentes, as cinco invariantes, a vida de um evento e o registro de decisões."
components_title: "Os componentes"
components:
  - name: "dop-core"
    stack: "Go"
    repo: "dop-core"
    role: "O core: domínio, estado, transações e o log de eventos."
    text: "Um binário, quatro modos (serve, worker, sched, launcher). É dono do schema e do vault; é a única fonte da verdade, e verifica a assinatura de todo chamador."
  - name: "dop-api"
    stack: "Python"
    repo: "dop-api"
    role: "O BFF: REST e SSE para o cockpit, gRPC para a CLI."
    text: "Não tem banco nem segredo. Traduz e encaminha; toda escrita passa pelo core."
  - name: "dop-app"
    stack: "React · Vite"
    repo: "dop-app"
    role: "O cockpit onde o desenvolvedor trabalha."
    text: "A caixa de atenção, a árvore de workspaces e projetos, o cockpit da demanda. Consome um contrato commitado e gera dele os seus hooks e schemas."
  - name: "dop-infra"
    stack: "Terraform · k3d"
    repo: "dop-infra"
    role: "A infraestrutura da plataforma."
    text: "Terraform para o Google Cloud (Cloud Run, Secret Manager, Identity Platform) e um ambiente local em k3d com Postgres, NATS JetStream e os emuladores do Firebase."
  - name: "sandbox"
    stack: "microVM"
    repo: ""
    role: "Uma por demanda, com um único worktree compartilhado."
    text: "A fronteira dura é entre contas e entre demandas. O conhecimento do projeto é clonado lá dentro como um repositório git; a verificação roda da fonte num runner efêmero."
diagrams:
  title: "Os diagramas"
  lead: "Seis figuras, da plataforma inteira até cada componente. Todo nome nelas é um pacote, um port ou um adaptador que existe nos repositórios; o que está planejado e não construído aparece tracejado."
  items:
    - id: architecture
      title: "A plataforma"
      caption: "O cockpit e a CLI chegam ao BFF; o BFF chega ao core; o core é dono do estado, dos eventos e das sandboxes. O Identity Platform autentica a pessoa na borda."
    - id: core
      title: "O core por dentro — o hexágono"
      caption: "Um binário, quatro modos, um domínio. O domínio nunca importa um driver: declara um port, e o adaptador à direita o implementa. Todo port tem pelo menos dois adaptadores e uma suíte de contrato que ambos precisam passar — é isso que faz do ambiente local e do Google Cloud a mesma plataforma."
    - id: bff
      title: "O BFF por dentro — dois transportes, uma regra"
      caption: "REST com SSE para o cockpit, gRPC para a CLI, e o mesmo caso de uso por baixo. A autorização está presa ao caso de uso, não ao router, então uma regra não pode existir num transporte e não no outro; um teste de paridade mantém isso."
    - id: events
      title: "Eventos e o motor de fluxos"
      caption: "O pipeline: uma escrita commita o seu evento na mesma transação; um relay publica; todo consumidor recebe. Os dois blocos marcados com um mais são subprocessos desenhados nas duas figuras seguintes — o motor de fluxos, que decide o que um evento dispara, e o caminho da falha, que cobre a entrega de todo consumidor, não de um só."
    - id: flow-engine
      title: "O motor de fluxos"
      caption: "Um fluxo é dado: uma lista versionada de estágios tipados, cada um com um portão, os seus artefatos e as suas ações na entrada e na saída. O fluxo efetivo desce por uma cadeia — plataforma, conta, workspace, projeto, demanda — em que o nível declarado mais próximo vence, e uma demanda congela a versão com que começou. Quando um estágio avança, as ações do estágio e a tabela de regras acumulada são decididas em ações planejadas; o executor as roda por um registro de quatro e registra cada uma como aplicada por evento, regra e ação, para uma retentativa nunca repetir uma."
    - id: resilience
      title: "O caminho da falha"
      caption: "O mesmo caminho cobre todo consumidor. Uma entrega que falha é reentregue com backoff até cinco vezes; uma esgotada vira uma dead letter com o envelope inteiro e as suas tentativas, publicada num subject próprio. O consumidor da DLQ reexecuta o mesmo handler mais três vezes, depois encerra a mensagem — mas toda tentativa, a sua classificação e o último sucesso ficam no ledger de erros, e uma assinatura que esgota duas vezes é aprendida como irrecuperável até um sucesso rebaixá-la."
    - id: app
      title: "O cockpit por dentro"
      caption: "Um roteador, dez páginas, seis famílias de componentes, dois hooks e um cliente gerado por baixo. O contrato é um arquivo OpenAPI commitado; o Orval produz dele os hooks do react-query e os schemas Zod; uma camada de fetch leva o token da pessoa e a conta em toda chamada. O Firebase Auth emite o token; o BFF responde REST e empurra o stream de atenção por SSE. Nada que o backend já decidiu é decidido de novo aqui."
    - id: callauth
      title: "Como o core verifica seus chamadores"
      caption: "Toda chamada ao core carrega uma assinatura que o core consegue checar: o token da pessoa, encaminhado inteiro pelo BFF, ou uma asserção da plataforma assinada com a chave do próprio chamador quando não há pessoa. O interceptor resolve o ator pelo token e a conta pela asserção, recusa quando os dois discordam, e deixa a autorização dizer não com uma mensagem que significa algo. O IAM do Cloud Run na nuvem e uma network policy num cluster restringem quem consegue sequer alcançar o serviço; nenhum dos dois substitui a assinatura."
    - id: external
      title: "A plataforma e com quem ela conversa"
      caption: "Todo serviço externo fica atrás de um port, então trocar um é um adaptador e um valor de configuração, não uma reescrita. O que está sólido tem adaptador hoje; o que está tracejado está no roadmap."
invariants_title: "As cinco invariantes"
invariants_lead: "As paredes estruturais da arquitetura. Uma mudança que quebra uma delas está errada mesmo quando compila e os testes passam."
invariants:
  - title: "O core é a única fonte da verdade."
    text: "É dono do schema e do vault. Toda escrita passa por ele."
  - title: "O BFF não tem banco nem segredo."
    text: "Traduz e encaminha; nunca persiste e nunca guarda uma credencial."
  - title: "Os arquivos .proto são o contrato."
    text: "Código gerado é regenerado, nunca editado à mão. Uma mudança incompatível é recusada de propósito."
  - title: "O cockpit consome um contrato commitado."
    text: "A spec OpenAPI é baixada e commitada; os hooks do react-query e os schemas Zod são gerados dela."
  - title: "O core verifica seus chamadores, e a verificação roda da fonte."
    text: "Uma assinatura em toda chamada, nunca um header de confiança. Uma verificação constrói do repositório num runner, não de uma imagem."
event:
  title: "A vida de um evento"
  lead: "Toda escrita no core emite um evento na mesma transação. O que acontece depois é infraestrutura, não problema do repositório."
  steps:
    - name: "escrita"
      text: "Uma transação no Postgres muda o estado e anexa o evento à outbox — um commit, ou nenhum."
    - name: "outbox"
      text: "Um relay lê a outbox e publica no NATS JetStream. O id da mensagem é o id do evento, então uma retentativa não duplica."
    - name: "envelope"
      text: "O evento viaja com o seu contexto: chave do agregado, ator, id da requisição, id da sessão, chamador."
    - name: "consumidor"
      text: "Um consumidor o processa. Em falha, é reentregue com backoff, até cinco vezes."
    - name: "dead letter"
      text: "Esgotado, o evento vai para uma única fila de dead letters com o histórico de tentativas e a sua classificação: recuperável, irrecuperável, desconhecida."
      dlq: true
    - name: "ledger de erros"
      text: "Toda falha é registrada por (evento, consumidor); uma assinatura que queima todas as retentativas duas vezes é aprendida como irrecuperável, e um sucesso a rebaixa."
adrs_title: "O registro de decisões"
adrs_lead: "Toda decisão estruturante, no formato MADR: contexto, decisão, alternativas consideradas, consequências. Um assunto, uma ADR."
---

O DOP são três repositórios que entregam e um que os fixa. Os componentes são
repositórios git independentes, agregados como submódulos pelo
[repositório guarda-chuva](https://github.com/barrosef/dop), que carrega a
documentação do produto: o PRD, as ADRs, as specs e as coleções da API.
