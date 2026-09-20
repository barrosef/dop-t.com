---
title: "Um banco só, e a janela que precisávamos fechar"
translationKey: "decision-0014"
adr: "0014"
adr_title: "PostgreSQL as the single database, and the outbox that moves its events"
adr_file: "0014-postgres-persistence.md"
date: 2026-08-30
weight: 14
group: "events"
description: "Três naturezas de dado, um motor. E o jeito ingênuo de publicar um evento — gravar, depois avisar o broker — tem uma brecha onde um processo pode morrer entre os dois. A outbox a fecha sem commit distribuído."
related: ["0004", "0001", "0018"]
---

## O que estava na mesa

Duas perguntas, que se revelaram um assunto só.

**Onde guardar.** Uma conversa antiga tinha sinalizado "um banco adequado,
como Mongo por exemplo". Quando o backend foi de fato desenhado, as cargas
ficaram claras, e são de três naturezas: relacional — contas, memberships,
grants, projetos; documental — fluxos, cards de agente, payloads de eventos; e
semântica — as memórias que os agentes buscam.

**Como tirar um evento de lá.** O produto pedia processos desacoplados,
resiliência forte e transações atômicas baseadas em eventos. O jeito ingênuo
— gravar no banco, depois publicar no broker — tem uma janela: o processo
morre entre os dois, o estado mudou, ninguém ouviu. Um commit distribuído
resolve e cobra caro em complexidade e disponibilidade.

## Os caminhos que pesamos

**MongoDB.** Bom para documentos, ruim para o trabalho relacional que domina
o domínio; a integridade multi-tenant viraria responsabilidade da aplicação —
o lugar errado.

**Postgres mais um armazenamento vetorial dedicado.** Rejeitado por YAGNI:
mais um serviço para operar antes de haver volume que o justifique.

**Postgres mais Kafka como log.** Rejeitado: o log mora no banco; o broker
transporta, não guarda a verdade. E o Kafka sozinho é um caminhão para a nossa
carga.

**Google Pub/Sub.** Gerenciado e bom, e nos amarra a uma nuvem — fica como
segundo adaptador do port do barramento para quem quiser um gerenciado.

**Redis Streams.** Sem as garantias do JetStream, e traz um serviço de que
não precisamos — não há requisito de cache.

**Publicar direto do código.** Essa é a janela.

## O que escolhemos, e por quê

**PostgreSQL para tudo.** Relacional na espinha — integridade multi-tenant é
uma chave estrangeira e uma constraint, não uma convenção, e toda tabela de
domínio carrega um `account_id`. JSONB para fluxos, cards e payloads de
eventos. pgvector para as memórias, sem armazenamento extra. O log de eventos
numa tabela só de acréscimo particionada por mês. As projeções começam como
views materializadas e viram tabelas alimentadas pelo worker só se o custo
exigir. E o mesmo motor nos dois mundos: Cloud SQL no Google, CloudNativePG
num cluster.

**Uma outbox transacional.** Toda mudança de estado grava, na mesma
transação, o estado novo e o evento. Commit significa atômico, por
construção. Um relay lê a outbox e publica no broker — entrega *at least
once*, sem commit em duas fases.

**NATS JetStream** como broker: um container, idêntico no k3s e num cluster
gerenciado, persistente, com grupos de consumidores e replay. Os consumidores
são idempotentes e constroem as projeções; uma mensagem envenenada vai para a
fila de dead letters.

**Processos longos são sagas** orquestradas pelo core — a finalização da
demanda é a canônica: cada passo um evento, uma falha compensa ou escala, o
estado no Postgres. A pessoa percebe como síncrono porque a borda empurra o
progresso por SSE; a execução sobrevive a reinícios.

## O que custou

A carga de eventos concentrada no mesmo banco, então particionamento,
retenção e vigilância são nossos. Busca vetorial no Postgres tem teto, e
quando chegar é extraída atrás do mesmo port. O relay faz polling, então há
latência entre o commit e a publicação. Idempotência vira obrigação de todo
consumidor. E o NATS é mais uma coisa para rodar.

## Desde então

A frase "uma mensagem envenenada vai para a DLQ" foi escrita como se a fila
existisse. Em 2026-09-13 descobrimos que não: no esgotamento o adaptador
chamava `Term()`, que descarta, sob uma linha de log dizendo que a mensagem
tinha sido salva. A retenção de trinta dias do stream era a única coisa entre
um evento esgotado e nada. A fila, a classificação de falhas e o ledger de
erros foram desenhados e construídos naquela semana, e o registro agora os
declara como construídos, com uma linha de revisão datada. Escrever o que você *acredita* que existe é
como se descobre que não existe.
