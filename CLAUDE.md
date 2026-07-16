# CLAUDE.md — maz-dashboard

Instruções para o Claude ao trabalhar nesta pasta.

## O que é este projeto

Dashboard HTML interativo do projeto **Museu das Amazônias 2026**, publicado via GitHub Pages.
Usado pelo time de PMO para acompanhar cronograma e requisições do projeto.

**URL pública:** https://pmo-creator.github.io/maz-dashboard/

## Estrutura da pasta

```
maz-dashboard/                  ← PASTA ÚNICA (git repo + tudo)
  index.html                    → Dashboard único, desktop e mobile (~330KB) — mobile.html removido em 01/07/2026
  Consulte ONBOARDING.md seção 7 ou 10 apenas se a tarefa envolver filtros específicos ou feature N2
  SERVE_DASHBOARD.bat           → Servidor local para preview (duplo-clique)
  CLAUDE.md                     → Este arquivo
  doc-sync/                     → Skill de sincronização de documentação
    SKILL.md                    → Lógica da skill doc-sync (fonte de verdade)
    context.md                  → Contexto técnico do dashboard
    _snapshot_index.html        → Snapshot do último doc-sync (autoritativo)
    doc-sync.skill              → Bundle instalável para Cowork
    reports/                    → Relatórios de cada execução
  Manual/                       → Documentação (versionada no git)
    DEV_GUIDE.html              → Guia técnico do desenvolvedor
    Manual de Uso e Manutenção Dashboard_v7.docx + .pdf
    Guia de Onboarding_Manutençao Dashboard_MAZ_2026_v12.docx + .pdf
    Ficha_Tecnica_Dashboard_MAZ_2026_v3.docx + .pdf
    Catalogo_Erros_Vibe_Coding_MAZ_v4.docx → Catálogo de anti-padrões (doc-sync atualiza com novos erros; versionado como os demais)
    old_versions/               → Versões anteriores arquivadas (inclui catalogo_erros_vibe_coding_v3.1_original.docx)
  00. Apoio/                    → Logos e banners
```

## Ambiente local

- **Caminho local do repo:** `C:\Users\gagui\GitHub\maz-dashboard`
- **⚠️ Não usar OneDrive** — o OneDrive corrompe a pasta `.git` ao sincronizar arquivos internos do git
- **Branches:**
  - `main` → produção (GitHub Pages) — protegido, exige PR para merge
  - `dev` → branch de trabalho padrão — todas as edições vão aqui

## Regras de trabalho

1. **Consultar ONBOARDING.md apenas se a tarefa envolver filtros, WBS ou feature N2** — e somente a seção relevante, nunca o arquivo completo.

2. **NUNCA rodar `git add`, `git commit` ou `git push` sem instrução explícita
   do usuário.** Esta é a regra mais importante — violar causa publicação acidental
   em produção.

3. **Editar index.html via Python str.replace() no bash**, nunca com
   o Edit tool — arquivos grandes truncam.

4. **Novos relatórios doc-sync** → salvar sempre em `doc-sync/reports/`.

5. **Versões antigas de documentos** → mover para `Manual/old_versions/` ao criar nova versão.

6. **Não gerar PDF automaticamente** (decisão Jul/2026) → doc-sync e demais
   fluxos produzem só o `.docx`. Quem precisar do PDF exporta manualmente
   quando for consumir o documento.

7. **Servidor local** → rodar `SERVE_DASHBOARD.bat` (duplo-clique) para preview antes
   de commitar. Ele abre http://localhost:8000 e não interfere com git.

8. **Scripts temporários** → deletar imediatamente após uso.

9. **Lógica crítica exige revisão manual antes de aceitar** — especialmente:
    `_parseWBS`, `_worstStatus`, `preprocessStatuses`, `matchesFilter`.
    "Parece certo" não é critério de aceite.

## Como rodar o doc-sync

Invocar a skill `doc-sync` diretamente no chat Cowork com maz-dashboard montado.

## Índice de Funções — index.html

Sem números de linha (o arquivo muda de tamanho a cada edição — números ficam
errados rápido). Para localizar, usar sempre `grep -n "nomeDaFuncao" index.html`.

### 🔧 Utilitários
fmtBR, fmtBRshort, escH, escSVG, getISOWeek, hl, abrevNome

### ⏳ Loading / Estado UI
showLoading, hideLoading, showConnErr, hideConnErr, retrySheets

### 🗂️ Navegação
switchTab

### 🔍 Filtros
buildRespFilter, getFilters, dateInRange, matchesFilter,
resetAllFilters, clearDates, applyFilter, updateFilterChip,
setFilterLevel, toggleModoEstrito, updatePeriodoBtn

### 📊 KPIs e Charts
renderKPIs, renderCharts, preprocessStatuses

### 🌳 Árvore WBS / EAP
renderTree, toggleRoot, toggleGroup, toggleComment,
toggleMarco, expandAll, collapseAll, buildCommentPanel,
expandAllEixosEAP, expandAllMarcosEAP, expandAllTarefasEAP

