# CLAUDE.md

Contexto e convenções para agentes de IA que trabalham neste repositório.
O **o quê** e o **porquê** do produto estão no [`README.md`](README.md); aqui está
o **como se trabalha**.

---

## O projeto em três frases

Yaris-Iguinho recebe no WhatsApp, todo dia útil num horário configurado, os
cardápios do dia de restaurantes cujos `@` do Instagram foram cadastrados pelo
usuário, junto com uma sugestão de onde almoçar — a opinião de uma IA que leu
todos os cardápios. É um sistema **de um usuário só** (o dono do repositório) e
**de mão única**: envia e não recebe nada de volta.

---

## Estado atual

| | |
|---|---|
| **Fase** | Definição do macro do negócio |
| **Código** | Não existe ainda |
| **Arquitetura** | **Não decidida** — ver a seção "Repositório e arquitetura" do README |
| **Stack** | Apenas sugerida no README; nada escolhido em definitivo |

**Consequência prática:** não invente estrutura de pastas de código, não escolha
framework e não crie `package.json` por conta própria. Enquanto a arquitetura não
estiver decidida e registrada em `docs/decisions/`, o trabalho é de definição, não
de implementação.

---

## Idioma

- **Documentação, ADRs, commits, comunicação com o usuário:** português do Brasil
- **Código — identificadores, tipos, nomes de arquivo, comentários:** inglês
- **Textos que o usuário final lê** (mensagens do WhatsApp, rótulos da interface):
  português do Brasil

