# context.md — Dashboard MAZ 2026
> Fundação da skill `doc-sync`. Leia este arquivo antes de qualquer operação de atualização de documentação.
> Última atualização: Mai/2026

---

## 1. O que é este projeto

**Museu das Amazônias (MAZ) 2026** é uma exposição de longa duração gerenciada pelo IDG (Instituto de Desenvolvimento e Gestão). O dashboard é uma ferramenta interna de gestão de projetos que exibe o cronograma, status, requisições e indicadores da exposição em tempo real.

- **URL pública:** https://pmo-creator.github.io/maz-dashboard/ (arquivo único, responsivo desktop + mobile — `mobile.html` removido em 01/07/2026)
- **Repositório GitHub:** https://github.com/PMO-creator/maz-dashboard
- **Conta GitHub:** pmo-creator (exposicoeseprojetos@gmail.com)

---

## 2. Arquitetura técnica

| Componente | Detalhe |
|---|---|
| Hospedagem | GitHub Pages — branch `main` |
| Arquivo único | `index.html` (single-file, desktop + mobile responsivo, ~330KB) |
| Dados WBS | Google Sheets API v4 — leitura ao vivo pelo browser |
| Dados REQS | Google Sheets API v4 — leitura ao vivo pelo browser |
| Auto-refresh | 15 minutos (`AUTO_REFRESH_MS = 15 * 60 * 1000`) |
| Biblioteca de gráficos | Chart.js 4.5.1 via CDN |
| Sem build, sem framework | HTML + CSS + JS vanilla, tudo em arquivo único |

### Planilhas Google Sheets

| Dado | ID da Planilha | Aba |
|---|---|---|
| Cronograma (WBS) | `17nttJ_ShqWztvDWH3l59iNqboLqkviZs3_PM5J3ihdA` | `master data` |
| Requisições (REQS) | `1azrdS4OGO-CWD1ods69i8iZJcwq4oyISdT2n_tu1uJM` | `Planilha de Status de Compras Prod` |

### Colunas do Cronograma (índices 0-based)

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
| AL | 36 | Progresso (coluna dinâmica — detectada por `_findWeekCols`) |
| AM | 37 | Impacto (coluna dinâmica) |
| AN | 38 | Encaminhamento (coluna dinâmica) |

### Colunas das Requisições (índices 0-based)

> ⚠️ Corrigido em commit 7662590 — índices anteriores estavam deslocados e causavam KPIs zerados.

| Coluna | Índice | Campo |
|---|---|---|
| A | 0 | N. Requisição |
| B | 1 | Comprador |
| D | 3 | Prioridade ← USAR APENAS ESTA |
| E | 4 | Descrição |
| F | 5 | Status |
| G | 6 | Fornecedor |
| H | 7 | Fornecedor (coluna alternativa — ignorar, usar G) |
| M | 12 | Data prevista |

> ⚠️ Prioridade: usar APENAS coluna D (índice 3). Coluna B gera falsos positivos.
> ⚠️ Filtro de Fornecedor adicionado em commit b4a0b53 — dropdown multi-select na aba Requisições (coluna G). Itens sem fornecedor agrupados como "Sem Fornecedor".

---

## 3. Estrutura do WBS (hierarquia de dados)

```
Projeto MAZ 2026
└── Eixo (9 eixos)
    └── Grupo (48 grupos no total — deduplicação PER EIXO)
        └── Marco
            └── Tarefa (subtask)
```

### Regras críticas de parsing

- **Deduplicação per eixo:** mesmo nome de grupo em eixos diferentes = entradas separadas (nunca deduplicar globalmente)
- **Linha-resumo do grupo:** linha onde `Tarefa == Grupo` é o summary row do grupo
- **Marco vs Tarefa:** linha onde `Tarefa == Marco` é o marco; linhas seguintes com o mesmo marco são subtasks
- **NUNCA resumir comentários:** exibir texto bruto exatamente como está na planilha

### Status e cores CSS

| Status | CSS var | Cor | Hex |
|---|---|---|---|
| Feito | `--feito` | Verde | `#05C46B` |
| Em andamento | `--andam` | Azul | `#1A56FF` |
| Risco de atraso | `--risco` | Laranja | `#FF6B00` |
| Atrasado | `--atraso` | Vermelho | `#FF2525` |
| Definir datas | `--definir` | Marrom | `#92400E` |
| A iniciar | `--iniciar` | Cinza azulado | `#5A6E85` |
| Cancelado/Congelado | `--cancel` | Cinza escuro | `#374151` |

