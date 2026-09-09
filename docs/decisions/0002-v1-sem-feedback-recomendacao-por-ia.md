# 0002 — Tirar feedback e aprendizado da v1; recomendar pela opinião da IA

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `produto` |
| **Relacionadas** | 0001 |

## Contexto

O escopo original da v1 previa um ciclo fechado: o sistema sugeria um
restaurante, perguntava "Você gostou da sugestão?", recebia Sim ou Não e usava
essa resposta num score heurístico para calibrar as sugestões seguintes.

Esse ciclo custava caro para a primeira versão:

- **Exigia integração de entrada no WhatsApp** — webhook, URL pública,
  idempotência de eventos reenviados pela Meta, interpretação de resposta livre.
  Metade do trabalho da integração existia só para receber uma palavra por dia.
- **Exigia um algoritmo de score** com pesos, penalidade de repetição e
  tratamento de empate, além dos testes correspondentes.
- **Não teria dados para funcionar.** O sistema começa com zero feedbacks e ganha
  no máximo um por dia. Nas primeiras semanas — justamente o período de validação
  da v1 — o score estaria calibrado com ruído, e não haveria como distinguir uma
  boa recomendação de uma ruim.

Ao mesmo tempo, os cardápios já chegam estruturados na etapa de extração. Um
modelo que recebe todos eles de uma vez consegue opinar qual é a melhor opção do
dia sem nenhum cálculo do nosso lado.

## Decisão

O feedback e o aprendizado **saem da v1**. A recomendação passa a ser uma
**segunda chamada de IA**, separada da extração: ela recebe todos os cardápios do
dia já em texto e devolve, em JSON validado por schema, o restaurante escolhido, o
motivo e a frase de sugestão pronta.

A mensagem do WhatsApp vira **mão única**: o sistema envia e não recebe nada de
volta.

Feedback e aprendizado passam a ser os dois primeiros itens do backlog pós-v1.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Manter o feedback como estava | Dobra o trabalho da integração com o WhatsApp para alimentar um algoritmo que, nas primeiras semanas, não teria dados suficientes para melhorar nada |
| Manter o score heurístico, mas sem feedback | Sobrariam apenas repetição e variedade como fatores. É código, teste e manutenção para uma regra que a IA aplica de graça ao ler os cardápios — e sem a explicabilidade em linguagem natural que ela entrega junto |
| Mandar todas as imagens numa única chamada, extraindo e recomendando de uma vez | Menos chamadas, mas impede cachear a leitura de um story já processado, mistura duas responsabilidades e torna impossível saber se um erro veio da leitura da imagem ou do julgamento |
| Coletar feedback por fora do WhatsApp (na interface web) | Evitaria o webhook, mas ninguém abre uma interface web para responder "sim" depois do almoço. Feedback que não é coletado no momento não é coletado |

## Consequências

**Ganhamos:**

- A integração com o WhatsApp fica só de saída: **sem webhook, sem URL pública,
  sem tratamento de mensagem de entrada**
- O deploy fica mais simples e com mais opções — basta execução agendada
  confiável e armazenamento persistente
- Nenhum algoritmo de score para escrever, ajustar e testar
- A justificativa da escolha sai em linguagem natural, pronta para a mensagem, sem
  camada de tradução entre score e frase
- Menos superfície para a v1 dar errado, que era o objetivo do escopo enxuto

**Abrimos mão de / passamos a conviver com:**

- **O sistema não tem como saber se está acertando.** Sem sinal de retorno, a
  única verificação é o usuário conferir o histórico na mão (risco R5 do README)
- **A IA pode repetir o mesmo restaurante vários dias seguidos**, já que não vê o
  histórico — o que recria justamente o problema que motivou o projeto
  (risco R6). Se acontecer, passar as últimas sugestões no prompt resolve barato
- Uma chamada de IA por dia a mais, com custo desprezível
- Uma nova superfície de erro: o modelo pode escolher um restaurante que não
  postou cardápio hoje. A escolha precisa ser **verificada contra a lista** antes
  de virar mensagem

**Precisa ser revisitado quando:**

- A repetição de restaurantes incomodar na prática — primeiro sinal, e o mais
  provável de aparecer
- As sugestões começarem a errar de forma perceptível e não houver como corrigir
  sem um sinal de retorno
- A v1 estiver estável e rodando: aí o feedback entra como planejado, com o custo
  do webhook justificado por dados que já existem
