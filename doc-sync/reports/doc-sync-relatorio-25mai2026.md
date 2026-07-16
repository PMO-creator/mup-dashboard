# doc-sync — Relatório de mudanças detectadas
**Data:** 25/05/2026 | **Snapshot base:** 22/05/2026

---

## Commits analisados (após snapshot)

| Commit | Descrição | Classificação |
|--------|-----------|---------------|
| `155be16` | Remover dead code do bloco CDN no index.html | ❌ IGNORAR |
| `7662590` | Corrigir mapeamento de colunas do sheet de Requisições | ✅ RELEVANTE_TECH |
| `b4a0b53` | Adicionar filtro de Fornecedor + botão "Limpar filtros" na aba Requisições | ✅ RELEVANTE_UX |
| `2ea1b35` | Refinar regras de negócio de status automático | ✅ RELEVANTE_TECH |
| `8080c50` | Calcular datas de marcos e grupos automaticamente | ✅ RELEVANTE_TECH |

---

## Mudanças relevantes detalhadas

### 1. [RELEVANTE_UX] Novo filtro de Fornecedor + botão "Limpar filtros" (commit b4a0b53)
- Dropdown multi-select de fornecedor na aba Requisições (coluna G do sheet)
- Itens sem fornecedor agrupados como "Sem Fornecedor"
- Botão verde "Limpar filtros" reseta status, fornecedor, comprador, prioridade e busca
- **Afeta:** Manual §6 (aba Requisições), Onboarding §8, Guia Usuário

### 2. [RELEVANTE_TECH] Mapeamento de colunas do sheet de Requisições corrigido (commit 7662590)
- Os índices estavam deslocados, causando KPIs zerados e quadros de prioridade vazios
- **Índices CORRETOS (código atual):**
  - r[0] = N. Requisição
  - r[1] = Comprador ← era r[2]
  - r[3] = Prioridade ← era r[4]
  - r[4] = Descrição ← era r[5]
  - r[5] = Status ← era r[6]
  - r[6] = Fornecedor ← era r[7]
  - r[12] = Data prevista (sem mudança)
- ⚠️ **context.md do doc-sync ainda tem os índices ERRADOS!** Precisa ser atualizado.
- **Afeta:** Onboarding §8 (colunas do sheet), §9 (nova armadilha), context.md

### 3. [RELEVANTE_TECH] Regras de status automático refinadas (commit 2ea1b35)
- "Definir datas" agora tem prioridade maior que "A iniciar" no ranking de pior status
  - Ranking anterior: `Atrasado > Risco > Em andamento > A iniciar > Definir datas`
  - Ranking novo: `Atrasado > Risco > Em andamento > Definir datas > A iniciar`
- Subtask sem status + sem data → "Definir datas"
- "A iniciar" sem data de início → "Definir datas"
- Padrão de grupo/eixo sem status → "Definir datas"
- **Afeta:** Manual §3.5 (legenda/ranking de status), Onboarding §8, Guia Usuário

### 4. [RELEVANTE_TECH] Cálculo automático de datas de marcos e grupos (commit 8080c50)
- Marco: início = mínimo das subtasks, fim = máximo das subtasks
- Grupo: início = mínimo dos marcos, fim = máximo dos marcos
- Marcos sem subtasks mantêm datas da planilha
- **Afeta:** Onboarding §1 (arquitetura/fluxo), §8, Manual §5 (Gantt — datas aparecem automaticamente)

---

## Documentos que precisam ser atualizados

| Documento | O que atualizar |
|-----------|----------------|
| **Manual v6 → v7** | §6 (novo filtro Fornecedor + botão Limpar) · §3.5 (ordem ranking de status) · §5 (datas automáticas no Gantt) |
| **Onboarding v10 → v11** | §8 (colunas REQS corrigidas, novo filtro Fornecedor) · §9 (armadilha: mapeamento colunas REQS) · §1 (cálculo automático de datas) |
| **Guia Usuário v2 → v3** | Slide Requisições (novo filtro Fornecedor + botão Limpar) |
| **Ficha Técnica v3** | Sem mudanças — manter |
| **context.md do doc-sync** | Corrigir índices das colunas REQS (seção 2) |

---

## Para retomar em nova sessão

1. Abra nova conversa no Cowork
2. Cole: **"Retomar doc-sync — relatório em `IDG - Relatórios de Análise/doc-sync-relatorio-25mai2026.md`. Executar Etapas 6 a 9 da skill doc-sync."**
3. O Claude vai ler este relatório e partir direto para as atualizações dos documentos (sem refazer o diff).
