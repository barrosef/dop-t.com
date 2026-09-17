---
title: "DOP"
translationKey: "home"
eyebrow: "orquestração de entrega · construído às claras"
headline: "O desenvolvedor e o agente constroem juntos. Do workspace ao PR entregue."
lead: "Um cockpit onde cada demanda vira uma sequência de eventos: o agente opera por baixo, o desenvolvedor acompanha e decide por cima. Nada acontece fora do registro."
why_title: "Quatro coisas verdadeiras sobre o DOP — e sobre pouquíssimas outras ferramentas."
values:
  - key: operadores
    title: "Dois operadores, um cockpit"
    text: "O agente executa a demanda; o desenvolvedor aprova a spec, dá contexto e decide. A divisão de responsabilidades está no fluxo, não num prompt."
  - key: eventos
    title: "Tudo é evento"
    text: "Cada escrita emite um evento que carrega o seu contexto — ator, requisição, sessão. Uma falha vai para uma fila de dead letters com tudo o que é preciso para reprocessar, não para um log que ninguém lê."
  - key: sandbox
    title: "Uma sandbox por demanda"
    text: "Isolamento entre contas e entre demandas, com o conhecimento do projeto montado lá dentro como um repositório git."
  - key: aberto
    title: "Construído às claras"
    text: "Cada decisão estruturante é uma ADR num repositório público, e o código que a implementa está ao lado."
how:
  title: "Um core que é dono da verdade. Todo o resto traduz."
  points:
    - title: "Um core em Go é a única fonte da verdade."
      text: "Ele é dono do schema, do vault e do log de eventos. Toda escrita passa por ele, e ele verifica quem está chamando."
    - title: "Um BFF em Python não tem banco nem segredo."
      text: "Traduz REST e SSE do cockpit em gRPC para o core, e encaminha. Nada persiste na borda."
    - title: "O cockpit consome um contrato commitado."
      text: "Os arquivos .proto são o contrato; os hooks e os schemas do cockpit são gerados de uma spec OpenAPI baixada, nunca escritos à mão."
stands_title: "Pré-lançamento, e honesto sobre isso."
closing: "A próxima demanda que você abrir pode ser a que o agente termina."
---
