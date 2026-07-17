# CLAUDE.md — mup-dashboard

Instruções para o Claude ao trabalhar nesta pasta.

> **Origem:** este projeto foi derivado do `maz-dashboard` (cópia limpa, sem histórico).
> Toda a arquitetura, CSS, JS, regras de negócio, índice de funções e armadilhas
> técnicas são **herdadas do MAZ e se mantêm**. A **única diferença estrutural** é a
> **fonte de dados** (ver seção "Fontes de dados"). As lógicas/regras específicas do
> MUP serão adaptadas no `index.html` ao longo do desenvolvimento — a princípio as
> funções são as mesmas; muda **de qual coluna** o código lê a informação.

## O que é este projeto

Dashboard HTML interativo do projeto **Museu do Petróleo e Novas Energias (MUP)**,
do **IDG — Instituto de Desenvolvimento e Gestão**, a ser publicado via GitHub Pages.
Usado pela **Diretoria de Projetos (PMO)** para acompanhar cronograma e requisições.

**URL pública (prevista):** https://pmo-creator.github.io/mup-dashboard/

> ⚠️ O repositório é **público**, mas a **GitHub Page ainda não foi configurada** — a URL
> acima só ficará no ar após ativar Pages (Settings → Pages → branch `main`).

## Estrutura da pasta

```
mup-dashboard/                  ← PASTA ÚNICA (git repo + tudo)
  index.html                    → Dashboard único, desktop e mobile (~425KB) — herdado do MAZ
  ONBOARDING.md                 → Guia técnico (filtros, WBS, feature N2). Herdado do MAZ,
                                  a ser adaptado ao MUP. Ler só a seção relevante, nunca inteiro.
  SERVE_DASHBOARD.bat           → Servidor local para preview (duplo-clique) — porta 8000
  CLAUDE.md                     → Este arquivo
  .gitignore                    → Arquivos ignorados pelo git (herdado do MAZ)
  .github/                      → Configs do GitHub (CODEOWNERS)
  .claude/                      → Configs locais do Claude Code (não afeta o dashboard)
  MUP _ EAP - CRONOGRAMA_16Jul.xlsx → Cronograma de origem (fonte de dados atual) — aba "master data"
```

> Obs.: a pasta `00. Apoio/` (logos/banners) do MAZ foi **removida do repo**. Adicionar
> os assets do MUP quando disponíveis.

## Ambiente local

- **Caminho local do repo:** `C:\Users\gagui\GitHub\mup-dashboard`
- **⚠️ Não usar OneDrive** — o OneDrive corrompe a pasta `.git` ao sincronizar arquivos internos do git
- **Branches:**
  - `main` → produção (GitHub Pages) — protegido, exige PR para merge
  - `marcela` e `joao` → branches de trabalho pessoais; edições vão aqui e sobem via PR para `main`
  - (não há branch `dev` neste repo — diferente do MAZ)

## Regras de trabalho

1. **Consultar ONBOARDING.md apenas se a tarefa envolver filtros, WBS ou feature N2** — e somente a seção relevante, nunca o arquivo completo.

2. **NUNCA rodar `git add`, `git commit` ou `git push` sem instrução explícita
   do usuário.** Esta é a regra mais importante — violar causa publicação acidental
   em produção.

3. **Editar index.html via Python str.replace() no bash**, nunca com
   o Edit tool — arquivos grandes truncam.

4. **Antes de refatorar ou reescrever qualquer função que funciona, perguntar e
   justificar.** Edição cirúrgica é preferível a substituição total.

5. **Scripts temporários** → deletar imediatamente após uso.

6. **Servidor local** → rodar `SERVE_DASHBOARD.bat` (duplo-clique) para preview antes
   de commitar. Ele abre http://localhost:8000 e não interfere com git.

7. **NUNCA apagar arquivos sem listar o que será deletado e aguardar confirmação
   explícita.**

8. **Lógica crítica exige revisão manual antes de aceitar** — especialmente:
    `_parseWBS`, `_worstStatus`, `preprocessStatuses`, `matchesFilter`.
    "Parece certo" não é critério de aceite.

## Fontes de dados do dashboard

