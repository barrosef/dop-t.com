---
title: "A verificação roda num runner que constrói da fonte"
translationKey: "decision-0023"
adr: "0023"
adr_title: "Verification builds from source in a runner, not from an image"
adr_file: "0023-verification-runs-from-source.md"
date: 2026-09-04
weight: 23
group: "delivery"
description: "Um runner efêmero puxa o commit, constrói a aplicação da fonte e a sobe — nenhuma imagem do projeto é jamais construída, empurrada ou implantada. O cache da conta torna a segunda execução rápida; o runner ocupa o endereço da demanda enquanto roda."
related: ["0001", "0003", "0005", "0017"]
---

## O que estava na mesa

A evidência de uma verificação verde precisa nomear o commit em que rodou e
o ambiente em que rodou. Rodar um pod a partir de uma imagem exigiria, no
caminho crítico de toda execução, traduzir o compose do projeto em
manifestos, construir a imagem da aplicação, empurrá-la para um registro e
puxá-la de volta — um construtor e um registro que existem só para mover
bytes entre dois lugares do mesmo cluster.

## Os caminhos que pesamos

**Compose dentro da sandbox.** Rejeitado para verificação: o ambiente do
agente não é limpo. Continua candidato para a bancada do próprio
desenvolvedor.

**Um pod efêmero de uma imagem construída.** Rejeitado pelo custo —
construtor, registro, três saltos de rede por execução — não pelo
argumento, que esta decisão mantém.

**Traduzir compose em manifestos.** Rejeitado: sem mapeamento fiel para
`build`, `healthcheck`, `depends_on`, volumes e perfis.

**Um runner mantido aquecido.** Rejeitado até a latência de partida ser
medida.

## O que escolhemos, e por quê

**O ambiente de verificação é um runner:** efêmero, puxa o commit, constrói
da fonte e sobe a aplicação — o que todo runner de CI faz. **A imagem do
runner é da plataforma**, publicada uma vez com as toolchains e guardada em
cache nos nós. **Dependências de terceiros são puxadas como imagens
publicadas**, declaradas pelo projeto em poucas linhas; nada de terceiro é
construído. **O volume de cache da conta é montado no runner** —
`node_modules`, o cache de build do Go, o Maven — então a primeira
execução de um projeto paga e as outras não.

**O runner é separado da sandbox**, criado quando uma execução começa e
destruído quando termina, nunca mantido ocioso. Ocupa o endereço da demanda
enquanto roda; execuções paralelas de uma demanda enfileiram; o endereço é
reconciliado se uma execução morre. "Rode este commit e me dê uma URL" é um
port, `VerificationRunner`, com adaptadores Kubernetes e Docker.

**Dois gatilhos:** o fim do desenvolvimento, automaticamente, quando o
processo de reação-como-dado existir para decidir; e o desenvolvedor
pedindo — caso em que a execução segura o ambiente depois das checagens
para o desenvolvedor usar. **A demanda não mantém aplicação rodando** fora
de uma verificação ou de uma execução segurada.

## O que custou

Uma imagem de runner que é artefato de produto com cadência de release, e a
deriva de versões das toolchains dos clientes como um fardo de manutenção
assumido em vez de empurrado para eles. Os projetos declaram as
dependências em poucas linhas.

## Desde então

Os dois gatilhos e "sem aplicação rodando" foram acrescentados em
2026-09-04 e 2026-09-05. O port tem as suas garantias numa spec; o runner
ainda não está construído.
