---
name: meirelles-agent-prompts
description: Boas práticas de prompt engineering para agentes do MEIrelles (sistema multi-agente WhatsApp para MEIs, construído em N8N). Use esta skill sempre que escrever, revisar, auditar ou versionar (v3.x, v4.x etc.) o system prompt de qualquer agente MEIrelles — Pivô, Onboarding, Contador, Business, Agenda, Políticas, Comunidade ou qualquer agente novo. Também use ao produzir documentação de diff para o Hebert, ao definir output contracts, ao desenhar fluxos de consentimento/dados sensíveis, ou ao decidir o formato de resposta (singular vs. lista) baseado em quantidade de resultados. Acione mesmo que o pedido seja só "revisa esse prompt" ou "ajuda a escrever o Agente X" sem mencionar explicitamente "boas práticas".
---

# Prompt Engineering para Agentes MEIrelles

Esta skill captura os padrões e convenções que já funcionam nos prompts do MEIrelles, validados em 16 agentes (0-15) + o documento mestre de produto. Use como checklist ao escrever, revisar ou versionar qualquer prompt de agente.

**Estrutura de referências:**
- `references/00-base-conhecimento-produto.md` — **Fonte de verdade de produto.** Edite aqui primeiro, propague pros agentes depois (nunca o contrário). Contém tabela de distribuição por seção e notas de divergência com os prompts vivos.
- `references/agente-0-personalidade.md` — Sistema de valores global
- `references/agente-1-pivo.md` — Padrão roteador puro
- `references/agente-2-onboarding.md` — Fluxo progressivo conversacional
- `references/agente-3-financeiro.md` — CRUD transacional
- `references/agente-4-business.md` — Análise + calendário
- `references/agente-5-agenda.md` — Coleta temporizada + confirmação
- `references/agente-6-pagamento.md` — Argumentação dinâmica + copywriting
- `references/agente-7-nota-fiscal.md` — First-use flow + API lookup
- `references/agente-8-politicas.md` — Q&A por tópico + acolhimento
- `references/agente-9-objetivo.md` — Fluxo de projeto com feedback de resultado
- `references/agente-10-feedback.md` — Coleta não-defensiva de feedback
- `references/agente-11-lista.md` — CRUD simples + confirmação implícita
- `references/agente-12-collab.md` — Consentimento duplo + output contracts
- `references/agente-13-duvidas-mei.md` — Q&A educativo + encaminhamento responsável
- `references/agente-14-cardapio.md` — Ativação multi-perfil + cruzamento automático de preços
- `references/agente-15-precificacao.md` — Estratégia com dados + fórmulas

## Estrutura padrão de um prompt de agente

Todo prompt de agente MEIrelles deve conter, nessa ordem:

1. **Cabeçalho de identificação** — nome do agente, versão (vX.Y), contexto do produto (MEIrelles | WhatsApp para MEIs).
2. **Instrução para o desenvolvedor** — quando/como o agente é ativado (gatilho direto via Pivô, gatilho proativo via outro agente, etc.). Isso é documentação operacional para o Hebert, não faz parte do system prompt em si.
3. **Requisitos de banco de dados** — campos necessários no cadastro/contexto para o agente funcionar (ex: `localizacao`, `tipo_servico`, `conexoes_realizadas`).
4. **Variáveis de contexto necessárias** — lista explícita do que precisa estar disponível no momento da chamada (nome, cluster, perfil_tipo, etc.). Se uma variável já está garantida no contexto, o prompt deve dizer explicitamente "NÃO peça de novo" — nunca deixe o agente redundante.
5. **Papel do agente** — 1-2 parágrafos definindo o que o agente faz e, criticamente, o que ele NUNCA faz (limites de escopo, dados que não compartilha sem autorização).
6. **Regras críticas nomeadas** (ex: "CONSENTIMENTO DUPLO") — qualquer fluxo que envolva dados sensíveis, autorização de terceiros, ou ordem obrigatória de etapas merece uma seção própria com nome em caixa alta e aviso explícito tipo "⚠️ ORDEM CRÍTICA — NUNCA PULE".
7. **Gatilhos** (Gatilho 1, Gatilho 2...) ou **Fluxos** (Fluxo de Primeiro Uso, Fluxo de Emissão, etc.) — cada forma de ativação do agente documentada separadamente, com passo a passo.
8. **Regras absolutas** (checklist final com ✓) — resumo compacto de tudo que não pode ser violado, repetindo os pontos mais críticos das seções anteriores. Serve como último filtro de sanidade.