### Paleta de destaque

| Variável | Hex | Uso |
|---|---|---|
| `--accent` | `#8AC43A` | Verde primário — botões, bordas, chips |
| `--accentD` | `#6BA030` | Verde escuro — hover states |
| `--bg` | `#1E2E0D` | Fundo do header e painéis escuros |
| `--bg2` | `#2F4A18` | Fundo secundário |

### Regra do pior status (rollup)

```
Rank: Atrasado(0) > Risco de atraso(1) > Em andamento(2) > Definir datas(3) > A iniciar(4) > Feito(5) > Cancelado/Congelado(6)
Marco = pior status das suas tarefas
Grupo = pior status dos seus marcos
Eixo  = pior status dos seus grupos
```

> Regras de auto-status (commit 2ea1b35): subtask sem status + sem data → "Definir datas"; "A iniciar" sem data de início → "Definir datas"; padrão de grupo/eixo sem status → "Definir datas".

---

## 4. Inventário de documentos

> ⚠️ Estrutura consolidada em Jul/2026 — Onboarding e SOP eram ~70% conteúdo
> duplicado (setup, fluxo de trabalho, índices de coluna, auto-status,
> armadilhas, rollback) e foram fundidos num documento só. Guia do Usuário
> Final (pptx) está descontinuado — não existe versão ativa desde então.

### 4.1 Manual de Uso Dashboard

- **Arquivo atual:** ver `ls Manual/` para a versão vigente (não confiar em número fixo aqui)
- **Público-alvo:** Gestores do projeto, equipe IDG — usuários do dashboard no dia a dia. Nível: não-técnico.
- **Tom:** Orientado a tarefa, direto, linguagem acessível. Sem jargão de código.
- **O que cobre:** SÓ como usar o dashboard (filtros, abas, visualizações, Gantt, Requisições), legenda de cores. **Não cobre** atualização de dados, publicação no GitHub ou modificação de código — isso é conteúdo do Guia Técnico Unificado, não do Manual de Uso.
- **Seções críticas:**
  - Legenda de Cores — tabela Status/Cor/Significado
  - Barra de filtros — Status, Eixo, Data, Responsável
  - VISUALIZAÇÃO dropdown — Eixos/Grupo/Marco/Tarefas
  - Aba Status Report (EAP)
  - Aba Gantt
  - Aba Requisições
- **Versionar quando:** Qualquer mudança na UX, novos filtros, novas abas, comportamento alterado de feature existente, nova legenda de status.

### 4.2 Guia Técnico Unificado (substitui Guia de Onboarding + SOP, fundidos Jul/2026)

- **Arquivo atual:** ver `ls Manual/` para a versão vigente
- **Público-alvo:** Desenvolvedor que vai manter o código, e eventualmente a TI do cliente. Nível: técnico, mas pode ser iniciante em JS.
- **Tom:** Técnico, preciso. Comandos git literais. Sem abstrações.
- **O que cobre:** Finalidade/escopo, RACI (papéis e responsabilidades), arquitetura, boas práticas, configuração inicial, fluxo de trabalho (teste → push → produção), reverter versões/rollback, referências técnicas (URLs, Sheets IDs, colunas, funções JS, auto-status), armadilhas conhecidas, skills automatizadas (doc-sync, code-audit), métricas de processo.
- **Seções críticas:**
  - Arquitetura + fluxo de dados
  - RACI — papéis e responsabilidades
  - Configuração inicial (git clone, API key)
  - Fluxo de trabalho
  - Referências técnicas (URLs, Sheets, colunas, funções JS, filtros)
  - Armadilhas técnicas conhecidas
  - Rollback
- **Versionar quando:** Qualquer mudança técnica — nova função JS relevante, novo parâmetro, mudança de URL, novo campo no Sheets, nova armadilha descoberta, novo fluxo de publicação, mudança de RACI/processo.
- **Regra:** não recriar Onboarding e SOP como arquivos separados. Todo conteúdo técnico de manutenção vive aqui, num documento só.

### 4.3 Ficha Técnica Dashboard

- **Arquivo atual:** ver `ls Manual/` para a versão vigente
- **Público-alvo:** Stakeholders, gestores seniores, TI. Leitura pontual para entender o que é o sistema.
- **Tom:** Conciso, formal, informativo. Uma página de referência.
- **O que cobre:** URLs, repositório, conta GitHub, fontes de dados, arquitetura em tabela, API key, conta de serviço.
- **Versionar quando:** Mudança de URL, repositório, conta, ID de planilha, dependência externa (ex: troca de biblioteca CDN).

