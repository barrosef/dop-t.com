---
title: "O ambiente local roda o SDK da produção, e o Terraform é dono do resto"
translationKey: "decision-0015"
adr: "0015"
adr_title: "Firebase emulators in the local environment; Terraform as the single owner"
adr_file: "0015-firebase-emulators-and-single-owner.md"
date: 2026-08-30
weight: 15
group: "foundations"
description: "Os emuladores do Firebase fornecem identidade e armazenamento de objetos localmente com o mesmo SDK e os mesmos arquivos de configuração da produção. A infraestrutura é criada só pelo Terraform, então não existe nada que o estado não conheça."
related: ["0001"]
---

## O que estava na mesa

O ambiente local precisa de identidade e de armazenamento de objetos com a
mesma semântica da produção, para que o que funciona no laptop funcione na
nuvem. E a infraestrutura não pode divergir entre o que o Terraform gerencia
e o que existe de fato.

## Os caminhos que pesamos

**O emulador de Auth do Firebase mais o MinIO para objetos.** Rejeitado:
dois clientes diferentes — um S3 local contra o GCS em produção — e duas
semânticas de URL assinada.

**O Emulator Suite do Firebase para os dois.** A escolha.

## O que escolhemos, e por quê

**O Emulator Suite do Firebase fornece autenticação e armazenamento
localmente**, com o SDK de produção selecionado por uma variável de
ambiente. O MinIO não é usado; fica como possível terceiro adaptador do port
de armazenamento para um cliente auto-hospedado sem Google Cloud.

**O emulador persiste entre reinícios** — exportação ao sair, importação
condicional, um período de tolerância — para os dados de um desenvolvedor
sobreviverem a uma sessão.

**A configuração do emulador é a configuração do deploy:** `firebase.json`,
`.firebaserc` e as regras ficam versionados e montados como somente-leitura
no emulador. Não há uma segunda configuração que possa discordar da
produção.

**O Terraform é o único dono do que gerencia.** Nada é criado pelo console;
o que a CLI do Firebase publica vive em arquivos versionados que o Terraform
referencia ou importa; onde a posse é ambígua, o README da infraestrutura
declara um único dono por recurso. Um recurso adotado de fora é importado
até o `terraform plan` não reportar mudanças.

## O que custou

A CLI do Firebase vira dependência de desenvolvimento, e a tabela de donos é
mantida à mão.

## Desde então

A receita operacional saiu do registro para a spec de infraestrutura em
2026-09-04, deixando só a decisão. A regra do dono único foi aplicada ao
ambiente de QA no Google Cloud em setembro, quando todo recurso foi importado
para o estado do Terraform.
