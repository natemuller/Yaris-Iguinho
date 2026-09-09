# Registro de decisões

Cada decisão relevante do projeto vira **um arquivo** nesta pasta. O objetivo é
simples: daqui a três meses, quando alguém — inclusive você — perguntar *"por que
foi feito assim?"*, a resposta está escrita, com o contexto da época e as
alternativas que foram descartadas.

O formato é o de um **ADR** (*Architecture Decision Record*), adaptado para
registrar também **qual agente** tomou a decisão.

---

## Quando escrever um ADR

Escreva quando a decisão:

- fecha uma escolha entre alternativas viáveis (banco, provedor, biblioteca, formato de mensagem);
- define um contrato entre partes do sistema;
- descarta um caminho por um motivo que alguém tentaria de novo mais tarde;
- altera o escopo da v1.

Não escreva para nome de variável, detalhe de formatação ou qualquer coisa que se
desfaça em cinco minutos.

Na dúvida, escreva. Um ADR curto custa dois minutos; uma decisão refeita custa um dia.

---

## Como escrever

1. Copie [`0000-template.md`](0000-template.md)
2. Nomeie como `NNNN-titulo-em-kebab-case.md`, com `NNNN` sendo o próximo número
   livre em sequência (`0002`, `0003`, ...). Números **não são reaproveitados**,
   nem quando um ADR é substituído
3. Preencha o cabeçalho e as seções
4. Adicione a linha correspondente no [índice](#índice) abaixo

Atalho: o comando [`/decisao`](../../.claude/commands/decisao.md) faz esses quatro
passos.

---

## Regras

**Um ADR é imutável depois de aceito.** Mudou de ideia? Escreva um novo ADR que
substitui o anterior, e marque o antigo como `Substituída por NNNN`. O histórico
das decisões erradas é parte do valor — ele impede que o mesmo erro volte.

**Alternativas descartadas são obrigatórias.** Um ADR que só diz o que foi
escolhido perde metade da utilidade. O que evita retrabalho é saber o que *já foi
considerado e por que não serviu*.

**Escreva o contexto da época, não o de hoje.** "Escolhemos SQLite porque é um
usuário só" envelhece bem. "Escolhemos SQLite" não.

**Nomeie o agente.** Cada ADR diz qual agente o produziu. Isso mostra se uma
decisão está sendo tomada no domínio certo — decisões que atravessam domínios são
do `arquiteto`.

---

## Status possíveis

| Status | Significa |
|---|---|
| `Proposta` | Escrita, aguardando validação do usuário |
| `Aceita` | Em vigor |
| `Substituída por NNNN` | Não vale mais; o ADR indicado a substituiu |
| `Descartada` | Foi considerada e recusada; fica registrada para não voltar |

---

## Índice

| # | Decisão | Agente | Status | Data |
|---|---|---|---|---|
| [0001](0001-registro-de-decisoes-em-adr.md) | Registrar decisões como ADRs numerados nesta pasta | `arquiteto` | Aceita | 2026-09-09 |
| [0002](0002-v1-sem-feedback-recomendacao-por-ia.md) | Tirar feedback e aprendizado da v1; recomendar pela opinião da IA | `produto` | Aceita | 2026-09-09 |
| [0003](0003-gemini-free-tier-como-provedor-de-ia.md) | Usar o free tier do Gemini como provedor de IA da v1 | `arquiteto` | Aceita | 2026-09-09 |
| [0004](0004-mensagem-em-dois-templates-com-imagem-composta.md) | Entregar a mensagem em dois templates, com os cardápios numa imagem composta | `integracao-whatsapp` | Aceita | 2026-09-09 |
| [0005](0005-grade-2x2-fixa-com-quatro-restaurantes.md) | Fixar a imagem em uma grade 2×2 de quatro restaurantes, com tile de indisponível | `integracao-whatsapp` | Aceita | 2026-09-09 |
| [0006](0006-recomendar-por-destaque-relativo-do-dia.md) | Recomendar pelo destaque relativo do dia, considerando os destaques recentes | `recomendador` | Aceita | 2026-09-09 |
