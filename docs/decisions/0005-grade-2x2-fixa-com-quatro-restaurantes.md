# 0005 — Fixar a imagem em uma grade 2×2 de quatro restaurantes, com tile de indisponível

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `integracao-whatsapp` |
| **Relacionadas** | 0004 |

## Contexto

O [ADR 0004](0004-mensagem-em-dois-templates-com-imagem-composta.md) decidiu
entregar os cardápios numa imagem composta, mas deixou dois pontos em aberto:

- o **layout** da imagem, com "grade de 2 colunas" registrado apenas como palpite
  a ser verificado com material real;
- ele também determinou que **só entrariam na imagem os restaurantes que
  postaram**, com os ausentes nomeados em texto na segunda mensagem.

O primeiro ponto era o maior risco da decisão (R12 no README): story é uma imagem
alta e estreita, e vários lado a lado encolhem na bolha do chat a ponto de o
cardápio ficar ilegível. O segundo ponto tinha um efeito colateral que só ficou
claro depois: excluir da imagem quem não postou faz as posições dos restaurantes
mudarem de um dia para o outro.

O usuário definiu então que a v1 acompanha **quatro restaurantes** — os mesmos que
usará para teste — e propôs a grade 2×2 com um tile de "Cardápio indisponível" no
lugar de quem não postou.

## Decisão

A imagem composta é uma **grade 2×2 fixa, com quatro tiles, sempre**:

```
Story 1 | Story 2
-----------------
Story 3 | Story 4
```

Cada tile corresponde a um restaurante cadastrado, em **posição estável**: o mesmo
restaurante ocupa o mesmo lugar todos os dias. A posição só muda quando o usuário
altera o cadastro.

Restaurante que não postou cardápio no dia **mantém seu tile**, preenchido com um
fundo neutro e o texto **"Cardápio indisponível"**. A grade nunca tem buraco.

Como consequência, a **v1 aceita no máximo 4 perfis ativos**, e o cadastro impede
o quinto. A segunda mensagem deixa de nomear quem não postou — a imagem já diz.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Só os que postaram entram na imagem (o que o ADR 0004 previa) | A grade encolhe e cresce conforme o dia, e as posições mudam. O usuário perde a referência espacial e precisa ler os nomes toda vez |
| Grade adaptativa (2×2, 2×3, 3×3 conforme a quantidade) | Resolveria o teto de 4, mas cada configuração tem uma proporção diferente, e só a 2×2 preserva o 9:16. Complexidade real para um ganho que a v1 não precisa |
| Três stories em fila única | Sai de proporção — vira uma faixa larga e baixa que o WhatsApp encolhe muito. Foi o que motivou o risco R12 |
| Tile ausente listado só em texto na mensagem 2 | Separa a informação em dois lugares. Ver "faltou o do Y" olhando o próprio grid é mais imediato do que ler uma linha embaixo |

## Consequências

**Ganhamos:**

- **A proporção certa de graça.** Quatro tiles de 1080×1920 numa grade 2×2 dão
  2160×3840, que é exatamente 9:16 — a mesma proporção de um story sozinho. A
  imagem ocupa a área máxima da bolha do chat, sem tarja nem corte
- **Referência espacial estável.** O restaurante está sempre no mesmo canto, o que
  permite bater o olho sem ler nome. Numa mensagem lida com fome, isso é o
  principal ganho
- Layout fixo é muito mais simples de montar do que um adaptativo
- A ausência de cardápio fica visível no mesmo lugar que a presença, sem exigir
  que o usuário cruze imagem com texto

**Abrimos mão de / passamos a conviver com:**

- **Teto de 4 perfis ativos na v1.** Cadastrar um quinto quebra o layout, então o
  cadastro precisa impedir. Isso contradiz o parâmetro anterior de "até 10
  restaurantes", que passa a valer como horizonte pós-v1, não como v1
- Cada tile ocupa um quarto da imagem: mesmo com a proporção correta, ler o
  cardápio provavelmente exige zoom. Aceitável, mas ainda não verificado
- Um dia em que ninguém postou gera uma imagem com quatro tiles vazios — inútil.
  Nesse caso, não mande imagem nenhuma

**Precisa ser revisitado quando:**

- O usuário quiser acompanhar mais de 4 restaurantes — momento em que a estratégia
  de grade precisa ser decidida de verdade
- O teste com stories reais mostrar que um quarto da imagem é pouco para ler o
  cardápio mesmo com zoom

## A verificar

- [ ] **Legibilidade de um tile com story real, no celular, com e sem zoom.**
      Continua sendo o maior risco (R12). A grade 2×2 melhora as chances, mas não
      resolve por decreto
- [ ] Onde colocar o nome do restaurante: faixa acima do tile (não cobre nada, mas
      aumenta a altura e sai do 9:16) ou sobreposta no topo do story (preserva a
      proporção, mas pode tapar o título do cardápio). Decidir olhando material
      real
- [ ] Quanto o WhatsApp recomprime uma imagem 2160×3840, e se o detalhe sobrevive
