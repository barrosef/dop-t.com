---
title: "O emulador que não pode mentir"
translationKey: "decision-0015"
adr: "0015"
adr_title: "Firebase emulators in the local environment; Terraform as the single owner"
adr_file: "0015-firebase-emulators-and-single-owner.md"
date: 2026-08-30
weight: 15
group: "foundations"
description: "Duas lições pagas por um projeto irmão, escritas aqui antes de pagarmos por elas de novo: o ambiente local roda o mesmo SDK e a mesma configuração da produção, e nada é criado por console."
related: ["0001"]
---

## O que estava na mesa

O ambiente local precisava de identidade e de armazenamento de objetos. A
primeira proposta era o emulador do Firebase para autenticação e o MinIO para
objetos. São dois clientes diferentes — um S3 local contra o GCS em produção —,
duas semânticas de URL assinada, e a falha mais velha do livro: funciona na
minha máquina, quebra na nuvem.

Um projeto irmão já tinha pago por duas lições nessa área, e não queríamos
comprá-las uma segunda vez.

## O que escolhemos, e por quê

**O Emulator Suite do Firebase cobre os dois**, autenticação e armazenamento,
com o mesmo SDK da produção, resolvido por uma variável de ambiente. O MinIO
não entra; fica como terceiro adaptador do port de armazenamento para o dia em
que um cliente auto-hospedado não tiver Google Cloud.

**O emulador mantém os dados entre reinícios.** Um emulador que esquece tudo a
cada parada empurra os desenvolvedores de volta para a nuvem em qualquer coisa
que dure mais de uma sessão. A mecânica — exportar ao sair, importar
condicionalmente, um período de tolerância — é operacional e vive na spec de
infraestrutura; o que está decidido aqui é que o ambiente local *precisa*
sobreviver a um reinício.

**A configuração do emulador é a configuração do deploy.** `firebase.json`,
`.firebaserc` e as regras ficam versionados e montados como somente-leitura no
emulador — os mesmos arquivos que o deploy usa. Um emulador com configuração
própria mente sobre a produção.

**O Terraform é o único dono do que ele gerencia.** Essa foi a segunda lição do
projeto irmão: um recurso criado pelo console ou pela CLI não está no estado, e
o próximo `apply` o reverte ou apaga — uma funcionalidade perdida em silêncio.
Então nada é criado pelo console; o que a CLI do Firebase publica vive em
arquivos versionados que o Terraform referencia ou importa; e onde a fronteira
é ambígua — provedores de autenticação, por exemplo — o README da
infraestrutura declara um único dono por recurso, numa tabela explícita.

## O que custou

Uma dependência da CLI do Firebase em todo ambiente de desenvolvimento. E a
tabela de donos precisa ser mantida à mão; é o que impede a perda silenciosa
no `apply`, e ela só vale o que valeu a sua última edição.

## Desde então

A decisão foi enxugada em 2026-09-04: a receita operacional saiu do registro e
foi para a spec de infraestrutura, deixando só o que é decisão. A regra sobre
o console foi testada uma semana depois, quando o ambiente de QA no Google
Cloud foi codificado em Terraform a partir dos recursos que tinham sido
criados à mão nos primeiros deploys — cada importação precisou produzir um
plano sem mudanças antes de o ambiente contar como próprio.