> **Principal diferença em relação ao MAZ.** No MAZ os dados vinham direto do Google
> Sheets. No MUP, na fase de desenvolvimento, os dados vêm de um **arquivo `.xlsx`
> no próprio repositório** (`MUP _ EAP - CRONOGRAMA_16Jul.xlsx`), o cronograma de
> origem usado como **modelo estrutural**. Depois a estrutura será a mesma, porém
> lida via **Google Sheets API**.

- **Cronograma (fase atual):** `MUP _ EAP - CRONOGRAMA_16Jul.xlsx` na raiz do repo — aba **`master data`**.
  - Formato: **xlsx** (é o que sai direto do Google Sheets, preserva as abas e as
    **cores das células** — e o dashboard usa cor de célula em `fetchSheetColors`
    como parte da lógica). Leitura no navegador exige uma biblioteca JS de parse de
    xlsx (ex.: SheetJS) — a ser integrada ao adaptar o `index.html`.
  - Cabeçalho real da `master data` está na **linha 2** (linha 1 tem "BASELINE").
    Colunas: `B=Eixo`, `C=Grupo`, `D=Marco\Entrega`, `E=Tarefa`, `F=Fornecedor`,
    `G=Responsável`, `H=Prioridade`, `I=Status`, `J=Data Início`, `K=Data Fim`,
    `L=Duração`, `M=Predecessores`, `O=Progresso`, `P=O que andou/travou`, e as
    **colunas de área a partir de `R`** (`R=Foyer`, `S=Área 0`, `T=Área 1`, ...,
    `AB=Área externa`, `AC=Áreas comuns`).
  - Abas do arquivo: `master data`, `longa por área`, `Página8`, `Contatos`,
    `Cópia cronograma SM v24-04-2026`, `Tabela dinâmica 6`, `status das atividades`.

> 🔴 **Sensibilidade dos dados:** este xlsx tem dados reais do cronograma e o repositório
> **já é público** — qualquer pessoa consegue baixar o arquivo. Se esses dados não deveriam
> ser públicos: tornar o repo privado OU substituir o xlsx por uma versão com dados fictícios.

- **Requisições/Compras:** **on hold.** Ainda não há compras a fazer no MUP. A aba/feature
  de Requisições (herdada do MAZ) segue no código, mas fica **pausada** até existir uma
  fonte de dados de compras — só então será reativada e a leitura, adaptada.

- **Futuro:** migração para **Google Sheets API**, mantendo a mesma estrutura de abas/colunas.

## Índice de Funções — index.html

Sem números de linha (o arquivo muda de tamanho a cada edição — números ficam
errados rápido). Para localizar, usar sempre `grep -n "nomeDaFuncao" index.html`.

> Herdado do MAZ. As funções são as mesmas; ao adaptar ao MUP, o que muda é **de qual
> coluna** cada função lê. A camada de dados (`fetchSheet`, `fetchSheetColors`,
> `loadSheetsData`) será adaptada para ler o `.xlsx` local em vez do Google Sheets.

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

### ☁️ Dados (camada a adaptar para xlsx local)
fetchSheet, fetchSheetColors (aceita coluna como parâmetro: usada com 'E' para detectar linha de tarefa), loadSheetsData,
_parseWBS, _parseREQS, _sg, _fmtDate, _worstStatus, _eixoTextColor

### 🎨 Cor dos eixos
No MAZ, fixa em `EIXO_FIXED_COLOR = '#949494'` (decisão da diretoria). `_eixoTextColor(hex)`
escolhe a cor do texto (claro/escuro) por contraste contra a cor de fundo.
Revisar/ajustar conforme a identidade visual do MUP.

### 📄 Export PDF
openExportPDFWizard, pdfWizNext, pdfWizBack,
_runExportPDF, _buildGanttSVGForExport

## ⚡ Riscos de Token — mup-dashboard

Arquivos pesados conhecidos — nunca ler inteiro sem autorização:
- `index.html` (~425KB) → usar `grep`, `sed` ou busca por âncora
- `ONBOARDING.md` → ler só a seção relevante, não o arquivo completo
- `MUP _ EAP - CRONOGRAMA_16Jul.xlsx` → nunca abrir direto — extração pontual (unzip + parse XML da aba)

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
| `node --check` | Extrair bloco `<script>` para `.js` temporário |
| Edit tool falha | Usar Python `str.replace()` via bash em vez do Edit tool (ver regra 3) |
| String não encontrada | Normalizar CRLF: `content.replace("\r\n","\n")` antes de substituir, converter de volta antes de gravar |