### 📋 N2 Pauta
loadN2, saveN2, toggleN2Marco, updateN2Fab,
clearN2Selection, applyN2Filter, publishN2Pauta, unlockN2Edit,
exportN2PPT (gera o export em HTML navegável, nome mantido por compatibilidade)

### 🛒 Requisições
buildReqStatusDropdown, buildReqFornDropdown,
renderReqKPIs, renderReqs, toggleReqs, clearReqFilters

### 📅 Gantt
renderGanttSection, toggleGantt, toggleGanttGroup,
expandAllGantt, renderGanttForEixo, setGanttMode,
expandAllTarefasGantt, expandLevel

### 🗺️ Áreas
buildAreasFilter, renderAreasTab, renderAreaGantt,
toggleAreaSection, toggleAreaMarco, setAreasGanttMode

### 🏛️ Diretoria — Gantt/EAP/N2/Comparativo
preprocessStatusesDiretoria, renderKPIs_Dir, renderTree_Dir, buildCommentPanel_Dir,
toggleGroup_Dir, toggleComment_Dir, toggleMarco_Dir, collapseAll_Dir,
expandAllEixosEAP_Dir, expandAllMarcosEAP_Dir, expandAllTarefasEAP_Dir,
renderGanttSectionDir, renderGanttForEixoDir, toggleGanttDir, setGanttModeDir,
expandAllGanttDir, collapseAllGanttDir,
loadN2_Dir, saveN2_Dir, toggleN2Task_Dir, updateN2Fab_Dir, publishN2Pauta_Dir,
unlockN2Edit_Dir, exportN2PPT_Dir (gera o export em HTML navegável, nome mantido por compatibilidade),
buildComparativoModal, openComparativoModal, closeComparativoModal

### ☁️ Google Sheets / Dados
fetchSheet, fetchSheetColors (aceita coluna como parâmetro: usada com 'E' para detectar linha de tarefa), loadSheetsData,
_parseWBS, _parseREQS, _sg, _fmtDate, _worstStatus, _eixoTextColor

### 🎨 Cor dos eixos
Fixa em `EIXO_FIXED_COLOR = '#949494'` (decisão da diretoria, Jul/2026) — não vem mais da coluna B
da planilha. `_eixoTextColor(hex)` só escolhe a cor do texto (claro/escuro) por contraste contra
essa cor de fundo fixa.

### 📄 Export PDF
openExportPDFWizard, pdfWizNext, pdfWizBack,
_runExportPDF, _buildGanttSVGForExport


## Fontes de dados do dashboard

- Cronograma: Google Sheets `17nttJ_ShqWztvDWH3l59iNqboLqkviZs3_PM5J3ihdA` — aba `master data`
- Requisições: Google Sheets `1azrdS4OGO-CWD1ods69i8iZJcwq4oyISdT2n_tu1uJM` — aba `Planilha de Status de Compras Prod`


## ⚡ Riscos de Token — maz-dashboard

Arquivos pesados conhecidos — nunca ler inteiro sem autorização:
- `index.html` (~397KB / ~6.283 linhas) → usar `grep`, `sed` ou busca por âncora
- `ONBOARDING.md` → ler só a seção relevante, não o arquivo completo
- `Manual/*.docx` → nunca abrir direto — usar PDF ou extração pontual

Antes de qualquer operação nesses arquivos, aplicar regra do Token Management global:
perguntar + listar alternativas leves.

## Roteamento de Edições — index.html

Antes de editar, classificar a mudança:

- **Visual** (cor, fonte, espaçamento, ícone)
  → `grep -n "nome-da-classe" index.html` → editar só esse trecho do `<style>`

- **Lógica** (cálculo, filtro, status, regra de negócio)
  → `grep -n "nomeDaFuncao" index.html` → editar só esse trecho do `<script>`

- **Estrutura HTML** (nova coluna, novo card, novo bloco)
  → Python str.replace() com âncora mínima (~10 linhas únicas ao redor)
  → NUNCA ler o arquivo inteiro para mudança pontual

Regra universal: grep primeiro, editar depois. Nunca abrir o arquivo completo.

## Armadilhas Técnicas — index.html
| Armadilha | Como evitar |
|---|---|
| Dashboard branco sem erro | null bytes, `function` ausente, JS truncado, template literals aninhados |
| Template literals aninhados | Nunca crase dentro de `${}` dentro de outro crase |
| JS truncado | Verificar `</script>` no final antes de editar |
| Prioridade REQS | Usar coluna D (índice 3) — nunca coluna B |
| `node --check` | Extrair bloco `<script>` para `.js` temporário |
| Edit tool falha | Usar Python `str.replace()` via bash em vez do Edit tool (ver regra 3) |
| String não encontrada | Normalizar CRLF: `content.replace("\r\n","\n")` antes de substituir, converter de volta antes de gravar |
