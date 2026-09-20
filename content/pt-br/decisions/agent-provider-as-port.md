---
title: "Um port por fornecedor de modelo, rodando onde a credencial está"
translationKey: "decision-0016"
adr: "0016"
adr_title: "The agent runtime: a port per vendor, running inside the core"
adr_file: "0016-agent-provider-as-port.md"
date: 2026-08-31
weight: 16
group: "agents"
description: "O runtime que fala com um modelo não vê SDK de fornecedor: envia uma conversa pelo port AgentProvider e o recurso da conta decide qual adaptador responde. Roda no core, ao lado do vault, então a credencial de um provedor nunca cruza a rede."
related: ["0001", "0008", "0009", "0012"]
---

## O que estava na mesa

Duas perguntas sobre o runtime — o componente que conversa com um modelo.
Com quais fornecedores ele fala: uma conta pode ter vários provedores de
agente, Claude e Codex entre eles, e a plataforma não pode ser escrita
contra um só. E onde ele roda: precisa da credencial do provedor, que mora
no vault dentro do core e nunca é devolvida por ele.

## Os caminhos que pesamos

Para o fornecedor: **um único adaptador, abstraído depois** — rejeitado,
como para todo port; **uma API compatível com OpenAI como denominador
comum** — rejeitada, porque a compatibilidade acaba exatamente onde a
plataforma precisa de precisão: cache de prefixo, formato de ferramentas,
contagem de tokens.

Para o lugar: **a borda com acesso próprio ao vault** — rejeitado, a borda
é exposta à internet e comprometê-la não pode expor as chaves de provedor
de todas as contas; **um token efêmero de provedor emitido pelo core** —
impossível, chaves de provedor são duráveis; **um serviço separado para o
runtime** — uma terceira fronteira de confiança para o mesmo problema; **a
credencial numa variável de ambiente** — sem isolamento por conta,
atribuição ou revogação.

## O que escolhemos, e por quê

**`AgentProvider` é um port com um adaptador por fornecedor.** O runtime
envia uma conversa — um prefixo estável, mensagens, ferramentas — e recebe
uma resposta com consumo e um motivo de parada; nenhum tipo de SDK cruza o
port. O adaptador é escolhido **por requisição, pelo recurso da conta**,
com vários ativos ao mesmo tempo — a segunda família de ports. O roteador
escolhe a classe de modelo; o catálogo do adaptador ativo resolve o nome
concreto, então a política de custo vale para todo fornecedor sem conhecer
nenhum.

**O runtime roda no core.** A credencial é lida do vault e usada no mesmo
processo. As operações de um turno — montar o contexto, rotear, registrar
consumo, postar uma mensagem, publicar um achado, respeitar o orçamento —
são as próprias do core, em processo. A borda autentica, agrega e traduz;
rodar um turno é uma chamada, e o acompanhamento ao vivo é o SSE existente
alimentado pelos eventos do core.

O que nenhum fornecedor consegue garantir fica fora do port e está
documentado nele: semântica de cache de prefixo, formato de chamada de
ferramenta, eventos de streaming, motivos de parada. Dois adaptadores e uma
suíte de contrato, como todo port.

## O que custou

O comportamento do cache de prefixo difere por fornecedor, então a economia
da decisão de custo é por fornecedor. O core faz chamadas externas longas,
então orçamento de conexões e timeouts são preocupação dele. E a regra da
fronteira da borda ganhou a sua segunda metade: sem banco, **sem segredo**.

## Desde então

Dois registros — o port e a localização do runtime — foram consolidados em
2026-09-04. "Sem segredo" é uma das cinco invariantes sobre as quais todo
contribuidor é instruído, e a razão de o segundo fator manter as sementes no
core.