### 4.4 [DESCONTINUADO] Guia do Usuário Final

Pptx descontinuado (última versão v2, arquivada em `Manual/old_versions/`). Não
recriar automaticamente — se o usuário pedir esse formato de volta, perguntar
antes de gerar.

### 4.5 ONBOARDING.md (repo)

- **Arquivo atual:** `maz-dashboard/ONBOARDING.md`
- **Público-alvo:** Desenvolvedor novo que clonou o repo, e eu (Claude) para manutenção assistida. Lê diretamente no GitHub sem precisar baixar nada. Nível: técnico.
- **Tom:** Técnico, direto. Comandos literais. Tabelas de referência rápida.
- **O que cobre:** Arquitetura, boas práticas, configuração inicial, fluxo de trabalho, referências técnicas (URLs, Sheets, colunas, funções JS, filtros), armadilhas.
- **Seções críticas:**
  - §1 Arquitetura + fluxo de dados
  - §3 Estrutura de pastas de trabalho
  - §7 Referências técnicas (colunas das planilhas, auto-status, filtros)
  - §8 Armadilhas técnicas
- **Versionar quando:** Qualquer mudança técnica que afetaria o Guia Técnico Unificado — nova função JS relevante, novo parâmetro, mudança de coluna no Sheets, nova armadilha, novo fluxo de trabalho, mudança na estrutura de pastas.
- **Importante:** É o complemento em Markdown do Guia Técnico Unificado. Quando o docx é atualizado para nova versão, o ONBOARDING.md também deve ser atualizado com as mesmas informações técnicas.

---

## 5. Regras de relevância — o que dispara atualização de docs

### ✅ RELEVANTE — atualiza docs

| Tipo de mudança | Exemplos |
|---|---|
| Nova funcionalidade | Novo filtro, nova aba, novo botão visível, nova visualização |
| Comportamento alterado | Filtro que antes era simples vira multi-select, Gantt muda modos |
| Nova estrutura de dados | Nova coluna lida do Sheets, novo campo parseado |
| Mudança de URL ou ID | URL do dashboard, ID da planilha, nome da aba |
| Novo status ou cor de status | Adição de novo valor de status |
| Nova armadilha técnica | Bug descoberto e corrigido que outros devs precisam saber |
| Mudança de dependência externa | Versão do Chart.js, novo CDN |
| Novo arquivo adicionado | Novo script auxiliar, SERVE_DASHBOARD.bat |

### ❌ NÃO RELEVANTE — não atualiza docs

| Tipo de mudança | Exemplos |
|---|---|
| Refactor interno | Rename de variável JS, extração de função, reorganização de código |
| Ajuste visual CSS | Mudança de cor hex, padding, border-radius, font-size |
| Dados embutidos atualizados | Snapshot WBS/REQS atualizado no HTML |
| Comentários no código | Adição ou remoção de `//` comentários |
| Correção de bug invisível ao usuário | Fix de lógica interna sem mudança de comportamento perceptível |
| Atualização de data de referência | `dateLabel` ou similar |

---

## 6. Mapeamento: tipo de mudança → documentos afetados

| Mudança | Manual de Uso | Guia Técnico Unificado | Ficha Técnica | ONBOARDING.md |
|---|---|---|---|---|
| Novo filtro visível | ✅ | ✅ referências | ❌ | ✅ §7 filtros |
| Nova aba no dashboard | ✅ nova seção | ✅ arquitetura+referências | ❌ | ✅ §1+§7 |
| Novo botão no header | ✅ | ❌ | ❌ | ❌ |
| Nova coluna lida do Sheets | ❌ | ✅ referências colunas | ❌ | ✅ §7 colunas |
| Mudança de URL | ✅ se menciona URL | ✅ referências URLs | ✅ | ✅ §7 URLs |
| Mudança ID planilha/aba | ❌ | ✅ referências | ✅ | ✅ §7 |
| Novo status/cor | ✅ legenda | ✅ referências | ❌ | ✅ §7 auto-status |
| Nova armadilha técnica | ❌ | ✅ armadilhas | ❌ | ✅ §8 |
| Mudança de dependência CDN | ❌ | ✅ arquitetura | ✅ | ✅ §1 |
| Novo arquivo no repo | ❌ | ✅ arquitetura+fluxo | ❌ | ✅ §3 estrutura pastas |
| Mudança de comportamento EAP | ✅ | ✅ referências | ❌ | ❌ |
| Mudança de comportamento Gantt | ✅ | ✅ referências | ❌ | ❌ |
| Mudança de comportamento REQS | ✅ | ✅ referências filtros | ❌ | ✅ §7 filtros REQS |
| Mudança na estrutura de pastas | ❌ | ✅ configuração+fluxo | ❌ | ✅ §3 |
| Mudança de RACI/processo | ❌ | ✅ RACI | ❌ | ❌ |

