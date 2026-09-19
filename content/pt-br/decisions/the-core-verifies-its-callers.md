---
title: "Um header não é uma prova"
translationKey: "decision-0022"
adr: "0022"
adr_title: "The core verifies a signature; it does not believe a header"
adr_file: "0022-the-core-verifies-its-callers.md"
date: 2026-09-03
weight: 22
group: "identity"
description: "O core acreditava em duas linhas de metadados gRPC. Quem conseguisse abrir uma conexão era qualquer ator de qualquer conta. A única coisa no caminho era uma network policy — uma propriedade do deploy, não do software."
related: ["0001", "0004", "0012", "0013"]
---

## O que estava na mesa

O core lia `x-actor-id` e `x-account-id` dos metadados da chamada e
acreditava neles. Metadado é texto: quem conseguisse alcançar a porta 9090
podia se declarar qualquer ator de qualquer conta, sem nada além do header.

Tinha sido uma escolha deliberada — uma fronteira de confiança, na borda, e um
core simples. O que a pôs na mesa foi o *tipo* de garantia por trás dela. "Só
o BFF pode chamar" era imposto por uma NetworkPolicy, e verificamos em
2026-09-03, de um pod solto no namespace, que ela segurava: a porta gRPC
bloqueada, a de health aberta. Segurava *hoje, aqui*. Três coisas tornavam
isso insuficiente: é uma propriedade do deploy, não do software — um cluster
cujo CNI ignora network policies não tem fronteira nenhuma; falha em silêncio
— nada quebra, a porta simplesmente está aberta; e esta plataforma roda código
de agente dentro do mesmo cluster de propósito. "Alguém hostil dentro da rede"
não é hipótese aqui. É funcionalidade.

## Os caminhos que pesamos

Discutidos com o idealizador lado a lado, numa lousa. **Deixar como está:**
custo zero, e uma configuração errada é o sistema inteiro. **Um segredo
compartilhado num header:** barato, e só prova que quem chama conhece o
segredo — quem o tem continua reivindicando qualquer ator. **mTLS:** forte,
sem *bearer*, caro em emissão e rotação — e prova a *conexão*, enquanto a
pergunta era sobre a *afirmação*. **Uma asserção assinada pela plataforma:**
amarra a afirmação, funciona onde não há pessoa, é barata de verificar.
**Encaminhar o JWT da pessoa** — a proposta do idealizador, e a mais forte onde
se aplica: a assinatura é do provedor de identidade, uma autoridade que nenhuma
das pontas controla, e o core já tinha a maquinaria para verificá-la. Cobre só
as chamadas com uma pessoa por trás.

A pergunta que decidiu: não *quem abriu a conexão*, mas *quem tem a autoridade
para afirmar quem é o ator*.

## O que escolhemos, e por quê

O core verifica uma assinatura em toda chamada, e qual assinatura depende de
quem chama. Uma chamada com pessoa carrega o token da pessoa, encaminhado
inteiro; o core o verifica pelo port que já tinha, e o ator vem do *subject*
do token. Uma chamada sem pessoa carrega uma asserção que a borda assina —
chamador, ator, tipo, conta, sessão, validade — com uma chave por chamador,
para que um componente comprometido forje só as próprias chamadas, e a
assinatura cobre a *afirmação*, para que uma asserção roubada valha um ator
numa conta por dois minutos.

A conta nunca vem do token — não está lá — sempre da asserção. Quando os dois
estão presentes e nomeiam atores diferentes, a chamada é recusada: não é uma
preferência entre fontes, é um bug ou um ataque. Uma chamada recusada segue
sem ator e falha na autorização, onde a mensagem significa algo; falhar no
interceptor diria "não autenticado" sobre uma chamada cujo problema real é que
não provou nada.

Três modos, e o padrão não é o estrito: `permissive` avisa, `strict` recusa, e
o deploy roda estrito enquanto o código continua utilizável para quem clona.
Virar tudo de uma vez teria quebrado todo chamador que ainda não aprendeu a
assinar — a suíte de contrato incluída.

## O que custou

A borda mudou também: carrega o token bruto para encaminhá-lo, e assina uma
asserção em toda chamada. O formato de fio é fixado por um vetor de teste
compartilhado, afirmado num teste em Go e num em Python, porque duas
linguagens só concordam por acidente — e uma divergência aqui não falha alto;
faz toda chamada chegar sem autenticação. Um HMAC e uma verificação de token
por chamada, com as chaves de assinatura e a busca do *subject* em cache. Uma
validade de dois minutos assume que dois relógios concordam, o que num cluster
acontece; está escrito para o dia em que não acontecer. E o que não está
resolvido: uma asserção é reproduzível dentro da janela, e uma borda
comprometida assina o que quiser — ambos inerentes a assinar as próprias
afirmações, ambos a razão de o token de terceiro ser preferido onde existe
uma pessoa.

## Desde então

Em 2026-09-07, a mudança para o Cloud Run alterou a objeção ao mTLS. A
permissão de *invoker* do IAM é a resposta com o formato do mTLS sem emissão e
sem rotação: o Google assina, o Google verifica, antes de a requisição chegar
ao processo. Ela substituiu a NetworkPolicy — que fica no cluster local, onde
não há IAM — e não substituiu nada decidido acima: três camadas respondem a
três perguntas (este chamador pode invocar este serviço; qual componente
afirma qual ator; quem é a pessoa), e nenhuma responde à de outra. Uma colisão
precisou ser tratada: o Cloud Run lê o seu token do `Authorization`, que o
token da pessoa já ocupa, então o token de serviço viaja em
`X-Serverless-Authorization`.

O Identity-Aware Proxy foi pesado e recusado — e um rascunho anterior dessa
emenda estava errado sobre o porquê. Não é custo: o modo de identidades
externas do IAP é gratuito até cinquenta mil usuários. É que o IAP decide quem
pode alcançar um recurso por política de IAM, e esta plataforma deixa qualquer
um se cadastrar: a política admitiria todo mundo e não decidiria nada,
enquanto a pergunta que importa — qual conta, qual papel, quais grants — é
estado por tenant para o qual o IAM não tem vocabulário. Também quebraria o
cockpit, uma aplicação de página única em outra origem que não consegue seguir
um redirecionamento para uma tela de login. Onde o IAP cabe é numa superfície
que só a equipe usa.
