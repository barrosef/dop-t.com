---
title: "O core verifica uma assinatura em toda chamada"
translationKey: "decision-0022"
adr: "0022"
adr_title: "The core verifies a signature; it does not believe a header"
adr_file: "0022-the-core-verifies-its-callers.md"
date: 2026-09-03
weight: 22
group: "identity"
description: "Uma chamada com pessoa carrega o token da pessoa; uma sem pessoa carrega uma asserção assinada pela plataforma. A fronteira de rede é adicional — IAM do Cloud Run na nuvem, uma network policy no cluster — nunca a única garantia."
related: ["0001", "0004", "0012", "0013"]
---

## O que estava na mesa

O core resolve quem está chamando, e em qual conta, a partir dos metadados
da chamada. Uma fronteira de rede sozinha — só a borda alcança a porta gRPC
— é uma propriedade de um deploy, e esta plataforma roda código de agente
dentro do mesmo cluster de propósito. O core precisava de uma garantia que
pertencesse ao software.

## Os caminhos que pesamos

**Confiar nos metadados.** Rejeitado: quem consegue abrir uma conexão
reivindica qualquer ator de qualquer conta.

**Um segredo compartilhado num header.** Rejeitado: prova que quem chama
conhece um segredo, não o que afirma.

**mTLS.** Rejeitado em clusters: emissão e rotação, e prova a conexão em vez
da afirmação. No Cloud Run o seu equivalente — a permissão de *invoker* do
IAM — não custa nenhum dos dois, e é usado.

**Uma asserção assinada pela plataforma.** Adotada para chamadas sem pessoa
por trás.

**Encaminhar o token da pessoa.** Adotado para chamadas com pessoa: a
assinatura é do provedor de identidade, uma autoridade que nenhuma das
pontas controla, e o core já tinha a maquinaria para verificá-la.

**Identity-Aware Proxy.** Rejeitado: autoriza por política de IAM, que não
expressa contas, papéis e grants por tenant, e quebra um cockpit de página
única em outra origem. Cabe em superfícies só da equipe.

## O que escolhemos, e por quê

**Qual assinatura depende de quem chama.** Uma chamada com pessoa carrega o
token da pessoa, encaminhado inteiro; o core o verifica pelo port de
identidade e tira o ator do *subject* do token. Uma chamada sem pessoa
carrega uma asserção que a borda assina — chamador, ator, tipo, conta,
sessão, validade — com uma chave por chamador, para que um componente
comprometido forje só as próprias chamadas, e a assinatura cobre a
afirmação, para que uma asserção roubada valha um ator numa conta por dois
minutos.

**A conta sempre vem da asserção**, nunca do token, que não a carrega.
**Token e asserção nomeando atores diferentes recusam a chamada.** Uma
chamada recusada segue sem ator e falha na autorização, onde a mensagem
significa algo.

**Três modos:** `strict` se recusa a preencher um ator sem assinatura
verificada, `permissive` avisa e segue, `off` confia nos metadados. O
padrão do código é permissivo, para um clone funcionar sem uma borda na
frente; o deploy roda estrito.

**A autenticação de transporte é adicional.** No Cloud Run o core não aceita
invocação não autenticada e só a conta de serviço da borda pode invocá-lo —
o token de *invoker* viaja em `X-Serverless-Authorization` porque o
`Authorization` carrega o da pessoa. Num cluster uma network policy restringe
quem alcança a porta. Nenhuma das duas substitui a assinatura.

## O que custou

A borda encaminha o token bruto e assina uma asserção em toda chamada. Um
HMAC e uma verificação de token por chamada, com as chaves de assinatura e a
busca do *subject* em cache. Uma validade de dois minutos assume que os dois
relógios concordam. E duas coisas que este desenho não resolve: uma asserção
é reproduzível dentro da janela, e uma borda comprometida assina as próprias
afirmações — ambas inerentes, ambas a razão de o token da pessoa ser
preferido onde existe uma pessoa. O formato de fio é fixado por um vetor de
teste afirmado em Go e em Python.

## Desde então

O IAM do Cloud Run tomou o lugar da network policy no deploy gerenciado em
2026-09-07; o cluster mantém a policy.
