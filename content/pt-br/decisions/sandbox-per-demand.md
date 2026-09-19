---
title: "Cinco microVMs, não vinte — e uma frase que precisamos retirar"
translationKey: "decision-0017"
adr: "0017"
adr_title: "A microVM per demand, with a single shared worktree"
adr_file: "0017-sandbox-per-demand.md"
date: 2026-08-31
weight: 17
group: "delivery"
description: "Onde cada agente roda, e onde roda a aplicação sob teste? A fronteira dura é entre contas e demandas, não entre threads que colaboram. A terceira cláusula desta decisão não sobreviveu."
related: ["0005", "0007", "0021", "0023"]
---

## O que estava na mesa

Uma demanda tem um agente principal e N subagentes — um lendo um log, outro
o banco, outro o código. Precisávamos decidir onde cada um roda, e onde roda
a aplicação sob teste. Três problemas chegaram juntos.

**Isolamento entre threads.** Uma microVM por thread dá isolamento forte — ao
custo de vinte microVMs num projeto com cinco demandas de quatro threads
cada, todas com o próprio kernel e centenas de megabytes de sobrecarga.

**Portas.** Subir a mesma aplicação mais de uma vez num ambiente significa
arbitrar uma porta e ainda expor uma rota para um humano olhar.

**De qual código o teste fala.** Foi este que decidiu.

## O que escolhemos, e por quê

**Uma microVM por demanda.** A fronteira que precisa ser dura é entre contas
e entre demandas, e essa é a que a sandbox por demanda garante. As threads de
uma demanda são agentes da mesma conta trabalhando no mesmo problema:
mutuamente confiáveis. Gastar uma microVM entre elas seria usar uma
ferramenta de segurança para resolver um problema de coordenação. O banco já
impunha isso: uma sandbox viva por demanda.

**Um worktree, compartilhado.** Um worktree por thread resolveria a colisão
de arquivos e acrescentaria complexidade real — todo comando teria que saber
em qual árvore roda. O risco foi aceito explicitamente: duas threads que
*editam* o mesmo arquivo se atropelam. Tolerável, porque a maioria das
threads lê — um log, um banco, fonte — e quem edita é tipicamente o agente
principal. Se a prática mostrar o contrário, a saída está mapeada.

**Um pod efêmero por execução de verificação.** O argumento veio do nosso
próprio código: o commit de uma execução de verificação é obrigatório, e a
recusa diz *"evidência que não diz em qual código rodou não é evidência."*
Um teste dentro da sandbox do agente roda contra a árvore de trabalho suja,
que não é commit nenhum. Um pod construído de um commit testa exatamente o
que vai ser mergeado.

## O que custou

Dois tipos de computação — a sandbox longa do agente e o pod curto da
verificação — com ciclos de vida diferentes de propósito. Um teste agora
exige um commit publicado antes de rodar, o que muda o fluxo do agente para
editar, commitar, verificar — mais perto do que um humano faz. E duas
perguntas ficaram escritas em aberto: quando provisionar, e onde o nível de
microVM poderia sequer ser validado, já que o cluster local não tem runtime
Kata e se recusa honestamente a fingir.

## Desde então

A terceira cláusula não sobreviveu, e o registro guarda a sequência inteira.
Em 2026-09-03 o idealizador decidiu que o endereço fica por demanda e as
execuções paralelas de verificação *enfileiram* — simplicidade acima de
latência dentro de uma demanda. A redação dessa decisão então acrescentou uma
frase que o idealizador não tinha dito: que a verificação "simplesmente roda
na sandbox da demanda". Isso pôs a execução de volta na árvore suja que esta
mesma decisão tinha recusado. Foi um erro de redação, corrigido no dia
seguinte pela [ADR-0023](../verification-runs-from-source/), que mantém o
argumento — a evidência precisa nomear um ambiente limpo construído de um
commit — e muda o mecanismo: não um pod de uma imagem construída, mas um
runner que puxa o commit e constrói da fonte. O que ainda vale aqui é a
própria sandbox: uma por demanda, um worktree compartilhado pelas suas
threads.