## 7 Padrões Arquiteturais Generalizados

Esses padrões aparecem repetidos entre múltiplos agentes MEIrelles. Use-os como moldes reutilizáveis ao escrever novos agentes.

### **1. Branching por Quantidade de Resultados** _(Agentes 12, 1, 4)_
Quando o agente pode retornar 0, 1, 2-3, ou N>3 resultados, **cada faixa tem formato próprio e não podem se misturar**:
- **0 resultados**: aviso sem drama, sem oferecer alternativa vaga, sem prometer prazo. Ex: "ainda não tenho X cadastrado... quando aparecer, eu te aviso."
- **1 resultado**: SEMPRE parágrafo único, sem hífen, sem a palavra "opções" (é singular). Nome, descrição, pergunta direta — tudo corrido.
- **2-3 resultados**: SEMPRE lista com quebras de linha, hífen (-) para cada item, usa a palavra "opções" (plural).
- **N>3 resultados**: precisa de um algoritmo determinístico de seleção (ex: ranking por proximidade geográfica, depois compatibilidade, depois desempate) — nunca liste tudo, nunca escolha "no olho".

Documente essa distinção explicitamente no prompt — é fonte comum de erro do modelo.

**Exemplo:** Agente 12 (Comunidade), Agente 1 (Pivô)

### **2. Confirmação Obrigatória Antes de Alterar** _(Agentes 3, 5, 6, 7)_
Nenhum registro, agendamento, pagamento ou emissão acontece sem confirmação explícita do profissional.

Padrão:
1. Mostre o resumo do que será feito
2. Apresente opções (Sim / Quero corrigir)
3. APENAS se Sim → execute
4. Se quiser corrigir → volte para a coleta, não para o resumo

Nunca cobre, cancele ou altere sem OK explícito. A responsabilidade é sempre do profissional.

**Exemplo:** Agente 3 (Financeiro), Agente 5 (Agenda), Agente 6 (Pagamento)

### **3. Coleta Temporizada: Delayed Onboarding** _(Agentes 2, 5, 6)_
Não coleta tudo de uma vez na primeira interação. Dados são coletados em momentos específicos (1ª segunda-feira, 2º mês, trial encerrado) e de forma gradual.

Padrão:
- Primeiro contato: coleta mínimo crítico
- 1ª segunda-feira: coleta complementar (descrição detalhada)
- 2º mês: coleta Contas Fixas
- Encerramento de trial: coleta consentimento

Nunca interrompa um fluxo em andamento pra coletar dados da fila — processe o pedido primeiro.

**Exemplo:** Agente 2 (Onboarding), Agente 5 (Agenda), Agente 6 (Pagamento)

### **4. Calendário + Gatilhos Secundários** _(Agente 4)_
Quando o agente toca dados financeiros (vendas, gastos, faturamento) ou temporais (datas), usar calendário anual explícito com sazonalidades por cluster.

Padrão:
- Calendário mês-a-mês com datas relevantes (Carnaval, Dia das Mães, Black Friday, etc.)
- Cada data com sugestão criativa específica pro cluster
- Gatilhos secundários (ex: Collab quando `gastos_recorrentes` aparecem 2+ meses)
- Timing: aviso com antecedência (7-14 dias), lembrete na semana

Sempre relevante pro cluster do profissional — nunca mencione datas irrelevantes.

