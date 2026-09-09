# 0003 — Usar o free tier do Gemini como provedor de IA da v1

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `arquiteto` |
| **Relacionadas** | 0002 |

## Contexto

O usuário definiu **orçamento zero** para o projeto. Isso criou um conflito
(registrado como risco R8 no README): o sistema faz cerca de 11 chamadas de IA por
dia útil — dez imagens de story para extração e uma chamada de texto para a
recomendação —, e modelo em nuvem cobra por uso. Eram poucos dólares por mês, mas
não zero.

As opções conhecidas eram ruins nos dois extremos: aceitar um custo recorrente que
o usuário havia descartado, ou rodar um modelo multimodal local via Ollama, que é
gratuito mas lê sensivelmente pior fotos de quadro branco com letra manuscrita em
português — justamente a capacidade que sustenta o produto.

O free tier do Gemini resolve o conflito. Os limites reportados para os modelos
Flash são de cerca de 15 requisições por minuto e **1.500 por dia**, contra as ~11
diárias que o sistema precisa. A folga é de mais de cem vezes, o que cobre com
sobra reprocessamento, testes de prompt e execuções repetidas.

O que o free tier custa não é dinheiro, é privacidade. Nos termos da API, no tier
gratuito o Google usa o conteúdo enviado para desenvolver seus produtos e
revisores humanos podem ler entradas e saídas — o que não acontece no tier pago.

## Decisão

A v1 usa o **free tier do Gemini (linha Flash)** como provedor das duas chamadas
de IA: extração do cardápio a partir da imagem e recomendação a partir dos
cardápios do dia.

Fica estabelecida uma regra que decorre disso: **nada de pessoal vai no prompt.**
Só entram imagens de stories públicos de restaurantes, texto de cardápio e nomes
de estabelecimento. Número de telefone, nome do usuário e preferências pessoais
estão proibidos de entrar em qualquer chamada enquanto o free tier for o provedor.

O modelo continua atrás de uma interface própria, como já previsto para o provedor
de stories.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Modelo em nuvem pago | Poucos dólares por mês, mas o usuário descartou custo recorrente. Perdeu a razão de ser quando o free tier mostrou folga suficiente |
| Modelo multimodal local via Ollama | Gratuito e sem questão de privacidade, mas a qualidade de leitura em foto de quadro branco com letra manuscrita cai bastante — e essa leitura *é* o produto. Fica como plano B se o free tier mudar |
| Free tier pago só para a extração, local para a recomendação | Complexidade de dois provedores para economizar uma chamada de texto por dia. Não compensa |
| Tier pago do Gemini | Resolveria a questão de privacidade, mas reintroduz o custo que a decisão inteira existe para evitar. Só faz sentido se o projeto passar a lidar com dado pessoal |

## Consequências

**Ganhamos:**

- **O risco R8 deixa de existir:** custo zero de verdade, com qualidade de modelo
  de fronteira em vez de modelo local
- Folga de mais de cem vezes sobre o uso previsto — testes de prompt e
  reprocessamento não são um problema de orçamento
- Sem cartão de crédito e sem prazo de expiração
- A suíte de regressão de prompt, que é o único sinal de qualidade da v1 (ver
  ADR 0002), pode rodar quantas vezes for preciso

**Abrimos mão de / passamos a conviver com:**

- **O conteúdo enviado alimenta o treinamento do Google, e revisores humanos podem
  lê-lo.** Aceitável aqui porque o conteúdo é público (stories de restaurantes),
  mas obriga a regra do "nada pessoal no prompt"
- **Isso restringe o backlog.** O item "perfil de preferências" — mandar "gosto de
  peixe, evito fígado" no prompt — passa a ser dado pessoal indo para um pipeline
  de treinamento. Ele exige revisitar esta decisão antes de ser implementado
- Limites de free tier mudam sem aviso e sem garantia contratual. Modelos já foram
  removidos do tier gratuito antes
- Dependência de um segundo fornecedor com termos próprios

**Precisa ser revisitado quando:**

- Algum dado pessoal do usuário precisar entrar no prompt — preferências,
  histórico identificável, qualquer coisa além de cardápio público
- Os limites do free tier caírem abaixo do uso real, ou a linha Flash sair dele
- A qualidade de leitura se mostrar insuficiente na suíte de regressão

## A verificar

Nada disso bloqueia a decisão, mas precisa ser confirmado antes de a integração
ser construída:

- [ ] Limites atuais e reais do free tier, no painel do Google AI Studio — a
      documentação oficial deixou de publicar a tabela, e os números acima vêm de
      fontes secundárias
- [ ] Disponibilidade do free tier na região do usuário (Brasil)
