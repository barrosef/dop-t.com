---
title: "A conta é dona de tudo"
translationKey: "decision-0002"
adr: "0002"
adr_title: "Tenancy: the account owns everything, and an organization proves itself by domain"
adr_file: "0002-account-as-unit-of-ownership.md"
date: 2026-08-29
weight: 2
group: "identity"
description: "Uma entidade, pessoal ou organização, é dona de toda integração, workspace e projeto; toda tabela carrega o seu id desde a primeira migração. Uma organização é criada na hora e se prova depois, por um registro de DNS."
related: ["0009", "0019", "0020"]
---

## O que estava na mesa

A plataforma é multi-tenant desde o começo: pessoas com a própria
autenticação; organizações com membros, papéis e acesso por recurso; tanto
pessoas quanto organizações donas de integrações, workspaces e projetos.
Duas restrições deram forma ao modelo: a integração de uma conta pessoal
precisa ser privada enquanto a de uma organização é compartilhada sob
controle — sem caso especial por situação — e criar uma organização não pode
exigir papelada, mas ela ainda precisa poder provar que é a empresa que diz
ser.

## Os caminhos que pesamos

**Um dono polimórfico** — `owner_type` mais `owner_id` em cada recurso.
Rejeitado: dois campos e um desvio em toda consulta, regras de acesso
escritas duas vezes, e uma transferência que vira migração.

**Namespaces aninháveis** com herança em qualquer profundidade. Rejeitado: a
hierarquia é fixa e tem três níveis.

**Mono-usuário primeiro, tenancy depois.** Rejeitado: encaixar isolamento
depois é a migração cara.

**Tudo construído antes de voltar ao produto.** Rejeitado: atrasa o produto
que justifica a plataforma.

**Propriedade validada por CPF ou procuração na criação.** Rejeitado:
atrito, integração com cadastro, e nenhum produto de referência faz isso —
GitHub e Google Cloud verificam um domínio, depois.

**Nenhuma verificação.** Rejeitado: um namespace de handles compartilhado
precisa de um caminho de disputa.

## O que escolhemos, e por quê

Uma **`Account`**, pessoal ou organização, é a única unidade de posse. Tudo o
que tem dono carrega um `account_id` e nenhum outro campo de dono; uma pessoa
se liga a uma conta por uma `Membership` com um papel; a conta pessoal é
criada com o usuário. A integração de uma conta pessoal é privada porque a
conta tem um membro; a de uma organização é compartilhada porque tem vários
— nenhum código precisa saber a diferença.

O modelo é **completo desde a primeira migração** — toda entidade carrega
conta, workspace e projeto; toda chamada resolve uma conta ativa — enquanto
a fase um constrói a autenticação, a conta pessoal e a hierarquia workspace
→ projeto. Organizações, membros, papéis e grants chegam depois sem
migração.

Uma organização é **criada na hora**: um nome e um CNPJ, que preenche a razão
social e o endereço. Ela **verifica o domínio depois**, opcionalmente,
publicando um registro TXT no DNS. A verificação destrava exatamente três
coisas: entrada automática de quem tem e-mail naquele domínio, o selo de
verificada, e o direito de contestar um handle que outro alguém tem. Todo o
resto funciona sem verificação.

## O que custou

Uma conta pessoal implícita. Um namespace de handles compartilhado, em que a
verificação mitiga a disputa mas não a remove. Um escopo maior antes de
qualquer valor de produto — autenticação, contas, papéis, grants. CPF e CNPJ
no sistema, com as obrigações de proteção de dados que vêm junto. E uma linha
que vale manter à vista: controlar uma zona de DNS prova controle do DNS, não
representação legal.

## Desde então

O modelo foi consolidado num único registro em 2026-09-04. O convite e o
segundo fator se apoiam em a conta ser a fronteira: é o que uma membership
concede, e o que uma política pode exigir segundo fator para operar.
