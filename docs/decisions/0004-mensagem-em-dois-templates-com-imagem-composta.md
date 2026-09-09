# 0004 — Entregar a mensagem em dois templates, com os cardápios numa imagem composta

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `integracao-whatsapp` |
| **Relacionadas** | 0002; **refinada pela [0005](0005-grade-2x2-fixa-com-quatro-restaurantes.md)**, que fixa o layout da imagem e revê a regra de quem entra nela |

## Contexto

Duas restrições do usuário fecharam o cerco sobre o formato da mensagem: **custo
zero** e **sem número de WhatsApp novo**. O caminho oficial (Cloud API com o número
de teste gratuito da Meta) atende às duas, mas impõe que toda mensagem iniciada
pelo sistema seja um **template aprovado** — e template tem duas limitações duras,
ambas confirmadas:

1. **Parâmetro de template não aceita quebra de linha, tab nem mais de 4 espaços
   seguidos**, com teto de 1024 caracteres. Uma lista de cardápios formatada não
   cabe numa variável.
2. **Mensagem interativa — lista clicável, botão com texto dinâmico — só pode ser
   enviada dentro de uma janela de 24h aberta pelo usuário.** Não existe lista
   clicável chegando sozinha às 11h.

A segunda limitação matou o desenho inicialmente preferido pelo usuário: uma lista
com um item por restaurante, onde tocar num item traria a foto daquele cardápio.
Esse desenho é possível, mas exige que o usuário toque em algo antes, o que gera
mensagem de entrada — e portanto **webhook e URL pública**, justamente o que o
ADR 0002 removeu.

O destravamento veio de uma observação do usuário: **os restaurantes são
estáticos.** Aparecem na mesma ordem todo dia; a lista só muda quando ele cadastra
ou remove um. E, mais importante, **os nomes podem viver dentro da imagem**, não no
texto do template.

## Decisão

A entrega diária são **duas mensagens de template**, sem nenhuma interação:

**Mensagem 1** — template com cabeçalho de mídia:

> Bom dia, {{nome}}. Vamos almoçar o que hoje? Confira os cardápios disponíveis!

acompanhada de **uma imagem composta**, montada por nós, com os stories dos
cardápios do dia lado a lado e o nome do restaurante sobre cada um.

**Mensagem 2** — template com uma variável de linha única:

> Sugestão do dia: {{1}}

onde `{{1}}` é a frase que o `recomendador` produziu, mais os restaurantes que não
postaram até o horário.

Só entram na imagem os restaurantes **que postaram**. Quem não postou é nomeado em
texto na mensagem 2.

Não há webhook, não há URL pública, não há mensagem de entrada.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Mensagem única em texto, cardápios em linha única separados por bullets | Cabe na restrição, mas entrega o pior formato possível: sem foto, sem quebra de linha, e o usuário não vê o cardápio original — só a transcrição da IA, sem poder conferir |
| Lista interativa clicável, uma foto por toque | É o desenho preferido do usuário e continua sendo o melhor. Mas exige janela de 24h aberta por ele, logo webhook e URL pública. Rodando na máquina local, um clique com o notebook dormindo falha em silêncio. Adiado para depois da v1 |
| Uma mensagem só, com a sugestão na legenda da imagem | Legenda longa no WhatsApp trunca com "Ler mais". A sugestão é a parte mais importante da mensagem — não pode estar escondida atrás de um toque |
| Nomes dos restaurantes no texto fixo do template | Possível, já que a lista é estática. Mas cada cadastro ou remoção exigiria **nova aprovação de template** pela Meta. Colocar os nomes na imagem elimina isso |
| Uma mensagem de template por restaurante, cada uma com sua foto | Resolve o formato, mas são 3 a 10 notificações às 11h. Vira spam |

## Consequências

**Ganhamos:**

- O usuário vê **a foto original do cardápio**, não só a transcrição da IA — o que
  também compensa parcialmente o risco R5 (extração errada) sem precisar abrir a
  interface
- Formato livre dentro da imagem: nome, ordem, tag de indisponível, o que for
- Adicionar ou remover restaurante **não exige nova aprovação de template**
- Sem webhook: a arquitetura de mão única do ADR 0002 permanece intacta
- Dois templates genéricos, aprovados uma vez e estáveis para sempre

**Abrimos mão de / passamos a conviver com:**

- **Montar a imagem é trabalho novo** que não existia no plano: composição,
  redimensionamento, sobreposição do nome. É a parte mais cara desta decisão
- **A legibilidade não está garantida.** Story é 1080×1920; vários lado a lado
  encolhem na bolha do chat, e o WhatsApp ainda recomprime. Com 3 restaurantes é
  administrável; com 10 numa fila única, não. Ver item de verificação abaixo
- Duas mensagens por dia em vez de uma
- A imagem depende de guardarmos o story original, o que amarra esta decisão à
  regra de retenção de 30 dias

**Precisa ser revisitado quando:**

- O usuário quiser a lista clicável de verdade — momento em que o webhook volta e,
  junto com ele, o feedback fica quase de graça (ver ADR 0002). Os dois devem
  entrar juntos, dividindo o custo da mesma infraestrutura
- O número de restaurantes crescer a ponto de a imagem composta ficar ilegível

## A verificar

- [ ] **Legibilidade da imagem composta, com stories reais, olhando no celular.**
      Grade de 2 colunas é o palpite atual, contra a fila única. Isso não se
      resolve no papel: monte e olhe
- [ ] Limites do número de teste gratuito da Meta, no painel — destinatários e
      volume
- [ ] Se as duas mensagens diárias contam como duas cobranças no modelo atual da
      Meta (irrelevante se o número de teste for gratuito, decisivo se não for)
