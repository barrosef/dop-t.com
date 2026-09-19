---
title: "O runtime estava do lado errado da credencial"
translationKey: "decision-0016"
adr: "0016"
adr_title: "The agent runtime: a port per vendor, running inside the core"
adr_file: "0016-agent-provider-as-port.md"
date: 2026-08-31
weight: 16
group: "agents"
description: "Dois erros pegos na mesma semana: um briefing que dizia 'chame a API da Anthropic', e uma divisão de aparência limpa que pôs a peça que precisa da credencial do modelo na camada aberta à internet."
related: ["0001", "0008", "0009", "0012"]
---

## O que estava na mesa

**O fornecedor.** Quando o runtime do agente — a peça que fala com o modelo —
foi especificado, o briefing dizia "escreva código que chame a API da
Anthropic". Estava errado, e o idealizador pegou: *"a plataforma tem que
pensar em isolamento com múltiplas opções de provedor, incluindo provedores
de agente; por exemplo Claude e Codex."* Era a decisão dos ports aplicada ao
lugar onde é mais fácil esquecer, porque o fornecedor do modelo parece o
produto e não infraestrutura. O modelo de dados já tinha antecipado e ninguém
tinha ligado os pontos: um provedor de agente já era uma integração de
categoria `agent` — um recurso com credencial — e o roteador de custo já
separava política (qual classe) de catálogo (qual modelo concreto).

**O lugar.** A decisão da stack tinha posto o runtime na borda em Python: o
core decide o quê, a borda roda a conversa. Limpo no papel. Na implementação
cobrou: o runtime precisa da credencial do provedor, e a credencial de um
recurso mora no vault, no core, que nunca entrega um segredo — por desenho,
com um teste vigiando.

## Os caminhos que pesamos

Para o fornecedor: **um único adaptador, trocar depois** — o que a decisão
dos ports existe para impedir; foi assim que o port de identidade passou
meses com um adaptador e escondeu um bypass de autenticação até alguém
escrever o segundo. **Uma camada compatível com OpenAI** como denominador
comum — rejeitada: a compatibilidade cobre o caso simples e vaza exatamente
onde a plataforma precisa de precisão: cache de prefixo, formato de
ferramentas, contagem de tokens.

Para o lugar, toda saída era ruim. **A borda com acesso próprio ao vault** —
a recomendação inicial da implementação, e o idealizador vetou com razão:
*"o BFF é uma camada muito insegura, aberta à internet."* Comprometê-la
entregaria as credenciais de agente de todas as contas; a sessão que discutiu
isso tinha acabado de encontrar um bypass total de autenticação nessa mesma
camada. **O core emitindo um token efêmero** — elegante, e impossível: uma
chave de API é durável, não há nada de curta duração para emitir. **Um
terceiro serviço só para o runtime** — um terceiro deploy e uma terceira
fronteira de confiança para o mesmo problema. **Ler de uma variável de
ambiente** — o paliativo que tinha sido entregue: sem isolamento por conta,
sem atribuição de custo, sem revogação.

## O que escolhemos, e por quê

**`AgentProvider` é um port com um adaptador por fornecedor.** O runtime
nunca vê um tipo de SDK: envia uma conversa — um prefixo estável, mensagens,
ferramentas — e recebe uma resposta com consumo e um motivo de parada. E ao
contrário dos ports de inicialização, o adaptador é escolhido **por
requisição, pelo recurso** — vários ativos ao mesmo tempo, como uma conta tem
um projeto no Jira e outro no ClickUp. O roteador escolhe a classe; o
adaptador ativo resolve o nome do modelo; a política de custo vale para todo
fornecedor sem conhecer nenhum.

**O runtime roda no core.** A credencial nunca cruza uma fronteira de rede:
lida do vault, usada no mesmo processo. E o que a implementação revelou pesou
tanto quanto a segurança: o runtime já era quase inteiramente a orquestração
do core. As seis coisas que ele faz num turno — montar contexto, rotear o
modelo, registrar consumo, postar uma mensagem, publicar um achado, respeitar
o orçamento — são todas operações do core, feitas de fora por gRPC. Dois
módulos existiam só por causa da fronteira e desapareceram: um contornando um
vault inacessível, outro refazendo o caminho de volta do nome de um modelo
para a sua classe.

## O que custou

A semântica de cache de prefixo não é a mesma entre fornecedores, e a
economia depende dela — a divergência mais cara, documentada no port. Os
formatos de chamada de ferramenta, os eventos de streaming e os motivos de
parada também divergem; o que todos não conseguem cumprir fica fora. Dois
adaptadores e uma suíte de contrato desde o início, o custo que esta
plataforma já tinha pago três vezes. Os adaptadores de provedor foram
reescritos em Go — cerca de seiscentas linhas; o desenho sobreviveu, a
linguagem mudou. E o core passou a fazer chamadas externas longas, então
orçamento de conexões e timeouts viraram preocupação dele.

## Desde então

A borda ganhou uma invariante mais forte que "sem banco": **sem segredo.** É
hoje uma das cinco paredes sobre as quais os agentes de código são instruídos
antes de tocar em qualquer coisa, e o segundo fator
([ADR-0020](../second-factor-in-the-core/)) a usou dois dias depois como a
razão de a semente TOTP morar no core.
