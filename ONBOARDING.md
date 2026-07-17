# Guia de Onboarding — Dashboard MUP
> Documento para novos desenvolvedores/responsáveis pelo projeto.
> Leia do início ao fim antes de fazer qualquer alteração.

---

## Índice
1. [O que é este projeto](#1-o-que-é-este-projeto)
2. [Boas práticas de desenvolvimento](#2-boas-práticas-de-desenvolvimento)
3. [Configuração inicial](#3-configuração-inicial)
4. [Fluxo de trabalho — do teste ao ar](#4-fluxo-de-trabalho--do-teste-ao-ar)
5. [Testar no celular pela rede local](#5-testar-no-celular-pela-rede-local)
6. [Reverter para uma versão anterior](#6-reverter-para-uma-versão-anterior)
7. [Referências técnicas](#7-referências-técnicas)
8. [Skills disponíveis](#8-skills-disponíveis)
9. [Armadilhas técnicas conhecidas](#9-armadilhas-técnicas-conhecidas)
10. [Feature: Pauta N2](#10-feature-pauta-n2)
11. [Feature: Aba Diretoria](#11-feature-aba-diretoria)

---

## 1. O que é este projeto

Dashboard interativo do **Museu do Petróleo e Novas Energias (MUP)**, do **IDG — Instituto de Desenvolvimento e Gestão** — acompanhamento de cronograma, status report e requisições de compras, usado pela Diretoria de Projetos (PMO).

> Este projeto foi derivado do `maz-dashboard` (cópia limpa, sem histórico). Arquitetura, CSS, JS e regras de negócio são herdados do MAZ — a diferença estrutural é a **fonte de dados** (ver abaixo).

### Arquitetura
- **Um único arquivo HTML** é o projeto inteiro: `index.html` → desktop e mobile no mesmo arquivo (responsivo, ≤768px)
- **Fase atual: dados vêm de um arquivo `.xlsx` local no próprio repositório** — lido no browser via SheetJS, sem backend (ver seção 7)
- **Publicado (previsto)** em `https://pmo-creator.github.io/mup-dashboard/` via GitHub Pages
  - ⚠️ O repositório **já é público**, mas a **GitHub Page ainda não foi ativada** (Settings → Pages → branch `main`) — a URL acima não está no ar ainda
- **Dependências externas (CDN):**
  - Chart.js 4.5.1 — gráficos de KPI
  - jsPDF 2.5.1 + svg2pdf.js 2.2.3 — export PDF
  - SheetJS (xlsx) 0.18.5, com SRI — leitura do cronograma local

### Como os dados chegam
```
MUP _ EAP - CRONOGRAMA_16Jul.xlsx  (arquivo no repositório)
        ↓  fetch + parse via SheetJS (tudo no browser, sem backend)
    Browser do usuário
        ↓  renderiza
    Dashboard (index.html — desktop e mobile no mesmo arquivo)
```

> **Futuro:** migração para **Google Sheets API**, mantendo a mesma estrutura de abas/colunas (ver seção 7 — os IDs de planilha já estão preservados no código, hoje sem uso).

### Indicador de status (canto superior direito)
| Indicador | Significado |
|---|---|
| 🟢 Ao vivo · HH:MM | Xlsx local lido com sucesso, dados atualizados na tela |
| 🟡 Cronograma OK · REQ erro | Cronograma OK, mas a aba de Requisições não foi encontrada (esperado — Requisições está **on hold**, ver seção 7) |
| 🔴 Erro — dados locais | Falha ao buscar/ler o xlsx, mostrando dados antigos |

> O texto "Ao vivo" é herdado do MAZ (onde de fato buscava do Google Sheets a cada refresh). Hoje, no MUP, significa "leu o arquivo xlsx local com sucesso" — não há conexão de rede com uma fonte externa nesta fase.

---

## 2. Boas práticas de desenvolvimento

### Ferramentas certas para cada tarefa

| Tipo de tarefa | Melhor ferramenta | Por quê |
|---|---|---|
| Editar arquivos HTML/CSS/JS | **Claude Code** | Acessa e edita arquivos diretamente |
| Git commit / push | **Claude Code** (ou terminal, se o hook de auditoria travar — ver seção 9) | Roda bash/git |
| Debug de código em arquivos | **Claude Code** | Lê o arquivo real, não uma cópia |
| Perguntas sobre o projeto | **Chat** | Não precisa de ferramentas, menos tokens |
| Análise de dados / planilhas | **Cowork** | Skills especializados |

**Regra geral:**
- 🔧 **Claude Code** → quando precisa **tocar em arquivos** ou **rodar comandos**
- 💬 **Chat** → quando é só **pergunta, explicação ou texto**

### Sempre fazer
- ✅ Testar local antes de qualquer push
- ✅ Hard refresh (`Ctrl+Shift+R`) ao testar — evita ver versão em cache
- ✅ Verificar o indicador 🟢 Ao vivo após atualizar
- ✅ Um commit por alteração com descrição clara do que foi feito
- ✅ Testar no celular também antes de publicar (redimensione o browser para ≤768px ou abra direto no celular — é o mesmo `index.html`)
- ✅ Validar JavaScript com `node --check` após qualquer alteração no código. Extrair o bloco script para um arquivo `.js` temporário e rodar: `node --check arquivo.js`
- ✅ Trabalhar sempre na sua branch pessoal (`marcela` ou `joao`), nunca direto na `main`

### Nunca fazer
- ❌ Editar direto no GitHub pelo browser (vai direto para produção sem teste)
- ❌ Dar push direto na branch `main` — mudanças entram via Pull Request revisado pelo owner (ver seção 4)
- ❌ Confiar só no botão "Atualizar" do dashboard — ele re-executa o JS em cache, não baixa HTML novo
- ❌ Publicar sem testar no celular também
- ❌ Push sem mensagem de commit descritiva
- ❌ Fazer `git push --force` na `main` (a branch protection bloqueia isso de qualquer forma — ver seção 6)

### Boas práticas adicionais recomendadas
- 📌 **Sempre descreva o commit em português** com o que foi alterado e por quê
- 📌 **Não há mais API Key hardcoded no código** (removida Jul/2026 — dados vêm do xlsx local). Se for trocar o nome do arquivo xlsx, atualizar a constante `LOCAL_XLSX_FILE` em `index.html` **e** o `CLAUDE.md` (seção "Fontes de dados")
- 📌 **Se o indicador mostrar 🔴**, verifique: (a) o arquivo xlsx está mesmo no repo com o nome exato esperado, (b) abra o DevTools (F12) → aba Network → confira se o fetch do `.xlsx` retornou 200
- 📌 **Qualquer mudança na estrutura das colunas do xlsx** (inserir/mover coluna) exige atualização do código de parse (`_parseWBS` — ver seção 7) — o código faz uma checagem de sanidade no console (`console.warn`), mas não impede dado errado silenciosamente se o aviso for ignorado
- 📌 **A `main` tem branch protection ativa de verdade** (1 aprovação obrigatória, Code Owner = `@PMO-creator`) — todo PR precisa de revisão antes do merge

---

## 3. Configuração inicial

### Pré-requisitos (instalar uma vez)
```
1. Git          → https://git-scm.com
2. Python 3.x   → https://python.org  (para servidor local)
3. Node.js      → https://nodejs.org  (para validar JS com node --check)
4. Claude Code  → https://claude.ai/code  (recomendado para edições)
```

Verificar instalações no terminal:
```bash
git --version
python --version
node --version
```

### Estrutura de pastas de trabalho

O projeto usa **uma única pasta** — edição, teste e publicação acontecem no mesmo lugar:

```
GitHub\
  mup-dashboard\                            ← PASTA ÚNICA — edita, testa e publica daqui
    index.html                              → Dashboard único, desktop e mobile (~430 KB)
    MUP _ EAP - CRONOGRAMA_16Jul.xlsx        → Cronograma de origem (fonte de dados atual)
    SERVE_DASHBOARD.bat                     → Servidor local (duplo-clique para preview)
    ONBOARDING.md                            ← este arquivo (leia antes de trabalhar)
    CLAUDE.md
    .github/CODEOWNERS                       → define quem revisa PRs
    .claude/                                 → configs locais do Claude Code (não afeta o dashboard)
```

> ⚠️ **Não usar OneDrive** para esta pasta — o OneDrive corrompe a pasta `.git` ao sincronizar arquivos internos do git. Use um caminho local puro, ex: `C:\Users\<usuário>\GitHub\mup-dashboard`.

### Clonar o repositório (primeira vez)
```bash
git clone https://github.com/PMO-creator/mup-dashboard
```

Você terá a pasta `mup-dashboard/` com tudo que precisa. O servidor local (`SERVE_DASHBOARD.bat`) cumpre o papel de ambiente de teste dentro da própria pasta.

### Configurar acesso ao GitHub
O owner (`@PMO-creator`) precisa te adicionar como colaborador:
- `github.com/PMO-creator/mup-dashboard` → Settings → Collaborators → Add people

> No momento, `marcela` e `joao` existem como **branches** no repositório, mas as pessoas ainda não foram adicionadas como colaboradoras — sem acesso de escrita, não conseguem dar push nas próprias branches. Confirmar/pedir o convite antes de tentar clonar+editar.

### Branches do projeto
- `main` → produção, protegida (1 aprovação obrigatória via PR, Code Owner = `@PMO-creator`)
- `marcela`, `joao` → branches pessoais de trabalho
- Não existe branch `dev` neste repo (diferente do MAZ)

---

## 4. Fluxo de trabalho — do teste ao ar

```
sua branch (marcela/joao)  →  edita + testa localmente  →  commit/push na sua branch
        →  Pull Request  →  revisão do owner  →  merge na main  →  GitHub Pages (quando ativado)
```

### Passo 0 — Sincronizar antes de começar

```bash
# Na pasta mup-dashboard:
git switch <sua-branch>      # marcela ou joao
git pull
git merge main                # traz o que entrou na main desde a última vez
```

### Passo 1 — Subir o servidor local

Na pasta `mup-dashboard`, rodar:
```
SERVE_DASHBOARD.bat
```
→ Abre automaticamente `http://localhost:8000`

Ou manualmente:
```bash
python -m http.server 8000
```

### Passo 2 — Fazer as alterações

Edite `index.html` com **Claude Code** (abrir o Claude Code na pasta `mup-dashboard`).

### Passo 3 — Testar localmente

```
Desktop → http://localhost:8000/index.html
Mobile  → redimensione o browser para ≤768px, ou abra o mesmo link no celular (ver seção 5)
```

- Fazer **hard refresh** (`Ctrl+Shift+R`) a cada alteração
- Verificar o indicador 🟢 Ao vivo
- Testar as funcionalidades afetadas pela mudança

### Passo 4 — Subir a branch e abrir o Pull Request

```bash
git add index.html
git commit -m "Descrição clara do que foi alterado"
git push -u origin <sua-branch>
```

Depois, abrir o Pull Request pedindo revisão do owner:
```bash
gh pr create --base main --title "Título curto" --body "O que mudou e por quê"
```
Ou pela interface do GitHub (o próprio `git push` mostra o link direto no terminal).

> ⚠️ A `main` tem **branch protection ativa**: todo PR precisa de **1 aprovação** de um Code Owner antes de poder ser mergeado — o GitHub bloqueia o botão "Merge" até isso acontecer. O autor do PR **não pode aprovar o próprio PR**. Se você for o único colaborador com acesso (situação comum no início do projeto), avise o owner — ele pode ajustar a proteção (`enforce_admins`) para conseguir mergear PRs próprios sem travar, mantendo a exigência de revisão para PRs de terceiros.

### Passo 5 — Publicação

Só depois do merge na `main` é que a mudança vai para o GitHub Pages **quando a Page estiver ativada** (hoje ainda não está — ver seção 1). Aguardar **~2 minutos** após o merge e fazer hard refresh na URL pública para confirmar.

---

## 5. Testar no celular pela rede local

Celular e computador precisam estar na **mesma rede Wi-Fi**.

### Encontrar o IP do computador
No terminal Windows:
```
ipconfig
```
Procurar por **"Endereço IPv4"**:
```
Adaptador de Rede Wi-Fi:
   Endereço IPv4 . . . : 192.168.1.105   ← esse é o IP
```

### Acessar no celular
No browser do celular digitar:
```
http://192.168.1.105:8000/index.html
```
A página é a mesma — o layout se adapta automaticamente ao tamanho da tela (responsivo).

> 💡 O IP pode mudar quando a rede muda. Sempre verifique com `ipconfig` antes de testar.

---

## 6. Reverter para uma versão anterior

### Ver o histórico de commits
```bash
git log --oneline
```

### Opção A — Reverter um commit específico (recomendado)
Cria um novo commit que desfaz as mudanças. Histórico fica intacto.
```bash
git revert <hash-do-commit>
git push
```
> Se o commit já estiver na `main`, seguir o fluxo normal de branch + PR (seção 4) para reverter — a `main` protegida não aceita push direto do revert também.

### Opção B — Ver como o arquivo estava em uma versão antiga
```bash
git show <hash>:index.html > versao_antiga.html
```
Abre `versao_antiga.html` para comparar ou copiar trechos.

### Opção C — Voltar o repositório inteiro para uma versão (cuidado)
⚠️ Apaga tudo que foi feito depois desse commit.
```bash
git reset --hard <hash>
git push --force
```
> Só usar em último caso, e só numa **branch pessoal** — a `main` bloqueia force-push no nível do GitHub (branch protection: `allow_force_pushes: false`), então esse comando nem funciona lá. Avisar o responsável antes de usar em qualquer branch.

### Pelo GitHub (interface visual)
1. Acessar `github.com/PMO-creator/mup-dashboard`
2. Clicar na aba **"Commits"**
3. Encontrar o commit desejado → clicar em **"<>"** (Browse files)
4. Baixar o arquivo da versão antiga manualmente

---

## 7. Referências técnicas

### URLs
| Recurso | URL |
|---|---|
| Dashboard público | `https://pmo-creator.github.io/mup-dashboard/` (GitHub Pages ainda não ativado) |
| Repositório GitHub | `https://github.com/PMO-creator/mup-dashboard` |
| Teste local | `http://localhost:8000/index.html` (mesma URL para desktop e mobile — responsivo) |

### Fonte de dados (fase atual — xlsx local)

| Arquivo | Aba | Observação |
|---|---|---|
| `MUP _ EAP - CRONOGRAMA_16Jul.xlsx` | `master data` | Cabeçalho real na **linha 2** (linha 1 tem só "BASELINE") |

Lido inteiramente no browser via **SheetJS** (`xlsx@0.18.5`, CDN com SRI) — sem API key, sem backend. Funções relevantes em `index.html`: `_loadLocalWorkbook()`, `fetchSheet()`, `fetchSheetColors()` (lê cor de célula da coluna E para diferenciar linha-tarefa de linha-cabeçalho).

> ⚠️ O repositório é **público** — qualquer pessoa consegue baixar o xlsx com dados reais do cronograma. Se isso não for aceitável, tornar o repo privado ou trocar por uma versão com dados fictícios.

**Futuro — migração para Google Sheets API:** mesma estrutura de abas/colunas se mantém. IDs preservados no código (hoje sem uso, os parâmetros `spreadsheetId` são ignorados pelas funções):

| Planilha | ID | Aba |
|---|---|---|
| Cronograma | `17nttJ_ShqWztvDWH3l59iNqboLqkviZs3_PM5J3ihdA` | `master data` |
| Requisições | `1azrdS4OGO-CWD1ods69i8iZJcwq4oyISdT2n_tu1uJM` | `Planilha de Status de Compras Prod` |

> A `SHEETS_API_KEY` foi **removida** do código (Jul/2026 — era código morto, credencial exposta sem uso num repo público). Só volta a ser necessária nessa migração futura.

### Colunas do cronograma (`_parseWBS`, índices 0-based)

| Coluna | Índice | Campo |
|---|---|---|
| B | 1 | Eixo |
| C | 2 | Grupo |
| D | 3 | Marco \ Entrega |
| E | 4 | Tarefa |
| F | 5 | Fornecedor |
| G | 6 | Responsável |
| H | 7 | Prioridade (existe na planilha, ainda não é lida pelo código) |
| I | 8 | Status |
| J | 9 | Data início |
| K | 10 | Data fim |
| L | 11 | Duração (não lida) |
| M | 12 | Predecessores (não lida) |
| O | 14 | Progresso |
| P | 15 | Encaminhamentos |
| R em diante | 17+ | Áreas (Foyer, Área 0...Áreas Comuns) — detectadas **dinamicamente** pelo texto "FOYER" no header, não por índice fixo (`_detectAreaCols`) |

> ⚠️ **Diferente do MAZ:** a coluna H (Prioridade) empurra Status/Início/Fim uma posição à frente — I/J/K em vez de H/I/J. Se `_parseWBS` for mexido de novo, conferir sempre contra o header real (`grep -n "function _parseWBS" index.html`) antes de confiar nesta tabela — o código já tem um `console.warn` de checagem de sanidade que avisa se o header não bater.

### Colunas das Requisições (`_parseREQS`)

> 🔶 **Requisições está em on hold** — a aba/feature segue no código (herdada do MAZ), mas fica dormente até existir uma fonte de dados de compras real para o MUP. A tabela abaixo é referência para quando essa fonte for definida.

| Coluna | Índice | Campo |
|---|---|---|
| A | 0 | Nº Requisição |
| B | 1 | Comprador |
| D | 3 | Prioridade ← USAR APENAS ESTA |
| E | 4 | Tipo |
| F | 5 | Descrição |
| G | 6 | Status |
| H | 7 | Fornecedor |
| I | 8 | Observações |
| K | 10 | Advogado |
| M | 12 | Data Limite |
| N | 13 | Chegada do Material |
| P | 15 | Finalização do Serviço |

> ⚠️ Prioridade: usar APENAS coluna D (índice 3). Coluna B gera falsos positivos.

### Regras de auto-status

| Condição | Status resultante |
|---|---|
| Subtask sem status + sem data | `Definir datas` |
| `A iniciar` + sem data de início | `Definir datas` |
| Grupo/eixo sem status definido | `Definir datas` |
| Qualquer status + data fim já passou | `Atrasado` |
| Qualquer status + data fim nos próximos 7 dias | `Risco de atraso` |
| `Feito`, `Cancelado`, `Cancelado/Congelado` | Nunca muda |

**Ranking de pior status (rollup):**
```
Atrasado(0) > Risco de atraso(1) > Em andamento(2) > Definir datas(3) > A iniciar(4) > Feito(5) > Cancelado/Congelado(6)
```

**Cálculo automático de datas:**
- Marco: início = mínimo das subtasks · fim = máximo das subtasks
- Grupo: início = mínimo dos marcos · fim = máximo dos marcos
- Marcos sem subtasks mantêm as datas da planilha

Rollup: **marco = pior status das tarefas / grupo = pior status dos marcos / eixo = pior status dos grupos**

### Auto-refresh
15 minutos — constante `AUTO_REFRESH_MS`. Com a fonte de dados sendo um xlsx local estático, o auto-refresh hoje só relê o **mesmo arquivo** — só reflete algo novo se alguém trocar o arquivo no repo e a página recarregar depois disso.

### Filtros do Dashboard

Os filtros globais (Status, Eixo, Data e Responsável) ficam na barra superior de `index.html`, que se adapta automaticamente para telas menores. Todos alimentam a função `applyFilter()` que re-renderiza a árvore EAP e o Gantt.

#### Filtro de Fornecedor — Aba Requisições

> 🔶 Dormente junto com a aba Requisições (ver "on hold" acima) — código intacto, sem dado pra filtrar hoje.

| Função JS | Responsabilidade |
|---|---|
| `buildReqFilters()` | Monta os filtros de status, comprador, prioridade e fornecedor |
| `applyReqFilter()` | Aplica todos os filtros e re-renderiza a lista de requisições |

#### Aba Gantt Diretoria

Aba `📊 Gantt Diretoria`, visualmente idêntica à aba Gantt, mas com regra de status diferente: usa o status **exatamente como preenchido na planilha**, sem os overrides automáticos por data que a aba Gantt aplica.

**Diferença de regra de negócio:**
| Regra | Aba Gantt | Aba Gantt Diretoria |
|---|---|---|
| Status vazio + sem data início | `Definir datas` | `Definir datas` |
| Status vazio + com data início | `A iniciar` | `A iniciar` |
| `A iniciar` + data início no passado | override → `Risco de atraso` | **mantém** `A iniciar` |
| Data fim no passado | override → `Atrasado` | **mantém** status da planilha |
| Data fim nos próximos 7 dias | override → `Risco de atraso` | **mantém** status da planilha |
| Rollup marco/grupo/eixo (pior status) | sim | sim (mantido) |
| `Feito`/`Cancelado`/`Cancelado/Congelado` nunca sobrescreve | sim | sim (mantido) |

**Fonte de dados:** `WBS_DIR` (deep copy de `WBS`) + `preprocessStatusesDiretoria()`.

**Implementação:** duplicação independente de toda a cadeia de funções do Gantt, sufixadas `Dir`, para não alterar a aba Gantt original:

| Função original | Função Dir (Gantt Diretoria) |
|---|---|
| `renderGanttSection()` | `renderGanttSectionDir()` |
| `renderGanttForEixo(gi)` | `renderGanttForEixoDir(gi)` — usa `WBS_DIR`, `ganttModeDir` |
| `toggleGantt(gi)` | `toggleGanttDir(gi)` |
| `toggleGanttGroup(gi,ri)` | `toggleGanttGroupDir(gi,ri)` |
| `toggleGanttMarco(gi,ri,ti)` | `toggleGanttMarcoDir(gi,ri,ti)` |
| `expandAllGantt()` | `expandAllGanttDir()` |
| `expandAllTarefasGantt()` | `expandAllTarefasGanttDir()` |
| `collapseAllGantt()` | `collapseAllGanttDir()` |
| `setGanttMode(mode)` | `setGanttModeDir(mode)` |

IDs de DOM têm o sufixo/infixo `dir`: `gantt-dir-container`, `gbody-dir-{gi}`, `gchev-dir-{gi}`, `gantt-svg-dir-{gi}`, `btn-mode-dir-mensal`, `btn-mode-dir-semanal`.

Os mesmos filtros globais (responsável, status, período) e a mesma trava de senha da aba Status Report (`sessionStorage` `eap_unlocked`) se aplicam à Gantt Diretoria — `switchTab()` exige desbloqueio para `name==='eap'||name==='eap-dir'`.

> ⚠️ A aba Gantt original (`WBS`, `preprocessStatuses()`, funções sem sufixo `Dir`) não foi alterada.

#### Aba Áreas — Gantt e Export PDF

A aba Áreas exibe o Gantt por área física do museu. As colunas de área são detectadas dinamicamente a partir do header da planilha (ver `_detectAreaCols` acima).

| Função JS | Responsabilidade |
|---|---|
| `_detectAreaCols(rows)` | Detecta os índices das colunas de área usando a coluna FOYER como âncora. Chamada no init após `loadSheetsData`. |
| `_buildGanttSVGForExport(ai, fDS, fDE, viewMode, fmt)` | Gera SVG do Gantt de uma área para export PDF. Parâmetro `fmt` (A1/A2/A3/A4) controla escala. |
| `pdfSelAll()` | Seleciona todas as áreas no wizard de Export PDF. |
| `pdfSelClear()` | Limpa a seleção de áreas no wizard de Export PDF. |

**Multi-select no Export PDF de Áreas:** o wizard permite selecionar múltiplas áreas. O PDF gerado contém todas em sequência. Arquivo nomeado `Gantt_multiplas_areas_N_[período].pdf` quando > 1 área selecionada.

#### Exportar Pauta N2 como HTML navegável (nome da função mantido por compatibilidade)

Botão **📊 Exportar PPT** aparece junto com o FAB da Pauta N2 quando há marcos selecionados. Apesar do nome do botão/função, gera um `.html` navegável (não `.pptx`) — não há dependência de biblioteca externa de PPT no código atual.

| Elemento | Detalhe |
|---|---|
| FAB | `id="n2-ppt-fab"` — `bottom:200px right:28px`, fundo `#065F46` |
| Visibilidade | Mesmo critério do FAB principal: `n > 0 && onEap && !locked` |
| Função JS | `exportN2PPT()` — lê seleção via `loadN2()`, aplica filtros ativos, gera HTML via `_buildN2HTMLDoc()` |
| Arquivo gerado | `Pauta_N2_YYYY-MM-DD.html` |

---

#### Filtro de Responsável (`index.html`, responsivo)

Localização no HTML: `<div class="ms-resp-wrap">` na barra de filtros, entre o filtro de Eixos e o botão "Limpar Filtros". O dropdown se adapta visualmente em telas pequenas, mas usa o mesmo dropdown multi-select e as mesmas funções em qualquer tamanho de tela.

| Função JS | Responsabilidade |
|---|---|
| `buildRespFilter()` | Lê os nomes únicos de responsável dos dados e monta o dropdown multi-select |
| `grupoHasResp(grupo, resp)` | Retorna `true` se o grupo ou qualquer uma de suas tarefas pertence ao responsável |
| `getFilters()` | Retorna o objeto de filtros ativos, incluindo o campo `resp` (array de nomes selecionados) |
| `applyFilter()` | Detecta "nenhum marcado" via **contagem DOM** (não via `f.resp`), exibe `#gantt-resp-msg` e oculta linhas sem correspondência |

**Label do dropdown:**
- Todos marcados → `"Todos os responsáveis"`
- 1–2 marcados → lista os nomes
- 3+ marcados → `"X selecionados"`
- Nenhum marcado → `"Nenhum selecionado"`

**Lógica de filtragem (hierarquia):**
- Marco com o responsável → mostra o marco **com todas** as suas tarefas
- Marco sem o responsável, mas com alguma tarefa do responsável → mostra o marco **só com as tarefas** do responsável
- Grupo sem nenhum marco/tarefa do responsável → desaparece
- Eixo sem nenhum grupo com o responsável → desaparece
- No Gantt: linhas sem o responsável são **ocultadas** (não esmaecidas)

---

## 8. Skills disponíveis

### code_audit — Auditor de código (skill global do Claude Code)

Analisa o `git diff` atual (ou o `index.html` completo, sob pedido) em busca de problemas de segurança, arquitetura, qualidade de código e dependências externas. É uma **skill global** (`~/.claude/skills/code_audit`, não uma pasta dentro deste repo) — acionada digitando em linguagem natural algo como `"audita o que mudou"` antes de um commit/push.

> ⚠️ Há um hook global (`~/.claude/settings.json`) que tenta forçar essa pergunta automaticamente antes de qualquer `git commit`/`git push` via Claude Code. Ele é do tipo `"prompt"` e **não recebe confirmação do chat** — reavalia o comando puro a cada tentativa e pode bloquear mesmo depois de você já ter confirmado. Se isso acontecer, rode o `git commit`/`git push` direto no terminal (fora do Claude Code) — não é um bypass de segurança, é uma limitação conhecida desse tipo de hook.

> Não existem mais as skills `doc-sync` nem uma pasta `code-audit/` dentro do repositório — eram específicas do MAZ e foram removidas do MUP (ver `CLAUDE.md`).

---

## 9. Armadilhas técnicas conhecidas

| Armadilha | Como evitar |
|---|---|
| Dashboard branco sem erro no console | Verificar: (a) null bytes no HTML, (b) palavra `function` ausente em declaração JS, (c) JS truncado sem `</script>`, (d) template literals aninhados |
| Template literals aninhados | Nunca usar crase dentro de `${}` dentro de outro crase — `node --check` passa mas browser quebra |
| JS truncado | Verificar se `</script>` existe no final do arquivo antes de editar |
| Prioridade REQS (dormente) | Usar APENAS coluna D (índice 3) — coluna B gera falsos positivos. Relevante só quando Requisições sair do on hold |
| `node --check` no Node v22 | Não aceita `.html` — extrair bloco script para arquivo `.js` temporário |
| Edit tool do Claude Code falha em `index.html` | Arquivo contém backticks (template literals JS) e é grande. Usar **Python `str.replace()` via Bash** (não PowerShell) — ler o arquivo, normalizar CRLF→LF antes de comparar, aplicar substituição, converter de volta pra CRLF antes de gravar (ver `CLAUDE.md` do projeto) |
| Colunas do cronograma MUP ≠ MAZ | A coluna H (Prioridade) desloca Status/Início/Fim de H/I/J (MAZ) para I/J/K (MUP). Ver seção 7 — `_parseWBS` já tem checagem de sanidade (`console.warn`) se o header não bater |
| SheetJS: `dateNF` no `sheet_to_json` | Não formata como esperado com `raw:false` (datas saem tipo `4/1/26`, que `_fmtDate` não reconhece). Usar `raw:true` + `cellDates:true` e converter objetos `Date` para ISO manualmente (ver `fetchSheet`/`_xlsxDateToISO` em `index.html`) |
| Branch protection na `main` | 1 aprovação obrigatória via PR (Code Owner `@PMO-creator`), autor não pode aprovar o próprio PR. Se você for o único colaborador, o PR fica travado até o owner ajustar `enforce_admins` ou adicionar outro revisor — ver seção 4 |
| `exportN2PPT()` sem pptxgenjs | Gera `.html`, não `.pptx` — não há dependência de biblioteca de PPT no código atual (nome da função mantido por compatibilidade histórica) |
| `preprocessStatusesDiretoria()` vs `preprocessStatuses()` | São duas funções separadas que processam fontes separadas (`WBS_DIR` vs `WBS`). Editar uma sem editar a outra causa divergência silenciosa entre a aba Status Report normal e a aba Diretoria. Ambas são chamadas em sequência no load — nunca remover uma achando que é redundante |
| `WBS_DIR` é deep copy, não referência | `WBS_DIR.length=0; JSON.parse(JSON.stringify(newWBS)).forEach(e=>WBS_DIR.push(e));` — cópia profunda tirada no momento do load. Edições de status manual numa árvore não propagam para a outra. Intencional (permite status divergente para comparação) |
| `window.print()` no Comparativo | O botão "Exportar PDF" do modal Comparativo (`comp-btn-pdf`) chama `window.print()` puro — depende do diálogo de impressão do browser, sem CSS `@media print` dedicado |
| Nome da Área 0 | Sempre **"Pinguela"** (confirmado no header do xlsx do MUP, coluna S) — evitar variações como "Pingueira" |

---

## 10. Feature: Pauta N2

Permite selecionar tarefas individuais dentro de marcos para levar à reunião de diretoria (N2), filtrar só as selecionadas e visualizar a pauta.

### Como funciona (UX)
1. Na aba **Status Report**, expandir um grupo e clicar em "▸ X marcos"
2. Expandir um marco — cada **task card** exibe um **checkbox** antes do badge de status
3. Marcar o checkbox → badge verde **N2** aparece no tile do marco pai; botão flutuante **📋 Pauta N2 · N** mostra a contagem de tarefas
4. Clicar **📋 Pauta N2 · N** → filtra a árvore mostrando só os marcos que têm tarefas selecionadas
5. Clicar **✕ Ver todos · N** → volta a exibir tudo
6. Botão **✕ Limpar N2** (abaixo do FAB) → limpa todas as seleções

### Comportamento de persistência
- Seleções ficam em `localStorage`, chave semanal `n2_pauta_YYYY-Wnn` (gerada por `getN2Key()`, usa `getISOWeek()`) — cada semana ISO tem seu próprio balde de seleções
- Sobrevive a hard refresh e nova carga da página, mas naturalmente "reseta" ao virar a semana ISO (chave muda)
- Persistido via `loadN2()` / `saveN2()`, que fazem `JSON.parse`/`JSON.stringify` no `localStorage`

### Estrutura técnica (index.html)

| Elemento | Detalhe |
|---|---|
| Funções JS | `toggleN2Task`, `toggleN2Marco`, `toggleN2Group`, `updateN2Fab`, `clearN2Selection`, `applyN2Filter`, `toggleN2Filter`, `initN2Checkboxes`, `initN2FromURL`, `publishN2Pauta`, `unlockN2Edit`, `_n2Hash`, `loadN2`, `saveN2`, `getN2Key` |
| Armazenamento | `localStorage.getItem(getN2Key())` — chave semanal, não in-memory |
| Estado view-only | `var _n2ViewMode=false` — true quando aberto via link publicado |
| Estado desbloqueado | `var _n2Unlocked=false` — true após PIN correto |
| ID dos checkboxes | `n2c-{gi}-{mi}-{ti}-{si}` — **4 partes** (eixo, grupo, marco, subtask) |
| ID dos tiles (marco) | `mband-{gi}-{mi}-{ti}` — referência do marco pai (3 partes) |
| ID das tasks | `taskcard-{gi}-{mi}-{ti}-{si}` — card individual da tarefa |
| ID dos badges | `n2badge-{gi}-{mi}-{ti}` — no tile do marco, `display:none` por padrão |
| Chave de seleção | `gi:mi:ti:si` (string com **4 partes** separadas por `:`) |
| Onde fica no template | `taskCard(t, n2Id)` — dentro de cada task card, ANTES do badge de status |
| CSS badge | `.n2-badge` — fundo `#1C2C0A`, fonte `#B8E85C`, 18px |
| FAB principal | `id="n2-fab"` — fixo `bottom:82px right:28px`, só visível na aba EAP |
| FAB limpar | `id="n2-clear-fab"` — só visível quando há seleções e não está em view-only |
| FAB publicar | `id="n2-publish-fab"` — `bottom:140px right:28px`, fundo `#1A3A8A`, só visível quando há seleções e não está em view-only |
| FAB desbloquear | `id="n2-lock-fab"` — `bottom:24px right:28px`, fundo `#5C3A8A`, só visível em view-only |
| FAB exportar HTML | `id="n2-ppt-fab"` — `bottom:200px right:28px`, fundo `#065F46`. Visível quando `n > 0 && onEap && !locked`. Chama `exportN2PPT()`. |

### Exportar Pauta N2 como HTML navegável (nome da função mantido por compatibilidade)

Gera `.html` com as tarefas selecionadas.

| Função | Responsabilidade |
|---|---|
| `exportN2PPT()` | Lê seleção via `loadN2()` (IDs no formato `gi:mi:ti:si` — 4 partes), agrupa por marco/eixo, gera HTML via `_buildN2HTMLDoc()`. Arquivo: `Pauta_N2_YYYY-MM-DD.html`. |

### Compartilhamento via URL+PIN

Formato da URL gerada:
```
https://pmo-creator.github.io/mup-dashboard/index.html?n2=ID1,ID2,...&ph=HASH
```
> ⚠️ IDs no formato `gi:mi:ti:si` (4 partes, nível tarefa). Exemplo: `0:1:2:0` = eixo 0, grupo 1, marco 2, subtask 0.

| Função | Responsabilidade |
|---|---|
| `initN2FromURL()` | Chamada no init. Lê `?n2=` da URL; se presente, ativa `_n2ViewMode=true` e carrega IDs. |
| `publishN2Pauta()` | Pede PIN via `prompt()`, gera URL com `_n2Hash(pin)` no parâmetro `ph=`, abre modal de cópia. |
| `unlockN2Edit()` | Pede PIN, compara `_n2Hash(pin)` com `?ph=` da URL. Se correto, seta `_n2Unlocked=true` e habilita checkboxes. |
| `_n2Hash(s)` | Hash djb2-like simples. Não é criptografia forte — serve para ofuscar o PIN na URL. |

### Armadilha específica do N2
- `initN2Checkboxes()` é chamado no final de `renderTree()` — re-checa `loadN2()` e restaura estado visual; também aplica `disabled` quando `_n2ViewMode && !_n2Unlocked`
- Ao desligar o filtro N2 ("Ver todos"), `toggleN2Filter` chama `applyFilter()` (não apenas mostra/oculta DOM) para garantir que accordions e dropdown de visualização funcionem normalmente
- O `n2-badge` (no tile do **marco**) começa com `display:none` — é setado via `toggleN2Task()` quando qualquer tarefa do marco entra na seleção salva via `saveN2()`
- `applyN2Filter()` filtra `taskcard-*` (mostra/oculta tasks) **e** `mband-*` (mostra/oculta marcos que têm ao menos 1 tarefa selecionada)
- Nunca usar `ev.preventDefault()` no handler do checkbox — impede o checkmark visual no browser

---

## 11. Feature: Aba Diretoria

Versão da aba "Status Report" (Gantt/EAP/N2) espelhada para uso da diretoria, com fonte de dados própria e um modal de comparação de divergências de status.

### Como funciona (UX)
1. Nova aba de topo, ao lado de "Status Report"
2. Contém sua própria árvore EAP, Gantt e Pauta N2 — visualmente idêntica à aba original, mas independente
3. Botão **📊 Comparar Status** (`openComparativoModal()`) abre um modal listando tarefas onde o status da aba Diretoria diverge do status da aba Status Report original
4. Dentro do modal, botão **📄 Exportar PDF** aciona `window.print()` do browser

### Estrutura técnica (index.html)

| Elemento | Detalhe |
|---|---|
| Fonte de dados | `WBS_DIR` (array separado de `WBS`) — ver armadilha na seção 9 sobre deep copy |
| Preprocessamento | `preprocessStatusesDiretoria()` — lógica de status espelhada de `preprocessStatuses()`, mas roda sobre `WBS_DIR` |
| Render | `renderKPIs_Dir`, `renderTree_Dir`, `buildCommentPanel_Dir`, `toggleGroup_Dir`, `toggleComment_Dir`, `toggleMarco_Dir`, `collapseAll_Dir`, `expandAllEixosEAP_Dir`, `expandAllMarcosEAP_Dir`, `expandAllTarefasEAP_Dir` |
| Gantt | `renderGanttSectionDir`, `renderGanttForEixoDir`, `toggleGanttDir`, `setGanttModeDir`, `expandAllGanttDir`, `collapseAllGanttDir` — usa `ganttModeDir` (estado separado de `ganttMode`) |
| N2 (Pauta) | `loadN2_Dir`, `saveN2_Dir`, `toggleN2Task_Dir`, `updateN2Fab_Dir`, `publishN2Pauta_Dir`, `unlockN2Edit_Dir`, `exportN2PPT_Dir` — todos com sufixo `_Dir`, estado isolado do N2 original |
| Modal Comparativo | `buildComparativoModal()` monta a lista de divergências comparando `WBS` × `WBS_DIR` por `eixo/grupo/marco/tarefa`; `openComparativoModal()` / `closeComparativoModal()` controlam o modal |
| Export PDF do Comparativo | Botão `.comp-btn-pdf` → `window.print()` — sem geração programática de PDF |

### Armadilhas específicas da Diretoria
- `WBS` e `WBS_DIR` são recarregados juntos no load (`fetchAll()`), mas depois disso vivem desacoplados — status editado manualmente numa árvore não reflete na outra
- Sempre que uma função nova for adicionada à aba Status Report original, avaliar se precisa de uma equivalente `_Dir`/`Dir` — não existe herança automática entre as duas abas
- O Comparativo (`buildComparativoModal`) depende de `eixo/grupo/marco` baterem como chave de correspondência entre `WBS` e `WBS_DIR` — renomear um eixo/grupo/marco em uma árvore sem atualizar a outra faz a tarefa "sumir" da comparação em vez de aparecer como divergência

---

*Guia adaptado para o MUP em 17/Jul/2026 — Dashboard MUP · IDG PMO*
