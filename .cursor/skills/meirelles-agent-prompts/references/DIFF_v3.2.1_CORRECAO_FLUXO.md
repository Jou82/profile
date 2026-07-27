# DIFF — Agente 12 Comunidade v3.2.1 (Correção de Fluxo)

**Data**: 25/07/2026  
**Motivo**: Gap identificado em TC 1 — agente reconhecia intenção mas não apresentava profissional antes de pedir consentimento  
**Status**: CORRIGIDO  

---

## Mudança Principal

### ❌ Antes (v3.2.1 Original — FLUXO 1)

```
### PASSO 2 — BUSCAR COMPATIBILIDADE
Consultar rede_meirelles e filtrar por:
- Cluster compatível
- Localização (se mencionada)
- Especialidade (se relevante)

Se encontrar match: prosseguir para Etapa 1 do Consentimento Duplo.

Se não encontrar: "Ainda não temos ninguém exatamente com esse perfil aqui..."

### PASSO 3 — CONSENTIMENTO DUPLO
Seguir REGRA CRÍTICA acima.
```

**Problema**: Após buscar, agente pulava direto para "Etapa 1 do Consentimento Duplo" SEM apresentar o profissional encontrado.

**Comportamento Observado**:
```
Usuário: "Tô procurando um eletricista aqui em Pinheiros"
Agente: "Oi! Consigo te ajudar... Vou dar uma olhada na rede..."
         [Sem apresentar profissional]
         [Sem pedir consentimento]
```

---

### ✅ Depois (v3.2.1 Corrigido — FLUXO 1)

```
### PASSO 2 — BUSCAR COMPATIBILIDADE
Consultar rede_meirelles e filtrar por:
- Cluster compatível
- Localização (se mencionada)
- Especialidade (se relevante)

Se encontrar match: prosseguir para Passo 2.5.

Se não encontrar: "Ainda não temos ninguém exatamente com esse perfil aqui..."

### PASSO 2.5 — APRESENTAR O PROFISSIONAL ⭐ NOVO v3.2.1
Após encontrar match, apresentar de forma clara:

"Achei alguém que pode ser perfeito pro seu [negócio]:

[NOME] — [Profissão]
[Localização] | [Especialidade/Descrição breve]

Posso apresentar você?"

REGRAS:
- Ser conciso (1-2 linhas de apresentação)
- Sempre incluir: NOME, PROFISSÃO, LOCALIZAÇÃO, DESCRIÇÃO
- Aguardar resposta ANTES de prosseguir
- Não mencionar mecanismo de envio de contato

SE RESPOSTA FOR SIM → Prosseguir para Passo 3
SE RESPOSTA FOR NÃO → Registrar em ofertas_rejeitadas e encerrar naturalmente
SE RESPOSTA FOR "TALVEZ DEPOIS" → Registrar em interesse_futuro e encerrar

### PASSO 3 — CONSENTIMENTO DUPLO
Apenas após SIM no Passo 2.5:
Seguir REGRA CRÍTICA acima...
```

**Benefício**: Agente agora apresenta profissional completo ANTES de pedir consentimento.

**Comportamento Esperado Agora**:
```
Usuário: "Tô procurando um eletricista aqui em Pinheiros"
Agente: "Achei alguém que pode ser perfeito pro seu negócio:
         
         João Silva — Eletricista
         Pinheiros | Especializado em residências
         
         Posso apresentar você?"
```

---

## Mudança Secundária — Header de Contexto

### ❌ Antes

```
# NOVO v3.2.1:
# - [SEND_CONTACT] marcado como PENDENTE DE
#   VALIDAÇÃO DE CANAL...
```

### ✅ Depois

```
# NOVO v3.2.1:
# - [SEND_CONTACT] marcado como PENDENTE DE
#   VALIDAÇÃO DE CANAL...
#
# NOVO v3.2.1 (Correção de Fluxo):
# - PASSO 2.5 ADICIONADO: APRESENTAR O
#   PROFISSIONAL explicitamente ANTES de pedir
#   consentimento duplo. Corrige gap onde agente
#   reconhecia intenção mas não apresentava
#   profissional listado antes de "Etapa 1 do
#   Consentimento Duplo". Agora fluxo é:
#   Entender → Buscar → APRESENTAR → Consentimento
```

---

## Mudança Terciária — ESTRUTURA CRÍTICA

### ❌ Antes

```
# ESTRUTURA CRÍTICA:
# 1. Identificar o profissional solicitante
# 2. Registrar a solicitação
# 3. CONSENTIMENTO DUPLO — perguntar
#    aos dois antes de conectar
# 4. [SEND_CONTACT] — enviar vCard
# 5. Registrar a conexão realizada
```

### ✅ Depois

```
# ESTRUTURA CRÍTICA:
# 1. Identificar o profissional solicitante
# 2. Registrar a solicitação
# 3. APRESENTAR O PROFISSIONAL (novo)
# 4. CONSENTIMENTO DUPLO — perguntar
#    aos dois antes de conectar
# 5. [SEND_CONTACT] — enviar vCard
# 6. Registrar a conexão realizada
```

---

## Impacto nos Test Cases

| Test Case | Afetado? | Mudança |
|-----------|----------|---------|
| **TC 1** | ✅ Sim | Agora aceita 1+ profissionais com detalhes (foi refatorado anteriormente) + espera apresentação clara |
| **TC 2** | ✅ Sim | Fluxo base de desambiguação mantém Passo 2.5 |
| **TC 3** | ⚠️ Mínimo | Reciclagem não é afetada (ocorre após apresentação) |
| **TC 4** | ✅ Sim | Ranking com distância agora inclui apresentação clara (Passo 2.5) |
| **TC 5** | ❌ Não | Consentimento duplo é após apresentação (já cobertura) |
| **TC 6** | ❌ Não | Máquina de estados é independente |
| **TC 7** | ✅ Sim | Fallback também inclui Passo 2.5 (ou encerrar se 0 resultados) |
| **TC 8** | ❌ Não | Cancelamento é após conexão |

---

## Validação Esperada

**Antes da Correção**:
- ❌ TC 1: Falha (agente não apresentava profissional)
- ❌ Judge Score: 56% (reconhecia intenção, mas não listava)

**Depois da Correção**:
- ✅ TC 1: Passa (agente apresenta nome, profissão, localização, descrição)
- ✅ Judge Score: Esperado ≥ 8/10 (apresentação clara + pedido de consentimento)

---

## Próximos Passos

1. **Hebert**: Deploy da versão corrigida (`agente12_comunidade_v3.2.1_CORRIGIDO.txt`)
2. **Joana**: Rodar avaliação completa no EvalLab com 8 TCs
3. **Verificar Score**: Esperado ≥ 90.4% (manutenção de v3.1)

---

**Arquivo Original**: `agente12_comunidade_v3.2.1.txt`  
**Arquivo Corrigido**: `agente12_comunidade_v3.2.1_CORRIGIDO.txt`  
**Changelog**: Vide seção final do arquivo corrigido
