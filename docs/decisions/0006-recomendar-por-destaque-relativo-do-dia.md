# 0006 — Recomendar pelo destaque relativo do dia, considerando os destaques recentes

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `recomendador` |
| **Relacionadas** | 0002, 0003 |

## Contexto

O [ADR 0002](0002-v1-sem-feedback-recomendacao-por-ia.md) tirou o score e o
feedback da v1: a escolha passou a ser a opinião de uma IA que lê todos os
cardápios do dia. Ficou aberto o risco R6 — sem enxergar o passado, a IA poderia
sugerir sempre o mesmo lugar, recriando o problema que motivou o projeto.

A saída óbvia era passar as últimas sugestões e pedir uma **penalidade por
repetição**: quanto mais recente a última vez que o restaurante foi sugerido, menor
a chance de ganhar de novo.

O usuário recusou, e o motivo expôs um defeito real: **penalidade por repetição é
uma regra de contabilidade se sobrepondo à comida.** Se o Restaurante X tem o
melhor prato de hoje, o motivo para não recomendá-lo não pode ser "ele ganhou
ontem" — isso faz o usuário almoçar pior por causa de bookkeeping do sistema.

A proposta dele foi julgar pelo **destaque relativo do dia**: o que esta opção tem
que as outras de hoje não têm. Sushi sobressai num dia em que os outros têm PF;
espetinho sobressai em outro. A rotação sairia de graça, porque os cardápios já
rodam.

Mas essa versão sozinha tem um furo, apontado pelo próprio usuário: **cardápio
fixo semanal.** Um restaurante que serve lasanha toda terça vai se destacar contra
três PFs *toda terça, para sempre*. A comparação do dia, isolada, não enxerga isso.

E ele formulou a correção de um jeito que muda o alvo: **a repetição que incomoda
é a da comida, não a do restaurante.** "Lasanha foi destaque nas últimas duas
semanas" é o problema real. Uma penalidade por restaurante não pegaria dois
estabelecimentos diferentes servindo lasanha em semanas alternadas — e o usuário
continuaria comendo lasanha.

## Decisão

O critério é o **destaque relativo entre os cardápios do dia, temperado pelos
destaques recentes**.

O prompt recebe duas coisas:

1. Os cardápios de hoje
2. As sugestões das **últimas duas semanas**, cada uma com o restaurante **e o
   motivo pelo qual foi escolhida** — não apenas o nome

O modelo raciocina sobre novidade real: *"lasanha seria um bom destaque contra os
PFs de hoje, mas já foi o destaque duas vezes nas últimas semanas — vale procurar
algo mais diferente."*

Três regras que delimitam isso:

- **O histórico é ponderação, nunca veto.** São 4 restaurantes e 5 dias por
  semana: em duas semanas quase tudo já apareceu. O modelo sempre escolhe alguém;
  "tudo repetido" não é resposta válida.
- **A comparação é entre as opções de hoje.** "Destaque" não significa variedade
  interna do cardápio de um restaurante — um lugar com oito pratos genéricos não
  ganha de um com um prato que se destaca.
- **Sem destaque real, justificativa honestamente fraca.** Quando os quatro
  oferecem essencialmente a mesma coisa, "hoje está tudo parecido" é a resposta
  correta. Inventar diferença para soar convincente é falha, não recurso.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Penalidade mecânica por dias desde a última sugestão do restaurante | Sobrepõe regra administrativa ao mérito da comida. Faz o usuário almoçar pior porque o sistema quer variar |
| Nenhum histórico, só a comparação do dia | Quebra no cardápio fixo semanal: lasanha vence toda terça, indefinidamente. Foi a versão inicial desta decisão, corrigida antes de valer |
| Histórico só com os nomes dos restaurantes, sem o motivo | Erra o alvo. O que cansa é comer lasanha de novo, não ir ao mesmo endereço. Dois restaurantes alternando o mesmo prato passariam batido |
| Avaliar cada cardápio isoladamente e pegar a maior nota | Perde o que torna a escolha útil: sushi só "sobressai" em comparação. Isolado, é só mais um prato bom |
| Rodízio entre os restaurantes, ignorando o cardápio | Garante variedade e destrói o propósito — a sugestão deixa de ter relação com a comida |

## Consequências

**Ganhamos:**

- Cobre o caso do **cardápio fixo semanal**, que era o furo da versão anterior
- A repetição é julgada **pela comida**, que é o que realmente enjoa
- A recomendação continua ancorada no cardápio, não numa regra interna — a
  justificativa sempre aponta para algo real
- O modelo consegue explicar a própria escolha de um jeito que faz sentido para o
  usuário: "seria a lasanha, mas ela já foi destaque duas vezes"

**Abrimos mão de / passamos a conviver com:**

- **Volta a dependência de estado.** A recomendação de hoje depende do histórico
  ter sido gravado corretamente. Se a gravação falhar, a qualidade cai em silêncio
- O prompt cresce e precisa ser montado a partir do banco, não só dos cardápios
- **O julgamento é difuso.** "Recente demais" fica a cargo do modelo, então a
  mesma entrada pode gerar decisões diferentes em execuções distintas
- **Risco de distinção fabricada.** O prompt pede diferenciação, e modelo cobrado
  por diferença tende a inventá-la quando não existe
- **Risco de saturação.** Com 4 restaurantes e 10 dias úteis de janela, quase tudo
  terá aparecido. Daí a regra de que histórico nunca veta

**Sobre a regra do [ADR 0003](0003-gemini-free-tier-como-provedor-de-ia.md):** o
histórico enviado é a lista do que **o sistema sugeriu**, não do que o usuário
comeu — sem feedback, essas coisas não são a mesma. São nomes de estabelecimento e
pratos, sem identificador de pessoa. Não viola a regra de "nada de pessoal no
prompt", mas é o item que mais se aproxima da fronteira. Se um dia carregar
comportamento real do usuário, revisitar.

**Precisa ser revisitado quando:**

- O histórico de 30 dias mostrar que a variedade não melhorou na prática
- O feedback entrar (primeiro item do backlog) — ele muda a base da conversa

## A verificar

- [ ] Caso de teste com **quatro cardápios quase idênticos**: a justificativa sai
      honestamente fraca ou o modelo inventa diferença?
- [ ] Caso de teste com **histórico saturado** (tudo já foi destaque nas duas
      semanas): o modelo ainda escolhe alguém, ou trava?
- [ ] Caso de teste do **cardápio fixo semanal**: lasanha toda terça deixa de
      vencer depois de duas ou três aparições?
- [ ] Se duas semanas é a janela certa, ou se com 4 restaurantes ela satura rápido
      demais