**Exemplo:** Agente 4 (Business)

### **5. Argumentação Dinâmica por Estágio de Uso** _(Agente 6)_
Quando o agente apresenta propostas (venda, upgrade, mudança de plano), a mensagem muda conforme `mes_uso`:

**Mês 1 (sem dados reais):** tom de possibilidade ("dá pra descobrir", "você poderia"), exemplos baseados no que é típico do cluster
**Mês 2+ (com dados reais):** tom de constatação ("você já", "seus dados mostram"), exemplos do Agente Business, com anchor de preço relativa (comparar R$ 30 com 1 unidade do serviço mais vendido)

Nunca generalize — sempre adapte ao cluster e ao momento.

**Exemplo:** Agente 6 (Pagamento)

### **6. First-Use Flow: Registro Único de Dados Complexos** _(Agentes 2, 7)_
Quando o agente coleta dados complexos ou sensíveis (certificado, senha, consentimentos), separar o fluxo de primeiro uso (7+ passos) do fluxo de uso subsequente (3-4 passos).

Padrão:
- Primeiro uso: confirmação completa de dados, transparência sobre o que é permanente
- Uso subsequente: dados já salvos, acesso direto ao fluxo de emissão/registro
- Sinalizar claramente: "Nunca mais vou te pedir esses dados"

Busca automática por API quando possível (CNPJ → razão social, endereço, etc.) — nunca pedir dado que pode ser olhado automaticamente.

**Exemplo:** Agente 2 (Onboarding), Agente 7 (Nota Fiscal)

### **7. Output Contracts + Regras Críticas Nomeadas** _(Agentes 12, 2, 3, 5)_
Sempre que o agente dispara uma ação estruturada, defina um contrato de saída nomeado em caixa alta com colchetes: `[SEND_CONTACT]`, `[AGENDA_CREATE]`, `[RIGHTS_REQUEST]`, etc.

Regras críticas que envolvem ordem obrigatória ou dados sensíveis ganham seção própria:
- Nome em caixa alta (ex: "CONSENTIMENTO DUPLO", "NUNCA REGISTRE SEM CONFIRMAÇÃO")
- Aviso explícito (⚠️)
- Ordem de etapas numerada se aplicável
- Exemplo operacional completo

Isso facilita auditoria e integração com N8N (o nó consegue parsear contratos determinísticos).

**Exemplo:** Agente 12 (Comunidade — CONSENTIMENTO DUPLO), Agente 3 (Financeiro — MOSTRAR + SALVAR + ABRIR CORREÇÃO)

### Algoritmos determinísticos para desempate/seleção
Quando há mais opções do que cabe mostrar, o prompt precisa de um algoritmo passo a passo com critério de prioridade explícito e ordenado (ex: 1º proximidade, 2º compatibilidade, 3º critério de desempate). Inclua sempre um **exemplo operacional completo** (com nomes fictícios) mostrando a lógica sendo aplicada do início ao fim — isso reduz ambiguidade muito mais que só descrever a regra.

### Regras de output visual
- Nunca usar opções numéricas (nem "1. Sim / 2. Não" nem "1) Sim 2) Não") — sempre hífen (-) para listas de itens, e opções de resposta cada uma em linha própria.
- Nunca usar emojis no texto de resposta ao usuário (diferente de emojis de categorização interna, como o "canonical emoji-per-cluster table", que é uma convenção separada).
- Seções com propósitos diferentes (aviso / oferta / opções) ficam em parágrafos separados, nunca coladas.

## Consentimento e dados sensíveis

Sempre que um agente compartilha dado de uma pessoa com outra (contato, informação de negócio, etc.), aplicar o padrão de **consentimento duplo explícito**:

1. Pergunta à parte que pediu se pode agir.
2. **Somente se sim**, pergunta à parte que será exposta se autoriza o compartilhamento.
3. **Somente se sim de ambos**, executa o compartilhamento via output contract (ex: `send-contact`/vCard) e registra a conexão para ambas as partes.

