# 0001 — Registrar decisões como ADRs numerados nesta pasta

| | |
|---|---|
| **Status** | Aceita |
| **Data** | 2026-09-09 |
| **Agente** | `arquiteto` |
| **Relacionadas** | — |

## Contexto

O projeto nasce dividido em domínios com agentes especializados: coleta de
stories, extração por IA, recomendação, WhatsApp, interface. Cada um vai fechar
escolhas dentro do próprio escopo, em momentos diferentes, e várias dessas
escolhas viram restrição para os outros.

Duas características do projeto tornam o registro necessário desde o começo:

1. **A arquitetura ainda não foi decidida.** As decisões vão se acumulando até
   consolidarem a seção "Repositório e arquitetura" do README. Sem registro
   incremental, essa consolidação vira arqueologia.
2. **Boa parte das escolhas é sobre terreno instável** — como obter stories do
   Instagram, como enviar mensagem iniciada pelo negócio no WhatsApp. São áreas
   onde caminhos aparentemente óbvios já foram testados e não funcionam. Sem o
   registro do que foi descartado *e por quê*, os mesmos becos sem saída serão
   percorridos de novo.

## Decisão

Toda decisão relevante é registrada como um arquivo Markdown numerado em
`docs/decisions/`, seguindo o template `0000-template.md`. O cabeçalho identifica
o **agente** que tomou a decisão. ADRs aceitos são imutáveis: mudança de rumo
gera um novo ADR que substitui o anterior.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Uma subpasta por agente | Espalha uma linha do tempo que é única. Decisões que atravessam domínios não teriam lugar óbvio, e a ordem cronológica — que é o que revela a evolução do raciocínio — se perderia. O campo `Agente` no cabeçalho dá a mesma informação sem fragmentar |
| Um único arquivo `DECISOES.md` acumulativo | Vira conflito de merge constante e um arquivo que ninguém lê inteiro. Também não permite referenciar uma decisão por identificador estável |
| Registrar só nas mensagens de commit | Some no histórico e não sobrevive a squash ou rebase. Não comporta as alternativas descartadas, que são a parte mais útil |
| Não registrar; a decisão está no código | Só funciona para o que virou código. Não cobre o caminho descartado, que é justamente o que se tenta de novo |

## Consequências

**Ganhamos:**

- Uma resposta escrita para "por que foi feito assim?", com o contexto da época
- Rastro dos caminhos descartados, evitando retrabalho nas áreas de maior risco
- Matéria-prima pronta para consolidar a seção de arquitetura do README
- Visibilidade de qual agente está decidindo o quê, revelando decisões tomadas no
  domínio errado

**Abrimos mão de / passamos a conviver com:**

- Alguns minutos de escrita por decisão
- O risco de ADRs desatualizados, mitigado pela regra da imutabilidade e do
  status `Substituída por`
- A necessidade de manter o índice em `docs/decisions/README.md` sincronizado

**Precisa ser revisitado quando:**

- O volume passar de algumas dezenas de ADRs e o índice manual virar gargalo
- O projeto ganhar mais de um contribuidor humano, tornando conflitos de
  numeração frequentes