---

## 7. Armadilhas técnicas conhecidas (não repetir esses bugs)

| Armadilha | Como evitar |
|---|---|
| Dashboard branco sem erro no console | Verificar: (a) null bytes no HTML, (b) `function` ausente em declaração JS, (c) JS truncado — verificar se `</script>` existe no final |
| Template literals aninhados | Nunca usar crase dentro de `${}` dentro de outro crase — `node --check` passa mas browser quebra |
| JS truncado | Verificar se `</script>` existe no final do arquivo antes de editar |
| Prioridade REQS | Usar APENAS coluna D (índice 3) — coluna B gera falsos positivos |
| `node --check` no Node v22 | Não aceita `.html` — extrair bloco `<script>` para arquivo `.js` temporário |
| Deduplicação de grupos | Sempre per eixo — nunca global |
| `--definir` (Definir datas) | Cor é MARROM `#92400E`, não roxo |

---

## 8. Boas práticas de desenvolvimento

**Sempre fazer:**
- Testar local antes de qualquer push
- Hard refresh (Ctrl+Shift+R) ao testar
- Salvar backup com timestamp antes de edições grandes

**Nunca fazer:**
- Editar direto no GitHub pelo browser (vai direto para produção sem teste)
- Confiar só no `node --check` — testar no browser sempre
- Resumir ou parafrasear comentários da planilha — exibir texto bruto

---

## 9. Histórico de versões dos documentos

| Documento | Versão atual | Notas |
|---|---|---|
| Manual de Uso e Manutenção | v7 | §6 filtro Fornecedor + botão Limpar · §3.5 ranking status · §5 datas automáticas (Mai/2026) |
| Guia de Onboarding | v12 | §10 skills disponíveis (code_audit + doc-sync) · §3 estrutura 2 pastas · regras auto-status revisadas + ranking com números · armadilha Prioridade D/3 corrigida (Mai/2026) |
| Ficha Técnica | v3 | — |
| Guia Usuário Final | v3 | Slide Requisições: filtro Fornecedor + botão Limpar (Mai/2026) |

---

## 10. Feature: Pauta N2 (implementada em 30/Mai/2026)

Funcionalidade de seleção de marcos para reunião de diretoria (N2), disponível na aba Status Report do `index.html`.

**Comportamento:** usuário expande um grupo → clica "▸ X marcos" → checkboxes aparecem em cada tile → marcar checkbox seleciona o marco → badge "N2" aparece ao lado do nome → FAB "📋 Pauta N2 · N" mostra contagem e filtra ao clicar.

**Pontos críticos para o doc-sync:**
- Seleções são **in-memory** (`var _n2Sel=[]`), não localStorage — resetam a cada refresh
- Checkboxes ficam no template `buildCommentPanel()`, entre a setinha e o badge de status
- IDs seguem o padrão `gi:mi:ti` (3 partes) — diferente de qualquer outra feature do dashboard
- FABs (`n2-fab`, `n2-clear-fab`) só aparecem na aba EAP
- `initN2Checkboxes()` é chamado no final de `renderTree()` para restaurar estado visual

**Funções JS:** `toggleN2Marco`, `updateN2Fab`, `clearN2Selection`, `applyN2Filter`, `toggleN2Filter`, `initN2Checkboxes`, `loadN2`, `saveN2`, `n2Id`, `getN2Key`

---

## 11. Snapshot de referência

> O arquivo `_snapshot_index.html` (gerado pela skill `doc-sync` após cada execução bem-sucedida) é a versão do `index.html` usada como linha de base para o próximo diff.
> Localização: `maz-dashboard\.claude\doc_sync\_snapshot_index.html`
> Snapshot atual: gerado em 22/05/2026 (última execução do doc-sync).