Nunca pule a etapa 2. Isso deve estar marcado no prompt como regra crítica nomeada, não como sugestão.

Trate respostas de "talvez depois" como uma categoria própria — nem aceite nem recusa. Não bloquear reoferta futura, mas também não repetir na mesma janela.

## Output contracts

Sempre que o agente precisa disparar uma ação estruturada (enviar contato, criar agendamento, registrar direito, mudar perfil), defina um contrato de saída explícito e nomeado em caixa alta com colchetes, ex: `[SEND_CONTACT]`, `[AGENDA_CREATE]`, `[SET_PROFILE]`, `[RIGHTS_REQUEST]`. Documente:
- Quando disparar
- Quais campos ele carrega
- O que NÃO fazer (ex: "NÃO enviar número por extenso no texto")

Isso facilita tanto a auditoria do prompt quanto a integração com N8N (o node consegue parsear o contrato de forma determinística em vez de fazer parsing de linguagem natural).

## Adaptação de vocabulário por perfil

Se o agente atende públicos diferentes (MEI, Autônomo, Profissional Liberal), documente explicitamente o mapeamento de vocabulário por perfil (ex: MEI/Autônomo → "negócio", "cliente"; PL → "consultório", "paciente"). Não deixe o modelo inferir — liste o de-para.

## Regras absolutas (checklist final)

Todo prompt deve terminar com uma seção compacta de regras absolutas (✓), reforçando:
- A regra crítica nomeada (ex: consentimento duplo) nunca pode ser pulada.
- Formatos de lista/resposta não podem se misturar.
- Nenhum dado sensível sai sem autorização explícita.
- Como encerrar sem drama quando algo é recusado ou não há match.
- Onde registrar o resultado da interação (ex: `conexoes_realizadas`) para auditoria futura.

## Regras de Linguagem Global (Agente 0 — Personalidade)

Essas regras aparecem no Agente 0 e propagam pra todos os demais agentes:

- **Zero emojis** (exceto 🔔 em lembretes automáticos, 🟢🟡⚪️ em saldos fiscais — exceções explícitas)
- **Zero opções numéricas** — sempre hífen (-) em listas de itens; opções de resposta cada uma em linha própria
- **Nunca "MEI" no fluxo conversacional** — use profissão específica ou "profissional"; "MEI" aparece apenas em contextos legais (DAS, Declaração Anual, CNPJ)
- **Linguagem adaptada por cluster** — vocabulário de-para explícito (ex: Comida → "vendas", "insumos"; Rodas → "corridas", "combustível")
- **Clareza antes de tudo** — comprimento de resposta é consequência do conteúdo, nunca do estilo
- **Nunca fragmentar em múltiplas mensagens** — entregar resposta completa de uma vez
- **Responder em tempo presente, não em futuro** — não incentive diálogo desnecessário; resolva com ação

## Correção guiada por testes (Test-Driven Prompt Fix)

Quando um teste/caso de avaliação (EvalLab, Phoenix, teste manual) encontra um comportamento incorreto, a correção do prompt deve deixar rastro explícito — não só corrigir e seguir em frente.

### Padrão observado (Agente 1 — Pivô, v6.0 → v6.2)

Um teste de avaliação (caso 08b) encontrou dois problemas simultâneos:
1. Roteamento incorreto — mensagens com "faturamento"/"lucro" junto de verbo de planejamento futuro ("quero definir uma meta de faturamento") caíam no Agente 4 (Análise) quando deveriam cair no Agente 9 (Objetivo)
2. Quebra da REGRA 1 — o Pivô gerava texto de transição antes de rotear, quando sua única saída deveria ser o roteamento em si

A correção documentou ambos os problemas diretamente no changelog da versão, citando o caso de teste que os encontrou:

```
# NOVO v6.2:
# - Desambiguação das keywords "faturamento"
#   e "lucro" — só indicam Análise (Agente 4)
#   quando a intenção é consultar o passado;
#   com verbo de planejamento futuro, indicam
#   Objetivo (Agente 9). Corrige roteamento
#   incorreto achado no teste caso 08b.
# - REGRA 1 reforçada: proibido gerar texto
#   explicativo/de transição antes de rotear,
#   mesmo em caso de ambiguidade. Corrige
#   quebra de REGRA 1 achada no teste caso 08b.
```

### Regras da prática

1. **Cite o caso de teste no changelog.** "Corrige roteamento incorreto achado no teste caso 08b" — não apenas "melhoria na desambiguação". Isso cria rastreabilidade entre eval e prompt.
2. **Separe sintoma de causa raiz quando há mais de um problema no mesmo teste.** O caso 08b revelou dois bugs (roteamento errado E quebra de regra) — cada um vira uma entrada própria no changelog, não uma linha genérica.
3. **A correção da regra é sempre mais explícita que a regra original.** Não baste corrigir o comportamento — reforce o texto da regra pra reduzir a chance do mesmo erro se repetir (ex: REGRA 1 ganhou um parágrafo extra citando explicitamente frases proibidas como exemplo).
4. **Adicione exemplo operacional do caso que falhava**, sempre que a correção for de lógica/desambiguação (ver Padrão 1 e "Algoritmos determinísticos" acima) — mostrar a mensagem exata que confundia o modelo e o roteamento correto esperado.
5. **Nunca leve a correção como "ajuste de redação".** Se um teste encontrou o problema, é uma mudança de comportamento — trate como tal no diff pro Hebert (ver seção "Ao revisar/versionar um prompt existente").

### Ao documentar esse tipo de correção pro Hebert

Inclua sempre:
- Qual teste/caso encontrou o problema
- Qual era o comportamento incorreto observado
- Qual é o comportamento correto esperado (com exemplo de mensagem real ou representativa)
- O que mudou no prompt pra corrigir — trecho antes/depois se relevante

## Ao revisar/versionar um prompt existente

Ao produzir um diff (ex: v3.1 → v3.2) para o Hebert:
- Aponte especificamente qual regra crítica, output contract, ou padrão de branching mudou — não apenas "melhorias gerais".
- Sinalize "impossibilidades arquiteturais" (ex: fluxos multi-sessão que o N8N não sustenta) separadamente de ajustes de linguagem/tom.
- Separe mudanças de conteúdo (regras/lógica) de mudanças de documento (organização/clareza).
- Documente com exemplo operacional quando for mudança de lógica, não só antes/depois textual.

## Checklist para novo agente

Antes de finalizar um novo agente, verificar:

- [ ] Estrutura de cabeçalho (identificação, instrução para dev, requisitos, variáveis, papel, regras críticas, gatilhos/fluxos, regras absolutas)
- [ ] Padrão 1: branching por quantidade explícito? (se aplicável)
- [ ] Padrão 2: confirmação obrigatória antes de alterar/emitir? (se aplicável)
- [ ] Padrão 3: coleta temporizada respeita ordem de prioridade? (se aplicável)
- [ ] Padrão 4: calendário anual com sazonalidades? (se aplicável)
- [ ] Padrão 5: argumentação/mensagem varia por `mes_uso`? (se aplicável)
- [ ] Padrão 6: first-use flow separado do fluxo subsequente? (se aplicável)
- [ ] Padrão 7: output contracts nomeados e regras críticas explícitas?
- [ ] Linguagem global: zero emojis (exceto permitidos), zero numéricas, nunca "MEI" conversacional, cluster-específica
- [ ] Exemplos operacionais completos (não apenas estrutura, mas como realmente funciona)
- [ ] Nenhuma coleta desnecessária (se pode ser buscada por API, busque automaticamente)
- [ ] Se a mudança corrige um bug achado em teste: changelog cita o caso de teste, separa sintoma de causa raiz, e inclui exemplo operacional do caso que falhava
