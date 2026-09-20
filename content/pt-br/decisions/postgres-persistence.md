---
title: "Um banco só, uma outbox, e uma fila para o que falha"
translationKey: "decision-0014"
adr: "0014"
adr_title: "PostgreSQL as the single database, and the outbox that moves its events"
adr_file: "0014-postgres-persistence.md"
date: 2026-08-30
weight: 14
group: "events"
description: "O PostgreSQL guarda a espinha relacional, os documentos e os vetores. Toda mudança de estado grava o seu evento na mesma transação; um relay publica no NATS; uma entrega que continua falhando pousa numa única fila de dead letters com tudo o que é preciso para tentar de novo."
related: ["0004", "0001", "0018"]
---

## O que estava na mesa

Três naturezas de dado — relacional (contas, memberships, grants,
projetos), documental (fluxos, cards, payloads de eventos) e semântica (as
memórias que os agentes buscam) — e dois requisitos: uma mudança de estado
e o seu evento são atômicos, e os processos que reagem a eventos são
desacoplados do que os gravou.

## Os caminhos que pesamos

**MongoDB.** Rejeitado: a integridade relacional iria para a aplicação.

**Postgres mais um armazenamento vetorial dedicado.** Rejeitado até o
volume exigir.

**Kafka como log.** Rejeitado: o log mora no banco; um broker transporta.

**Google Pub/Sub.** Mantido como segundo adaptador do port do barramento,
não o padrão.

**Redis Streams.** Rejeitado: garantias mais fracas, um serviço a mais.

**Publicar sem outbox.** Rejeitado: um evento se perde sempre que o processo
morre entre a gravação e a publicação.

## O que escolhemos, e por quê

**PostgreSQL para tudo**, o mesmo motor nos dois ambientes. A integridade
relacional está no schema — toda tabela de domínio carrega a conta, e o
isolamento é uma constraint, não uma convenção. JSONB para fluxos, cards e
payloads. pgvector para as memórias. O log de eventos numa tabela só de
acréscimo particionada por mês. As projeções começam como views
materializadas.

**Uma outbox transacional.** Toda mudança de estado grava o estado novo e o
evento numa transação; um relay lê a outbox e publica no broker — entrega
*at least once*, sem commit distribuído.

**O NATS JetStream** carrega os eventos: um container, idêntico localmente e
na nuvem, persistente, com replay. Os consumidores são idempotentes e
constroem as projeções. Uma entrega é retentada até cinco vezes com backoff
— um segundo, cinco, quinze, um minuto. **Uma entrega esgotada vai para uma
única fila de dead letters** carregando o envelope completo do evento, o
consumidor e o histórico de tentativas; um consumidor dedicado a retenta
mais três vezes, toda tentativa é registrada num ledger de erros, e cada
falha é classificada como recuperável, irrecuperável ou desconhecida — uma
classificação aprendida da assinatura do erro e rebaixada por um sucesso
posterior.

**Processos longos são sagas** orquestradas pelo core: cada passo um evento,
falhas compensando ou escalando para a caixa de atenção, estado no Postgres,
progresso chegando ao cockpit por SSE.

## O que custou

Um banco que também carrega a carga de eventos, então particionamento,
retenção e vigilância são nossos. Busca vetorial tem teto no Postgres. O
relay faz polling, acrescentando latência entre commit e publicação. Todo
consumidor precisa ser idempotente. E o NATS é mais uma coisa para rodar.

## Desde então

O caminho de dead letters — a fila, a classificação e o ledger de erros —
foi especificado em detalhe e construído em 2026-09-13, junto com os campos
de contexto do envelope do evento.
