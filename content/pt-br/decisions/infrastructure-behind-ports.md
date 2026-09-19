---
title: "Dois adaptadores, ou não é um port"
translationKey: "decision-0001"
adr: "0001"
adr_title: "Infrastructure behind ports with pluggable adapters"
adr_file: "0001-infrastructure-behind-ports.md"
date: 2026-08-29
weight: 1
group: "foundations"
description: "A primeira decisão, e a que todas as outras se apoiam: o domínio nunca toca um fornecedor, e um port com um único adaptador é um palpite com o formato desse fornecedor."
related: ["0012", "0016", "0020"]
---

## O que estava na mesa

A plataforma tinha que rodar em dois lugares desde o começo: no Cloud Run do
Google e num cluster Kubernetes operado por outra pessoa — k3s, Rancher, OKD.
A lista do que ela precisaria do ambiente ia crescer: segredos, identidade,
armazenamento de objetos, persistência, mensageria. E em cada lugar a mesma
necessidade é atendida por um serviço diferente. Secret Manager de um lado; um
Secret do Kubernetes do outro.

O movimento tentador era escolher o Google, entregar e "portar depois". Já
tínhamos visto o que isso faz: *depois* é o momento em que o acoplamento já
está em todo lugar, e a primeira escolha — feita sob pressão, no primeiro dia —
vira o desenho.

## Os caminhos que pesamos

**Acoplar ao GCP agora, abstrair depois.** Mais rápido até o primeiro
resultado. Rejeitado, porque rodar num cluster no laptop não era uma ambição
futura, era um requisito de desenvolvimento desde a primeira semana.

**Uma biblioteca genérica multi-cloud.** Rejeitada por uma razão mais sutil:
uma biblioteca dessas entrega o denominador comum *dos fornecedores da
biblioteca*, não do seu domínio. Troca um acoplamento por outro, e o novo é
mais difícil de enxergar.

**Ports e adaptadores — com regras junto.** Foi o que venceu, mas só porque
fomos honestos: "hexagonal" costuma ser o nome de uma pasta. A palavra sozinha
não faz nada.

## O que escolhemos, e por quê

O domínio alcança a infraestrutura só por um port que ele mesmo define, na
sua própria língua: `SecretStore.get(ref)`, não `accessSecretVersion`. Os
adaptadores são escolhidos num único *composition root*, por configuração, e o
domínio não importa SDK de fornecedor nenhum.

Três disciplinas tornam isso real em vez de decorativo:

1. **Dois adaptadores por port, desde o primeiro dia.** O adaptador local não
   é "para depois" — é a prova de que o port está certo. Um port com um único
   adaptador sai com o formato do fornecedor que o inspirou.
2. **Uma suíte de contrato por port**, que todo adaptador precisa passar.
   Substituibilidade de fato, não de intenção.
3. **O que um adaptador não consegue prometer fica fora do port.** Versão de
   segredo fica fora (o Kubernetes não tem). Os *claims* do Firebase nunca
   cruzam a fronteira; o port de identidade devolve um principal normalizado.

Também notamos que "port" esconde duas coisas diferentes. Os ports de
infraestrutura — segredos, identidade, armazenamento, o barramento — são
escolhidos uma vez, na inicialização, pelo ambiente. Os ports de provedor —
git, gerenciadores de tarefa, depois os fornecedores de modelo — são
escolhidos a cada requisição pela configuração da conta, e vários ficam ativos
ao mesmo tempo. Confundir as duas famílias é o erro típico desse desenho, então
o registro as nomeia separadamente.

## O que custou

Dois adaptadores para cada port, escritos e mantidos, antes de qualquer um ser
estritamente necessário. Mais uma indireção em cada chamada de infraestrutura.
E a capacidade mais forte de um fornecedor fica inacessível ao domínio por
construção — de propósito.

## Desde então

A regra foi testada de verdade, e registrada em 2026-09-04; segurou na
direção que não tínhamos planejado. O `SecretStore` promete *read-after-write*. A promessa
nasceu do adaptador Kubernetes, onde é trivialmente verdadeira. O adaptador do
Google não conseguia cumprir: o Secret Manager só é fortemente consistente
quando você lê uma versão *pelo número*, e o `latest` converge "tipicamente em
minutos, mas pode levar algumas horas". No GCP de verdade, um `Get` logo depois
de um `Put` podia responder "isso não existe" — para uma credencial recém
gravada — e o emulador local nunca mostraria.

Afrouxar a promessa para "eventualmente consistente" foi rejeitado: empurra a
lógica de retentativa para todo chamador, que não consegue distinguir "ainda
não" de "nunca". Guardar o valor em cache no processo foi rejeitado: um segundo
lugar onde uma credencial vive. **A garantia ficou e o adaptador pagou**: ele
confirma a escrita pela versão, espera o alias alcançar, e recusa em voz alta
se não alcançar. Um `Put` que tem sucesso enquanto o `Get` seguinte diz "não
encontrado" é pior que um `Put` que falha — o primeiro produz uma integração
quebrada em silêncio, o segundo um erro que alguém lê.

Esse episódio foi escrito primeiro como uma ADR própria e depois fundido nesta:
não é uma decisão nova, é o que esta decisão significa quando dói.
