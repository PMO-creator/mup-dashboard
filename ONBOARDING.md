# Guia de Onboarding — Dashboard MAZ 2026
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
8. [Skills disponíveis no repositório](#8-skills-disponíveis-no-repositório)
9. [Armadilhas técnicas conhecidas](#9-armadilhas-técnicas-conhecidas)

---

## 1. O que é este projeto

Dashboard interativo do **Museu das Amazônias 2026 (MAZ ELD)** — acompanhamento de cronograma, status report e requisições de compras.

### Arquitetura
- **Um único arquivo HTML** é o projeto inteiro:
  - `index.html` → versão desktop e mobile (responsiva, ≤768px), inclui as abas Diretoria (Gantt Diretoria, Status Report Diretoria, Comparativo de Status)
  - `mobile.html` foi removido em 01/07/2026 — a pedido do Sergio, o mobile dedicado saiu de linha e agora todo acesso (celular ou desktop) usa o `index.html`
- **Dados ao vivo** — buscados direto do Google Sheets via API Key no browser, sem backend
- **Publicado** em `https://pmo-creator.github.io/maz-dashboard/` via GitHub Pages
- **Dependências externas (CDN):**
  - Chart.js 4.5.1 — gráficos de KPI
  - jsPDF 2.5.1 + svg2pdf.js 2.2.3 — export PDF

### Como os dados chegam
```
Google Sheets (Cronograma + REQS)
        ↓  API Key (sem OAuth)
    Browser do usuário
        ↓  renderiza
    Dashboard (index.html — desktop e mobile no mesmo arquivo)
```

### Indicador de status (canto superior direito)
| Indicador | Significado |
|---|---|
| 🟢 Ao vivo · HH:MM | Tudo funcionando, dados frescos |
| 🟡 Cronograma OK · REQ erro | Cronograma OK mas REQS falhou |
| 🔴 Erro — dados locais | Fetch falhou, mostrando dados antigos |

---

## 2. Boas práticas de desenvolvimento

### Ferramentas certas para cada tarefa

| Tipo de tarefa | Melhor ferramenta | Por quê |
|---|---|---|
| Editar arquivos HTML/CSS/JS | **Claude Code** | Acessa e edita arquivos diretamente |
| Git commit / push | **Claude Code** | Roda bash/git |
| Debug de código em arquivos | **Claude Code** | Lê o arquivo real, não uma cópia |
| Perguntas sobre o projeto | **Chat ou Cowork** | Não precisa de ferramentas, menos tokens |
| Explicações gerais de tecnologia | **Chat** | Puramente conversacional |
| Brainstorm / planejamento de features | **Chat ou Cowork** | Sem necessidade de arquivos |
| Criar documentos Word/PDF/PPT | **Cowork** | Skills especializados |
| Análise de dados / planilhas | **Cowork** | Skills de data analysis |

**Regra geral:**
- 🔧 **Claude Code** → quando precisa **tocar em arquivos** ou **rodar comandos**
- 💬 **Chat** → quando é só **pergunta, explicação ou texto**
- 🤝 **Cowork** → quando precisa de **skills especializados**

### Sempre fazer
- ✅ Testar local antes de qualquer push
- ✅ Hard refresh (`Ctrl+Shift+R`) ao testar — evita ver versão em cache
- ✅ Verificar o indicador 🟢 Ao vivo após atualizar
- ✅ Um commit por alteração com descrição clara do que foi feito
- ✅ Testar no celular também antes de publicar (redimensione o browser para ≤768px ou abra direto no celular — é o mesmo `index.html`)
- ✅ Validar JavaScript com `node --check` após qualquer alteração no código. Extrair o bloco script para um arquivo `.js` temporário e rodar: `node --check arquivo.js`
- ✅ Ao criar ou alterar o filtro de responsável, verificar os **3 estados**: (a) todos marcados → cronograma completo, (b) alguns marcados → mostra só os responsáveis selecionados, (c) nenhum marcado → conteúdo some e aparece mensagem de aviso verde

### Nunca fazer
- ❌ Editar direto no GitHub pelo browser (vai direto para produção sem teste)
- ❌ Confiar só no botão "Atualizar" do dashboard — ele re-executa o JS em cache, não baixa HTML novo
- ❌ Publicar sem testar no celular também
- ❌ Push sem mensagem de commit descritiva
- ❌ Fazer `git push --force` na branch main

### Boas práticas adicionais recomendadas
- 📌 **Sempre descreva o commit em português** com o que foi alterado e por quê (ex: `"Corrigir nome da aba REQS: 'Compras P' → 'Compras Prod'"`)
- 📌 **Nunca altere as constantes de API Key ou IDs de planilha** sem confirmar com o responsável do projeto
- 📌 **Se o indicador mostrar 🔴**, não é bug do dashboard — é problema de conectividade com o Sheets. Verifique compartilhamento da planilha e API Key
- 📌 **Qualquer mudança na estrutura das colunas das planilhas** exige atualização do código de parse (`_parseWBS` e `_parseREQS`)
- 📌 **Ao testar no celular**, use sempre o link do GitHub Pages (`pmo-creator.github.io/maz-dashboard`) ou o layout responsivo local — não existe mais redirecionamento, é o mesmo `index.html` em qualquer tela

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
  maz-dashboard\                 ← PASTA ÚNICA — edita, testa e publica daqui
    index.html                   → Dashboard único, desktop e mobile (~330 KB) — mobile.html removido em 01/Jul/2026
    SERVE_DASHBOARD.bat          → Servidor local (duplo-clique para preview)
    ONBOARDING.md                ← este arquivo (leia antes de trabalhar)
    CLAUDE.md
    doc-sync\                    ← skill doc-sync + contexto + snapshot
    Manual\                      ← documentação versionada (docx/pdf)
    00. Apoio\                   ← logos e banners
```

> A separação anterior em duas pastas foi eliminada em 26/Mai/2026. Tudo acontece diretamente em `maz-dashboard`; o servidor local (`SERVE_DASHBOARD.bat`) permite testar antes de commitar sem risco de publicação acidental.

### Clonar o repositório (primeira vez)
```bash
git clone https://github.com/PMO-creator/maz-dashboard
```

Você terá a pasta `maz-dashboard/` com tudo que precisa. Não há pasta de ambiente de teste separada — o servidor local (`SERVE_DASHBOARD.bat`) já cumpre esse papel dentro da própria pasta.

### Configurar acesso ao GitHub
O responsável anterior precisa te adicionar como colaborador:
- `github.com/PMO-creator/maz-dashboard` → Settings → Collaborators → Add people

---

## 4. Fluxo de trabalho — do teste ao ar

```
maz-dashboard  →  (edita + testa aqui mesmo)  →  commit/push  →  GitHub Pages
  (pasta única)                                                     (produção)
```

### Passo 0 — Puxar a versão mais recente

Antes de começar, garantir que a pasta local está atualizada:

```bash
# No terminal, dentro de maz-dashboard:
git pull
```

### Passo 1 — Subir o servidor local

Na pasta `maz-dashboard`, rodar:
```
SERVE_DASHBOARD.bat
```
→ Abre automaticamente `http://localhost:8000`

Ou manualmente:
```bash
python -m http.server 8000
```

### Passo 2 — Fazer as alterações

Edite `index.html` com **Claude Code** (abrir o Claude Code na pasta `maz-dashboard`).

### Passo 3 — Testar localmente

```
Desktop → http://localhost:8000/index.html
Mobile  → redimensione o browser para ≤768px, ou abra o mesmo link no celular
Celular → http://[SEU-IP]:8000/index.html  (ver seção 5)
```

- Fazer **hard refresh** (`Ctrl+Shift+R`) a cada alteração
- Verificar o indicador 🟢 Ao vivo
- Testar as funcionalidades afetadas pela mudança

### Passo 4 — Publicar (só quando aprovado)

```bash
# No terminal, dentro de maz-dashboard:
git add index.html
git commit -m "Descrição clara do que foi alterado"
git push
```

Aguardar **~2 minutos** → `https://pmo-creator.github.io/maz-dashboard/` atualizado.

Fazer **hard refresh** no GitHub Pages para confirmar (`Ctrl+Shift+R`).

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
A página é a mesma — o layout se adapta automaticamente ao tamanho da tela (responsivo). Não existe mais redirect para arquivo mobile separado.

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
> Só usar em último caso. Avisar o responsável antes.

### Pelo GitHub (interface visual)
1. Acessar `github.com/PMO-creator/maz-dashboard`
2. Clicar na aba **"Commits"**
3. Encontrar o commit desejado → clicar em **"<>"** (Browse files)
4. Baixar o arquivo da versão antiga manualmente

---

## 7. Referências técnicas

### URLs
| Recurso | URL |
|---|---|
| Dashboard público | `https://pmo-creator.github.io/maz-dashboard/` |
| Repositório GitHub | `https://github.com/PMO-creator/maz-dashboard` |
| Teste local | `http://localhost:8765/index.html` (mesma URL para desktop e mobile — responsivo) |

### Google Sheets
| Planilha | ID | Aba |
|---|---|---|
| Cronograma | `17nttJ_ShqWztvDWH3l59iNqboLqkviZs3_PM5J3ihdA` | `master data` |
| Requisições | `1azrdS4OGO-CWD1ods69i8iZJcwq4oyISdT2n_tu1uJM` | `Planilha de Status de Compras Prod` |

> ⚠️ Ambas as planilhas precisam estar com **"Qualquer pessoa com o link pode ver"** ativado. Sem isso o dashboard retorna erro 403/400.

### API Key Google Sheets
- Chave: `[solicitar ao responsável]`
- Projeto GCP: `maz-dashboard-495414`
- Restrição: `pmo-creator.github.io/*`
- Gerenciar em: `console.cloud.google.com` → APIs e serviços → Credenciais

### Colunas das planilhas (índices 0-based)

**Cronograma (`_parseWBS`):**
| Coluna | Índice | Campo |
|---|---|---|
| B | 1 | Eixo |
| C | 2 | Grupo |
| D | 3 | Marco |
| E | 4 | Tarefa |
| G | 6 | Responsável |
| H | 7 | Status |
| I | 8 | Data início |
| J | 9 | Data fim |

**Requisições (`_parseREQS`):**

> ⚠️ Atualizado Jul/2026 — tabela anterior estava com colunas deslocadas em relação ao código atual. Conferir sempre contra `_parseREQS` em `index.html` (`grep -n "function _parseREQS" index.html`) antes de confiar nesta tabela.

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

> Atualizado em commit 2ea1b35 — ranking e regras de default revisados.

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

**Cálculo automático de datas** (commit 8080c50):
- Marco: início = mínimo das subtasks · fim = máximo das subtasks
- Grupo: início = mínimo dos marcos · fim = máximo dos marcos
- Marcos sem subtasks mantêm as datas da planilha

Rollup: **marco = pior status das tarefas / grupo = pior status dos marcos / eixo = pior status dos grupos**

### Auto-refresh
15 minutos — constante `AUTO_REFRESH_MS` em ambos os HTMLs.

### Filtros do Dashboard

Os filtros globais (Status, Eixo, Data e Responsável) ficam na barra superior de `index.html`, que se adapta automaticamente para telas menores (chips/dropdowns responsivos). Todos alimentam a função `applyFilter()` que re-renderiza a árvore EAP e o Gantt.

#### Filtro de Fornecedor — Aba Requisições (commit b4a0b53)

Dropdown multi-select na barra de filtros da aba Requisições. Lê valores únicos da coluna G (Fornecedor). Itens sem fornecedor agrupados como "Sem Fornecedor". Botão verde **"Limpar filtros"** reseta status, fornecedor, comprador, prioridade e busca.

| Função JS | Responsabilidade |
|---|---|
| `buildReqFilters()` | Monta os filtros de status, comprador, prioridade e fornecedor |
| `applyReqFilter()` | Aplica todos os filtros e re-renderiza a lista de requisições |

#### Aba Gantt Diretoria (Jun/2026)

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

**Fonte de dados:** `WBS_DIR` (deep copy de `WBS`) + `preprocessStatusesDiretoria()` — ambos já existiam no projeto, criados para a aba "Status Report Diretoria". A aba Gantt Diretoria reaproveita essa mesma fonte.

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

A função `expandLevel(section, level)` ganhou o branch `section==='gantt-dir'` para o dropdown VISUALIZAÇÃO da nova aba.

Os mesmos filtros globais (responsável, status, período) e a mesma trava de senha da aba Status Report (`sessionStorage` `eap_unlocked`) se aplicam à Gantt Diretoria — `switchTab()` exige desbloqueio para `name==='eap'||name==='eap-dir'`.

> ⚠️ A aba Gantt original (`WBS`, `preprocessStatuses()`, funções sem sufixo `Dir`) não foi alterada.

#### Aba Áreas — Gantt e Export PDF (Jun/2026)

A aba Áreas exibe o Gantt por área física do museu. As colunas de área são detectadas dinamicamente a partir do header da planilha.

| Função JS | Responsabilidade |
|---|---|
| `_detectAreaCols(rows)` | Detecta os índices das colunas de área no Sheets usando a coluna FOYER como âncora. Chamada no init após `loadSheetsData`. Antes usava índices fixos — agora é dinâmico. |
| `_buildGanttSVGForExport(ai, fDS, fDE, viewMode, fmt)` | Gera SVG do Gantt de uma área para export PDF. Parâmetro `fmt` (A1/A2/A3/A4) controla escala — adicionado Jun/2026. |
| `pdfSelAll()` | Seleciona todas as áreas no wizard de Export PDF. |
| `pdfSelClear()` | Limpa a seleção de áreas no wizard de Export PDF. |

**Multi-select no Export PDF de Áreas (Jun/2026):** o wizard agora permite selecionar múltiplas áreas. O PDF gerado contém todas em sequência. Arquivo nomeado `Gantt_multiplas_areas_N_[período].pdf` quando > 1 área selecionada.

#### Feature: Exportar Pauta N2 como HTML navegável (nome da função mantido por compatibilidade)

Botão **📊 Exportar PPT** aparece junto com o FAB da Pauta N2 quando há marcos selecionados. Apesar do nome do botão/função, gera um `.html` navegável (não `.pptx`) — a troca de formato não renomeou a função para evitar quebrar referências existentes.

| Elemento | Detalhe |
|---|---|
| FAB | `id="n2-ppt-fab"` — `bottom:200px right:28px`, fundo `#065F46` |
| Visibilidade | Mesmo critério do FAB principal: `n > 0 && onEap && !locked` |
| Função JS | `exportN2PPT()` — lê seleção via `loadN2()`, aplica filtros ativos, gera HTML via `_buildN2HTMLDoc()` |
| Arquivo gerado | `Pauta_N2_YYYY-MM-DD.html` |

---

#### Filtro de Responsável (`index.html`, responsivo)

Comportamento único — não há mais implementação mobile separada (`buildRespFilterMobile`, `grupoHasRespM`, `toggleRespChip` e afins foram removidas do código junto com o `mobile.html` em 01/07/2026; confirmado via grep, zero ocorrências em `index.html`).

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

## 8. Skills disponíveis no repositório

Cada skill vive na sua própria pasta na raiz do repo, com o bundle `.skill` junto.

### doc-sync — Sincronização de documentação (Cowork)

Compara o `index.html` atual com o snapshot anterior, identifica mudanças relevantes e atualiza automaticamente os manuais (Manual de Uso, Guia de Onboarding, Ficha Técnica, ONBOARDING.md). Roda via **Cowork** com a pasta `maz-dashboard` montada.

**Como instalar:** abrir `doc-sync/doc-sync.skill` no Cowork e clicar em "Save skill".  
**Como acionar:** digitar `doc-sync` ou `"atualizar docs"` no chat Cowork.  
**Documentação completa:** `doc-sync/SKILL.md`

### code-audit — Auditor de código (Claude Code)

Analisa `index.html` completo em busca de problemas de segurança, arquitetura, qualidade de código, armadilhas JavaScript e dependências externas. Roda via **Claude Code** (terminal) com a pasta `maz-dashboard` aberta.

**Como instalar:** abrir `code-audit/code-audit.skill` no Claude Code e instalar.

| O que digitar | Modo |
|---|---|
| `"audita o que mudou"` | Só o `git diff` atual — rápido |
| `"auditoria completa"` | Lê `index.html` inteiro |

---

## 9. Armadilhas técnicas conhecidas

| Armadilha | Como evitar |
|---|---|
| Dashboard branco sem erro no console | Verificar: (a) null bytes no HTML, (b) palavra `function` ausente em declaração JS, (c) JS truncado sem `</script>`, (d) template literals aninhados |
| Template literals aninhados | Nunca usar crase dentro de `${}` dentro de outro crase — `node --check` passa mas browser quebra |
| JS truncado | Verificar se `</script>` existe no final do arquivo antes de editar |
| Prioridade REQS | Usar APENAS coluna D (índice 3) — coluna B gera falsos positivos. Índices antigos (E/4) causavam KPIs zerados (corrigido commit 7662590) |
| `node --check` no Node v22 | Não aceita `.html` — extrair bloco script para arquivo `.js` temporário |
| Edit tool do Claude Code falha | Arquivo contém backticks JS (template literals). Usar PowerShell com `[System.IO.File]::ReadAllBytes` |
| String não encontrada no Replace | Arquivo usa CRLF. Normalizar: `$content.Replace("\`r\`n", "\`n")` antes de substituir |
| Nome "Pingueira" na Área 0 | Nome correto é **"Pinguela"** — corrigido em commit 2970ecd. Usar sempre "Pinguela" em código e docs. |
| `exportN2PPT()` sem pptxgenjs | Se `PptxGenJS` não estiver definido (falha de CDN), a função alerta o usuário e retorna. Nunca presumir que o CDN carregou. |
| `preprocessStatusesDiretoria()` vs `preprocessStatuses()` | São duas funções separadas que processam fontes separadas (`WBS_DIR` vs `WBS`). Editar uma sem editar a outra causa divergência silenciosa entre a aba Status Report normal e a aba Diretoria. Ambas são chamadas em sequência no load (`preprocessStatuses(); preprocessStatusesDiretoria();`) — nunca remover uma achando que é redundante. |
| `WBS_DIR` é deep copy, não referência | `WBS_DIR.length=0; JSON.parse(JSON.stringify(newWBS)).forEach(e=>WBS_DIR.push(e));` — é uma cópia profunda tirada no momento do load. Edições de status manual feitas na aba Diretoria **não** propagam de volta para `WBS`, e vice-versa. Isso é intencional (permite status divergente para comparação), mas quebra a expectativa de "é a mesma árvore". |
| `window.print()` no Comparativo | O botão "Exportar PDF" do modal Comparativo (`comp-btn-pdf`) chama `window.print()` puro — não gera PDF programaticamente, depende do diálogo de impressão do browser. Sem CSS `@media print` dedicado, o layout impresso pode não coincidir com o modal na tela. |

---

## 10. Feature: Pauta N2

Permite selecionar tarefas individuais dentro de marcos para levar à reunião de diretoria (N2), filtrar só as selecionadas e visualizar a pauta.

### Como funciona (UX)
1. Na aba **Status Report**, expandir um grupo (ex: "#1.1 TR 01 OBRAS CIVIS") e clicar em "▸ X marcos"
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
| **FAB exportar PPT** | `id="n2-ppt-fab"` — `bottom:200px right:28px`, fundo `#065F46`. Visível quando `n > 0 && onEap && !locked`. Chama `exportN2PPT()`. |

### Exportar Pauta N2 como HTML navegável (nome da função mantido por compatibilidade)

Gera `.html` com as tarefas selecionadas — não mais `.pptx` (trocado em Jul/2026, função manteve o nome antigo).

| Função | Responsabilidade |
|---|---|
| `exportN2PPT()` | Lê seleção via `loadN2()` (IDs no formato `gi:mi:ti:si` — 4 partes), agrupa por marco/eixo, gera HTML via `_buildN2HTMLDoc()`. Arquivo: `Pauta_N2_YYYY-MM-DD.html`. |

### Compartilhamento via URL+PIN (commit 8d3268f)

Formato da URL gerada:
```
https://pmo-creator.github.io/maz-dashboard/index.html?n2=ID1,ID2,...&ph=HASH
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
| Fonte de dados | `WBS_DIR` (array separado de `WBS`) — ver [[armadilha #9]] sobre deep copy |
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

*Guia atualizado em 02/Jul/2026 — v17 — Dashboard MAZ 2026 · IDG PMO*
