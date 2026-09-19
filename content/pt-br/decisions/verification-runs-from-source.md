---
title: "Três documentos discordavam, e o caminho barato era construir da fonte"
translationKey: "decision-0023"
adr: "0023"
adr_title: "Verification builds from source in a runner, not from an image"
adr_file: "0023-verification-runs-from-source.md"
date: 2026-09-04
weight: 23
group: "delivery"
description: "Um pod roda uma imagem, e uma imagem precisa de um construtor e de um registro no caminho crítico de toda execução. Cortar os três saltos em volta do registro deixa o que todo runner de CI do mundo faz."
related: ["0001", "0003", "0005", "0017"]
---

## O que estava na mesa

Três documentos discordavam sobre onde a aplicação de uma demanda roda, e a
discordância era nossa para resolver. A spec de execução dizia que um Docker
interno sobe a stack dentro da sandbox. A decisão da sandbox dizia que uma
verificação roda num pod efêmero, porque um teste dentro da sandbox roda
contra uma árvore suja. E a redação de uma decisão posterior do idealizador
tinha posto a execução de volta na sandbox — uma frase que o idealizador
nunca disse.

Resolver isso expôs o custo real de "um pod no cluster". Um pod roda uma
*imagem*, então a verificação precisaria: ler o compose do projeto, traduzir
em manifestos, construir a imagem da aplicação a partir do commit, empurrar
para um registro, e puxar de volta num nó. Um construtor e um registro que
não existiam, no caminho crítico de toda execução.

## Os caminhos que pesamos

**Compose dentro da sandbox** — a spec original. O mais barato de construir.
Rejeitado como caminho de verificação: o ambiente é do agente, com o que ele
instalou pelo caminho, e evidência dali fala daquele ambiente, não de um
limpo. Continua candidato para a bancada do próprio desenvolvedor.

**Um pod efêmero de uma imagem construída** — o mecanismo da decisão da
sandbox. Ambiente honesto; custa um construtor, um registro e três saltos de
rede por execução. Rejeitado pelo custo, não pelo argumento.

**Traduzir compose em manifestos.** Funciona no caso simples e mente no
resto: `build`, `healthcheck`, `depends_on`, volumes e perfis não têm
equivalente limpo, e o desenvolvedor acaba depurando um manifesto que nunca
escreveu.

**Um runner mantido aquecido.** Sobe mais rápido, e queima dinheiro enquanto
nada acontece. Revisitar com uma latência medida.

## O que escolhemos, e por quê

**O ambiente de verificação é um runner**: efêmero, puxa o commit, constrói
da fonte e sobe a aplicação. Nenhuma imagem do projeto é jamais construída,
empurrada ou implantada. A sequência lenta nunca foi a construção — era
construir, empurrar, puxar, subir, três saltos em volta de um registro que só
existia para mover bytes entre dois lugares do mesmo cluster. Cortá-la deixa
puxar, construir, subir: o que todo runner de CI faz, e o que um
desenvolvedor faz na própria máquina. Também remove a questão do compose por
inteiro: não há descrição de stack para ler, só um repositório, um commit e
um comando.

**A imagem do runner é nossa, construída uma vez** — as toolchains vivem numa
imagem que publicamos e que o nó guarda em cache. Uma imagem gorda, de
propósito: uma imagem grande em cache em todo lugar vale mais que uma pequena
construída por demanda. **Dependências de terceiros são puxadas, nunca
construídas** — um banco é uma imagem publicada. **O volume de cache da conta
é montado no runner**, então a primeira execução de um projeto paga e as
outras não; sem isso, a decisão não se sustenta. **É separado da sandbox**,
de propósito: a evidência é honesta sobre o ambiente, o teste não compete com
o agente por recursos, e um runner que morre não leva nada da demanda junto.
Ele toma o endereço da demanda enquanto roda — um por demanda, execuções
paralelas enfileiram — e reconciliar esse endereço quando uma execução morre
faz parte do seu ciclo de vida. E "rode este commit e me dê uma URL" é um
port, com adaptadores Kubernetes e Docker, para que os dois executores não
divirjam justamente na coisa que produz evidência.

## O que custou

Uma imagem de runner que é artefato de produto nosso — versões, tamanho, uma
cadência de release — e a deriva de versões das toolchains dos nossos
clientes como um fardo de manutenção que assumimos em vez de empurrar para
eles. O projeto declara as dependências em poucas linhas: um imposto, o menor
disponível.

## Desde então

O dia seguinte fechou a pergunta que esta decisão abriu: **a demanda não
mantém aplicação rodando.** Ela existe só durante uma verificação, ou
enquanto um desenvolvedor pediu para olhá-la — o mesmo runner, sem checagens,
segurado até um prazo. A bancada é onde o código é escrito, não onde roda. A
opção mais barata e mais pobre, escolhida sabendo. Depois dois gatilhos foram
fixados: o fim do desenvolvimento, automaticamente, quando o processo de
reação-como-dado existir para dizer isso; e o desenvolvedor clicando em
*testar* e ficando com o preview — o que transformou "uma execução sem
checagens segura" em "uma execução segura quando foi pedido". O port do
runner tem forma e nove garantias numa spec; ainda não está construído.
