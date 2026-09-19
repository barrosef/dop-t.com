---
title: "O gargalo mudou para a mesa do revisor"
translationKey: "decision-0005"
adr: "0005"
adr_title: "No green, no PR: native verification, and a merge queue per repository"
adr_file: "0005-no-green-no-pr.md"
date: 2026-08-29
weight: 5
group: "delivery"
description: "Os agentes já fecham o ciclo até o pull request. Sem verificação antes do humano, o paralelismo só move a fila. E depois do verde, três PRs testados contra o main de ontem ainda podem quebrar a produção juntos."
related: ["0004", "0008", "0011", "0023"]
---

## O que estava na mesa

Duas falhas no mesmo caminho, e são consecutivas.

**Antes do humano.** A pesquisa era conclusiva: os agentes já fecham o ciclo
até o pull request, e o gargalo do fluxo tinha virado a capacidade humana de
revisão. Sem verificação nativa, o paralelismo de um executor só move a fila
— do desenvolvimento para a mesa do revisor. O produto já tinha fixado que um
merge é decisão humana; isto decide o que acontece antes de o humano ser
chamado.

**Depois do verde.** Paralelismo é requisito: três demandas ao mesmo tempo,
"independentemente de os repositórios se sobreporem". Três PRs verdes, cada
um testado contra o `main` de quando o seu branch nasceu. O primeiro merge
invalida os outros dois — na melhor hipótese um conflito de texto, na pior
uma quebra semântica silenciosa: um PR remove a checagem que o outro assumia.
O CI de um PR não vê isso. A produção vê. Frotas de agentes transformam esse
acidente mensal num diário.

## Os caminhos que pesamos

**Revisão só por humanos.** O padrão do mercado, e onde a frota afoga o
revisor.

**Merge automático no verde.** Rejeitado: o portão humano é um não-objetivo
fixado pelo produto, e o crítico não substitui responsabilidade.

**Merge otimista**, na ordem de chegada. Rejeitado: é exatamente o cenário da
quebra semântica.

**Um arquivo, um dono.** Rejeitado: mata o paralelismo que é requisito — uma
fila disfarçada.

**Só a fila de merge do provedor.** Rejeitada como rota única: nem todo
provedor tem uma, e a visão entre demandas é algo que o provedor não tem.

## O que escolhemos, e por quê

**Antes do PR, quatro regras em ordem.** A aceitação nasce na spec,
executável — um critério que não executa é um desejo. O agente itera até o
verde; nenhum PR abre com a aceitação falhando, e uma falha persistente vira
uma pergunta ao humano, nunca um PR quebrado. Um crítico revisa antes do
humano — uma instância independente, contexto limpo, sem o histórico de quem
implementou, recebendo diff, spec e evidência e emitindo um veredito: a
primeira linha de defesa contra o carimbo. E o PR carrega o pacote de
evidências, para que o humano revise a exceção, não a regra.

**Depois do verde, uma fila de merge por repositório, como conceito de
domínio.** Um PR verde entra na fila; a fila reaplica cada um sobre o `main`
atualizado, reexecuta a verificação e mergeia um de cada vez. Só o que está
verde contra o estado real entra. Um conflito é tarefa do agente primeiro, do
humano na escalada. A sobreposição é detectada cedo, antes do PR, por um
orquestrador que depois ganhou nome. E a fila nativa do provedor é usada onde
existe, com a da plataforma orquestrando por cima.

## O que custou

O crítico custa tokens — modelo forte, sem economia aqui. Um merge
serializado por repositório significa que a latência de entrega cresce com a
fila, então a posição e a previsão ficam visíveis no cockpit. Reverificar a
cada posição custa computação. E a sintaxe exata de um critério executável
ficou para o modelo de trabalho, não fixada aqui.

## Desde então

O registro foi escrito como dois — as regras antes do PR, a fila depois dele
— e fundido em um em 2026-09-04, porque são o mesmo caminho do verde ao
`main`. O artefato de spec em que os critérios vivem ganhou um endereço quando
o conhecimento do projeto virou um repositório
([ADR-0021](../project-knowledge-as-a-git-repository/)). E a pergunta de
*onde* a verificação roda — que esta decisão não fez — passou por duas
respostas antes de pousar num runner que constrói da fonte
([ADR-0023](../verification-runs-from-source/)).