Motivo: o vocabulário do domínio é português, mas código misturando os dois
idiomas envelhece mal. O [glossário do README](README.md#glossário-do-domínio) dá
o par oficial de cada termo — use exatamente ele. `Profile`, não `InstagramUser`.
`DailyMenu`, não `Menu`.

---

## Registro de decisões

**Toda decisão relevante vira um arquivo em [`docs/decisions/`](docs/decisions/).**

Uma decisão é relevante quando:

- fecha uma escolha entre alternativas viáveis (banco, provedor, biblioteca, formato);
- define um contrato entre partes do sistema;
- descarta um caminho por um motivo que alguém, daqui a três meses, tentaria de novo;
- altera o escopo da v1.

Não é relevante: nome de variável, detalhe de formatação, decisão que se desfaz em
cinco minutos.

O processo, o template e a numeração estão em
[`docs/decisions/README.md`](docs/decisions/README.md). Cada ADR nomeia **qual
agente** a tomou.

> Se você fechou uma decisão e não escreveu o ADR, a decisão não aconteceu — ela
> vai ser refeita, provavelmente de outro jeito.

---

## Agentes

Os agentes vivem em [`.claude/agents/`](.claude/agents/), um por domínio do
projeto. Cada um tem escopo declarado e sabe quando devolver o trabalho.

| Agente | Domínio |
|---|---|
| `produto` | Escopo, user stories, backlog, critérios de aceite |
| `arquiteto` | Stack, estrutura, contratos entre módulos, modelo de dados |
| `coletor-instagram` | Obtenção de stories dos perfis |
| `extrator-cardapio` | Ler a imagem do story e produzir cardápio estruturado |
| `recomendador` | A chamada de IA que lê todos os cardápios e diz onde almoçar |
| `integracao-whatsapp` | Envio da mensagem diária (só de saída) |
| `interface-web` | Telas de cadastro, configuração e histórico |
| `qa` | Testes, casos de borda, verificação dos critérios de aceite |
| `professor` | Ensina o usuário a chegar na solução; não implementa nem é dono de domínio |

Regra geral: **o agente decide dentro do próprio domínio e registra em ADR.** Se a
decisão atravessa domínios (um contrato, o modelo de dados, a stack), ela é do
`arquiteto`.

O `professor` é a exceção ao "um por domínio": ele não decide nada do produto,
ensina o usuário a extrair a solução e aciona os outros agentes quando precisa da
análise de domínio para montar a explicação. Só escreve código sob pedido
explícito e com confirmação.

---

## Convenções de código

Valem a partir do momento em que existir código.

### TypeScript

- `strict: true`, sem exceção
- **`any` é proibido.** Se o tipo é desconhecido, é `unknown` e passa por validação
- Toda **borda externa** — resposta de provedor de stories, saída da IA,
  formulário, variável de ambiente — é validada com schema antes de virar tipo
  interno. Nada entra no sistema não validado
- Tipos derivados do schema, não escritos em paralelo a ele
- Sem `enum` do TypeScript; use união de literais ou objeto `as const`

### Estrutura e dependências

- **Domínio não conhece infraestrutura.** A regra de recomendação não sabe que
  existe WhatsApp; o extrator não sabe qual provedor trouxe a imagem
- Todo serviço externo entra por uma **interface do nosso lado**, com uma
  implementação falsa para testes. Isso vale especialmente para o provedor de
  stories, que vai ser trocado (ver risco R1 no README)
- Dependência nova precisa de justificativa. Prefira a biblioteca padrão

### Erros

- Uma execução diária que roda sem ninguém olhando **não pode falhar em silêncio**
- Falha em um perfil não derruba os outros; o resultado é parcial e explícito
- Log estruturado com contexto suficiente para reconstruir o que aconteceu no dia
- Nunca "mais ou menos aceitar" uma saída da IA que não bate com o schema:
  descarte, registre e siga

### Testes

- O que precisa de teste: normalização de `@`, parsing e validação das duas
  saídas de IA (extração e recomendação), verificação de que o restaurante
  escolhido está mesmo na lista do dia, formatação da mensagem
- Testes usam as implementações falsas e respostas gravadas — **nenhum teste chama
  Instagram, WhatsApp ou a API da IA de verdade**
- Sem feedback do usuário na v1, as suítes de regressão de prompt são o **único**
  sinal de qualidade do projeto. Trate-as como tal

---

## Segurança e dados sensíveis

- **Nenhum segredo no repositório.** Tokens da Meta, chave da API de IA,
  credenciais de provedor: todos em variáveis de ambiente
- Mantenha um `.env.example` com as chaves e **sem** os valores
- **Nada de pessoal vai no prompt.** O provedor de IA é um free tier que treina
  com o conteúdo enviado e permite revisão humana. Só entram imagens de stories
  públicos, texto de cardápio e nome de estabelecimento. Telefone, nome do usuário
  e preferências pessoais estão proibidos — ver
  [ADR 0003](docs/decisions/0003-gemini-free-tier-como-provedor-de-ia.md)
- O número de WhatsApp do usuário é dado pessoal: não o coloque em log, em
  mensagem de erro, em fixture de teste nem em prompt
- Imagens de stories são conteúdo de terceiros: guarde apenas o necessário para o
  histórico, e nunca republique

---

## Parâmetros dados pelo usuário

Não são suposições — foram respondidos diretamente. Dimensione as decisões por eles.

| Parâmetro | Valor |
|---|---|
| Volume de restaurantes | **4 na v1**, travado pela grade 2×2. Até 10 é horizonte pós-v1 |
| Frequência | Dias úteis apenas |
| Perfil sem cardápio no dia | Vira tile de **"Cardápio indisponível"** na grade — a imagem nunca tem buraco |
| Orçamento | **Custo zero.** Nada de assinatura ou provedor pago |
| Número de WhatsApp | Sem número novo; número de teste gratuito da Meta como remetente |
| Histórico | 30 dias, só na interface. A conversa do WhatsApp é descartável |

Todos já acomodados nas decisões registradas. Não os contorne silenciosamente
escolhendo algo pago, que exija número novo ou que quebre a grade: levante a
questão.

## Restrições que valem a pena lembrar

Erros que já custaram tempo em projetos parecidos:

1. **A Graph API oficial do Instagram não devolve stories de perfis de terceiros.**
   Só de contas Business que o próprio usuário administra. Não gaste tempo
   procurando o endpoint — ele não existe. Ver risco R1.
2. **A Cloud API do WhatsApp exige um número dedicado para o remetente** (não para
   o destinatário) e **template aprovado** para mensagens iniciadas pelo negócio.
   A mensagem diária é iniciada pelo negócio.
3. **O provedor de IA é o free tier do Gemini** ([ADR 0003](docs/decisions/0003-gemini-free-tier-como-provedor-de-ia.md)).
   São ~5 chamadas por dia útil (até quatro imagens de story e uma de
   recomendação) contra um teto reportado de ~1.500 — folga enorme. Mas no free
   tier **o Google usa o conteúdo enviado para treinar seus
   produtos, e revisores humanos podem lê-lo**. Daí a regra abaixo.
4. **Story tem prazo de 24h.** Se a coleta não rodou, o dado se perdeu — não dá
   para buscar depois.
5. **Fuso horário:** tudo em `America/Sao_Paulo`. Um cardápio "de hoje" calculado
   em UTC dá o dia errado à noite.

---

## Como responder ao usuário

- Português do Brasil, direto, sem preâmbulo
- Recomende **uma** opção e diga por quê; não devolva um catálogo de alternativas
- Quando faltar informação para decidir, faça a pergunta específica em vez de
  assumir em silêncio
- Nunca dê como pronto o que não foi verificado
