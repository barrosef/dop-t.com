---
title: "Estágio"
translationKey: "status"
seo_title: "Estágio do DOP — o que foi entregue, o que está desenhado, o que vem a seguir"
layout: "status"
description: "Em que ponto está a plataforma DOP: os subprojetos e o seu estado, o que foi entregue e quando, o que vem a seguir."
subprojects_title: "Os subprojetos"
subprojects_lead: "A plataforma foi decomposta em sete subprojetos, decididos nesta ordem porque cada um opera dentro do anterior. Cinco estão desenhados; dois estão abertos."
shipped_title: "Entregue"
shipped:
  - date: "2026-09-13"
    title: "Eventos com contexto, e a fila de dead letters que faltava"
    text: "Todo evento viaja com a chave do agregado, o ator, a requisição e a sessão. Um evento esgotado vai para uma única fila de dead letters com o histórico de tentativas; as falhas são classificadas e aprendidas."
    ref: "ADR-0014 · dop-core"
  - date: "2026-09-09"
    title: "O ambiente de QA no Google Cloud, sob os domínios do produto"
    text: "Codificado em Terraform, com o core e o BFF no Cloud Run, Postgres e NATS numa VM, e api.qa.dop-t.com e auth.qa.dop-t.com verificados. A cadeia de contas roda de ponta a ponta: login, BFF, core, banco."
    ref: "dop-infra"
  - date: "2026-09-03"
    title: "O conhecimento do projeto é um repositório git, hospedado pela plataforma"
    text: "Um servidor git com tokens por demanda; a sandbox clona o projeto em /project com escrita; todo artefato é commitado e todo push é um evento."
    ref: "ADR-0021"
  - date: "2026-09-03"
    title: "O core verifica seus chamadores"
    text: "Uma assinatura em toda chamada — o token da pessoa onde há pessoa, uma asserção da plataforma onde não há — com o formato de fio fixado pelo mesmo vetor de teste em Go e em Python."
    ref: "ADR-0022"
  - date: "2026-09-02"
    title: "O segundo fator, de ponta a ponta"
    text: "TOTP escrito na biblioteca padrão, verificadores por e-mail e SMS atrás de ports, um gate de step-up em quatro operações sensíveis, e as telas do cockpit."
    ref: "ADR-0020"
next_title: "A seguir"
next:
  - title: "As ferramentas do agente e o cockpit"
    text: "A peça que falta para a plataforma executar em vez de só modelar. Foco de provedor: Anthropic; o port multiprovedor fica."
  - title: "Claude Code hospedado como laboratório"
    text: "A sandbox roda o binário sem modificação e o desenvolvedor entra com a própria assinatura — o lugar onde medir o custo real de uma demanda não custa nada."
  - title: "As histórias de usuário"
    text: "Só depois de a estrutura estar pronta. A reação a um evento como dado (P-29) vem antes, porque escrever as histórias contra um switch seria reescrevê-las depois."
---

A plataforma é construída às claras. Esta página é um resumo do
[roadmap](https://github.com/barrosef/dop/blob/main/docs/ROADMAP.md) no
repositório guarda-chuva, que é a fonte de todo estado e toda data abaixo.
