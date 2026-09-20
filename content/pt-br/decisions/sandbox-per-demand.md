---
title: "Uma microVM por demanda, um worktree compartilhado pelas threads"
translationKey: "decision-0017"
adr: "0017"
adr_title: "A microVM per demand, with a single shared worktree"
adr_file: "0017-sandbox-per-demand.md"
date: 2026-08-31
weight: 17
group: "delivery"
description: "A fronteira dura é entre contas e entre demandas, então cada demanda ganha uma sandbox e os seus agentes dividem um workspace. A verificação nunca roda ali: roda num ambiente separado construído de um commit."
related: ["0005", "0007", "0021", "0023"]
---

## O que estava na mesa

Uma demanda tem um agente principal e vários subagentes. Onde cada um roda,
e onde roda a aplicação sob teste, são perguntas separadas com
necessidades de isolamento diferentes: os agentes de uma demanda confiam
uns nos outros; demandas e contas não; e um teste precisa falar de um
commit, não do que a árvore de trabalho do agente contém no momento.

## Os caminhos que pesamos

**Uma microVM por thread.** Rejeitada: N vezes a sobrecarga para isolar
agentes que cooperam.

**Um worktree git por thread.** Adiado: todo comando teria que saber em qual
árvore roda; mantido como a evolução mapeada se duas threads editarem os
mesmos arquivos com frequência.

**Testes dentro da sandbox do agente.** Rejeitado: uma árvore de trabalho
não é um commit, e o ambiente do agente não é limpo.

## O que escolhemos, e por quê

**Uma microVM por demanda.** A fronteira que precisa ser dura é entre
contas e entre demandas, e o banco impõe uma sandbox viva por demanda. **Um
worktree, `/workspace`, compartilhado pelas threads da demanda:** a maioria
das threads lê — um log, um banco, fonte — e quem edita é tipicamente o
agente principal.

**A fronteira do conhecimento é o projeto**, não a demanda: toda sandbox de
um projeto clona o repositório de conhecimento do projeto em `/project`. A
fronteira do workspace continua por demanda.

**A verificação não roda na sandbox.** Roda num runner efêmero construído de
um commit, ocupa o endereço da demanda enquanto roda —
`<service>--<demand>.<domain>` — e execuções paralelas de uma demanda
enfileiram. Os níveis de isolamento são honestos: pedir isolamento por
hardware onde o executor não consegue fornecer é recusado, nunca rebaixado
em silêncio.

## O que custou

Dois ciclos de vida de computação — a sandbox longa da demanda e o runner
curto da verificação. Um teste exige um commit publicado, o que faz do
fluxo do agente editar, commitar, verificar. O nível de hardware só é
exercitado em clusters que tenham runtime de microVM.

## Desde então

O endereço passou a ser por demanda com execuções enfileiradas em
2026-09-03, e o mecanismo de verificação virou [o
runner](../verification-runs-from-source/) no dia seguinte.
