---
title: "Um dono só, e a empresa se prova por DNS"
translationKey: "decision-0002"
adr: "0002"
adr_title: "Tenancy: the account owns everything, and an organization proves itself by domain"
adr_file: "0002-account-as-unit-of-ownership.md"
date: 2026-08-29
weight: 2
group: "identity"
description: "Três perguntas chegaram juntas — quem é dono das coisas, quando a multi-tenancy é construída, como uma organização prova que é uma — e respondê-las separadas tinha produzido três documentos. Um modelo respondeu às três."
related: ["0009", "0019", "0020"]
---

## O que estava na mesa

Três perguntas chegaram de uma vez, e a primeira tentativa respondeu em três
registros separados.

**Quem é dono das coisas.** Uma pessoa é dona de integrações, workspaces e
projetos; uma organização também. Os requisitos carregavam uma tensão: uma
integração "só pode ser vista e manipulada por aquele usuário", mas um membro
de uma organização "tem acesso controlado a todas" as integrações dela.
Precisávamos de um modelo em que as duas frases fossem verdadeiras ao mesmo
tempo, sem caso especial por situação.

**Quando a multi-tenancy é construída.** A documentação anterior fixava o
produto como mono-usuário, vários projetos em paralelo, sem RBAC — a
ferramenta local de um desenvolvedor. A direção tinha mudado: usuários com a
própria autenticação, organizações com membros, papéis, acesso por
integração, rodando num cluster ou na nuvem.

**Como uma organização prova que é uma.** O requisito original pedia, na
criação, validar se o CPF de quem está logado é o dono da empresa ou tem uma
procuração — citando GitHub e Google Cloud como referências de fluidez, com a
instrução "não invente, não dificulte". Então fomos ver o que essas
referências fazem de fato. O GitHub cria uma organização na hora, de graça,
sem nenhuma checagem de propriedade; a verificação vem depois, é do *domínio*
(um registro TXT), e rende um selo. O Google Cloud exige um domínio verificado
— também por DNS. Nenhum pede CPF ou procuração. As duas metades do requisito
puxavam em direções opostas.

## Os caminhos que pesamos

**Um dono polimórfico** — `ownerType: user | org` mais um id em cada recurso.
Modela o texto do requisito ao pé da letra. Rejeitado: toda consulta precisa
de dois campos e um desvio, as regras de acesso são escritas duas vezes, e
mover um recurso de uma pessoa para uma organização vira migração em vez de
update. O GitLab teve esse modelo e migrou para longe dele.

**Namespaces aninháveis** — uma árvore genérica com herança em qualquer
profundidade. Rejeitado por YAGNI: a hierarquia pedida é fixa e tem três
níveis.

**Continuar mono-usuário e adicionar tenancy depois.** Mais rápido até o
produto. Rejeitado: encaixar isolamento depois está entre as migrações mais
caras que existem — toda consulta escrita sem filtro de conta é um vazamento
em potencial, e o custo cresce com o código.

**Construir tudo antes de voltar ao produto.** Rejeitado pela razão oposta:
atrasa o produto que justifica a plataforma.

**Validar propriedade por CPF.** Rejeitado: precisa de integração com um
cadastro empresarial, lida mal com procuração (um documento que um humano
precisa ler) e cria atrito exatamente onde se pediu fluidez.

**Criação livre sem verificação nenhuma.** Rejeitado porque deixa a disputa de
handle sem resposta — e o namespace compartilhado torna essa disputa
inevitável.

## O que escolhemos, e por quê

Uma **`Account`**, pessoal ou organização, é a única unidade de posse. Tudo o
que tem dono carrega um `accountId` e nada mais; uma pessoa se liga a uma
conta por uma `Membership` com um papel; a conta pessoal nasce junto com o
usuário. A tensão dos requisitos se dissolve por construção: a integração de
uma conta pessoal é privada porque a conta tem um membro; a de uma organização
é compartilhada porque tem vários. Nenhum código precisa saber a diferença.

O modelo **nasce completo na primeira migração** — toda entidade carrega
conta, workspace e projeto; toda chamada resolve uma conta ativa — enquanto só
a autenticação e a conta pessoal são construídas agora. As organizações chegam
depois *sem migração*, porque o schema já as esperava. E a fase um exercita a
multi-tenancy de verdade: toda consulta filtra por conta desde o primeiro
dia; a conta simplesmente é sempre pessoal.

Uma organização é **criada na hora** — um nome e um CNPJ que preenche a razão
social — e **verifica o domínio depois**, opcionalmente, publicando um
registro TXT. A verificação destrava deliberadamente pouco: entrada automática
de quem tem e-mail `@domínio`, o selo de verificada, e o direito de contestar
um handle que outro pegou. Todo o resto funciona sem ela.

## O que custou

Uma conta pessoal implícita que a pessoa nunca pediu. Um namespace de handles
compartilhado: se alguém pega `acme` como conta pessoal, a organização Acme não
pode — mitigado pela verificação, não removido. Um escopo maior antes de
qualquer valor de produto: autenticação, contas, papéis, grants. CPF e CNPJ
entram no sistema, com as obrigações de proteção de dados que isso implica. E
uma linha que escrevemos para ninguém se enganar: controlar uma zona de DNS
não prova representação legal. Se um dia uma obrigação contratual precisar
disso, é outra decisão, em outra camada.

## Desde então

Em 2026-09-04 os três registros foram fundidos em um — os dois números
absorvidos foram aposentados, e em 2026-09-17 a sequência inteira foi
renumerada para os buracos não lerem como erros. O convite
([ADR-0019](../invite-without-token/)) e o segundo fator
([ADR-0020](../second-factor-in-the-core/)) se apoiam na conta ser a
fronteira: é o que uma membership concede, e o que uma política pode exigir
segundo fator para operar.
