---
description: Registra uma decisão do projeto como ADR numerado em docs/decisions/
argument-hint: [descrição curta da decisão]
---

Registre a decisão a seguir como um ADR em `docs/decisions/`.

**Decisão:** $ARGUMENTS

Passos:

1. Leia `docs/decisions/README.md` e `docs/decisions/0000-template.md`.
2. Liste `docs/decisions/` e determine o **próximo número livre** em sequência.
   Números nunca são reaproveitados, nem os de ADRs substituídos.
3. Crie `docs/decisions/NNNN-titulo-em-kebab-case.md` a partir do template,
   preenchendo todas as seções. O campo **Agente** recebe o agente responsável
   pelo domínio da decisão; se ela atravessa domínios, é `arquiteto`.
4. Adicione a linha correspondente ao índice em `docs/decisions/README.md`.

Ao preencher:

- **Contexto** é para quem não estava presente. Descreva a situação da época, o
  que se sabia e o que não se sabia.
- **Alternativas consideradas** é obrigatório e é a parte mais valiosa do ADR.
  Diga por que cada caminho foi descartado — é isso que impede alguém de tentar
  de novo daqui a três meses.
- **Consequências** inclui o que se perde e o que passa a ser conviver, não só o
  que se ganha.

Se faltar informação para preencher contexto ou alternativas, **pergunte ao
usuário** em vez de inventar. Um ADR com contexto fabricado é pior do que
nenhum ADR.

Se a decisão ainda não foi validada pelo usuário, marque o status como
`Proposta`, não `Aceita`.
