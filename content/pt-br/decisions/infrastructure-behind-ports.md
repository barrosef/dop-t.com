---
title: "Todo port tem dois adaptadores e um contrato"
translationKey: "decision-0001"
adr: "0001"
adr_title: "Infrastructure behind ports with pluggable adapters"
adr_file: "0001-infrastructure-behind-ports.md"
date: 2026-08-29
weight: 1
group: "foundations"
description: "O domínio nunca toca um fornecedor. Cada peça de infraestrutura é um port que o domínio define, com pelo menos dois adaptadores e uma suíte de contrato que ambos precisam passar — é o que deixa o mesmo binário rodar no Google Cloud e num cluster no laptop."
related: ["0012", "0016", "0020"]
---

## O que estava na mesa

A plataforma roda em dois tipos de lugar desde o começo: no Cloud Run do
Google e num cluster Kubernetes operado por outra pessoa — k3s, Rancher,
OKD. Cada lugar oferece um serviço diferente para a mesma necessidade:
Secret Manager ou um Secret do Kubernetes; Cloud Storage ou um sistema de
arquivos; Identity Platform ou um provedor OIDC. O domínio não podia ser
escrito contra nenhum deles.

## Os caminhos que pesamos

**Acoplar a uma nuvem agora, abstrair depois.** Mais rápido até o primeiro
resultado; rejeitado porque rodar num cluster no laptop é um requisito de
desenvolvimento, não uma ambição futura.

**Uma biblioteca genérica multi-cloud.** Rejeitada: abstrai os fornecedores
da biblioteca, não o domínio, e troca um acoplamento por outro.

**Ports e adaptadores, com disciplinas junto.** A escolha — com as regras que
impedem "hexagonal" de ser só o nome de uma pasta.

## O que escolhemos, e por quê

O domínio alcança a infraestrutura só por um port que ele define, no seu
próprio vocabulário — `SecretStore.Get(ref)`, não `AccessSecretVersion`. Os
adaptadores são ligados num único *composition root* e escolhidos por
configuração; nenhum condicional sobre o ambiente existe em outro lugar.

Três regras tornam isso real:

1. **Dois adaptadores por port, desde o primeiro dia** — um de produção e um
   local. O adaptador local é a prova de que o port tem a forma do domínio e
   não a do primeiro fornecedor.
2. **Uma suíte de contrato por port**, que todo adaptador passa.
3. **Um port só carrega o que todo adaptador consegue garantir.** Versões de
   segredo ficam fora; *claims* específicos de um provedor ficam fora; o
   port de identidade devolve um principal normalizado.

Os ports vêm em duas famílias, e a diferença importa para a fiação: os de
infraestrutura — segredos, identidade, armazenamento, o barramento de
eventos — são escolhidos uma vez, na inicialização, pelo deploy; os de
provedor — hospedagens git, gerenciadores de tarefa, fornecedores de modelo
— são escolhidos a cada requisição pela configuração da conta, com vários
ativos ao mesmo tempo.

E uma regra para o dia em que um adaptador não consegue cumprir uma garantia
nativamente: **o adaptador paga; a garantia do port não é rebaixada.** O
armazenamento de segredos promete *read-after-write*. O Secret Manager do
Google só garante isso ao ler uma versão pelo número, então o adaptador
confirma a escrita pela versão, espera o alias `latest` convergir, e recusa
com um erro explícito se não convergir — em vez de responder "não
encontrado" para uma credencial recém gravada.

## O que custou

Dois adaptadores para cada port, escritos e mantidos antes de qualquer um
ser estritamente necessário. Mais uma indireção em cada chamada de
infraestrutura. A capacidade mais forte de um fornecedor fica inalcançável
do domínio por construção. E uma escrita no Secret Manager é mais lenta que
num Secret do Kubernetes, com um modo de falha que o emulador local não
reproduz.

## Desde então

A regra de *read-after-write* para o Secret Manager foi acrescentada em
2026-09-04. Todo port posterior — o provedor de agente, o canal de SMS, o
repositório do projeto, o runner de verificação — nasceu sob estas três
regras.
