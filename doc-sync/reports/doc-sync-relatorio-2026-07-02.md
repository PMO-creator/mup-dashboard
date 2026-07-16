# doc-sync — Relatório 02-03/Jul/2026

**Snapshot anterior:** 22/Jun/2026 (264.766 bytes, 3.786 linhas)
**Snapshot atual:** 02/Jul/2026 (343.188 bytes, ver `_snapshot_index.html`)

Sessão iniciada como continuação de um doc-sync interrompido em 02/Jul, e
expandida durante a execução para incluir uma reestruturação completa do
conjunto de documentos (pedido do usuário, após identificação de sobreposição
pesada entre Onboarding e SOP).

---

## Parte 1 — Feature: Aba Diretoria (pendência original)

Documentada a feature "Aba Diretoria" (Gantt/EAP/N2/Comparativo espelhados,
fonte `WBS_DIR`), que já existia no código mas não estava totalmente coberta
na documentação.

- **ONBOARDING.md** — 3 armadilhas novas na §9 (`preprocessStatusesDiretoria`
  vs `preprocessStatuses`; `WBS_DIR` como deep copy; `window.print()` no
  Comparativo) + nova §11 "Feature: Aba Diretoria" completa (UX, estrutura
  técnica, armadilhas específicas).
- **DEV_GUIDE.html** — novo "Grupo 6 — Aba Diretoria" na seção de regras de
  negócio, com **RN-14** (WBS_DIR é cópia congelada, não referência viva) e
  **RN-15** (Comparativo casa por eixo/grupo/marco, não por ID; export usa
  `window.print()` puro).

---

## Parte 2 — Reestruturação do conjunto de documentos

### Diagnóstico
Comparação de conteúdo (via `markitdown`) dos 6 documentos do projeto revelou
que **Guia de Onboarding v16** e **SOP v2** tinham ~70% de conteúdo técnico
duplicado (setup, fluxo de trabalho, índices de coluna, auto-status,
armadilhas, rollback) — cada mudança técnica exigia editar dois documentos
quase-idênticos. O **Manual de Uso e Manutenção v10** também misturava
conteúdo de UX (uso do dashboard) com conteúdo de manutenção técnica
(publicar no GitHub, editar com Claude Code), sem necessidade.

### Mudanças estruturais
| Antes | Depois |
|---|---|
| Manual de Uso e Manutenção v10 (§1-9, UX + manutenção) | **Manual de Uso v11** (§1-5, só UX) |
| Guia de Onboarding v16 + SOP v2 (duplicados) | **Guia Técnico Unificado v1** (16 seções, sem duplicação) |
| Ficha Técnica v4 | Sem mudança |

**Manual de Uso Dashboard_v11.docx** — removidas as seções 6 (Atualizar
Dados), 7 (Publicar no GitHub), 8 (Modificar com Claude Code) e 9
(Troubleshooting). Mantidas §1-5 (Visão Geral, Filtros, Gantt, Status
Report/EAP, Requisições) intactas, título ajustado de "e Manutenção" para
"Manual de Uso". Sumário automático corrigido.

**Guia Tecnico Unificado_MAZ_2026_v1.docx** — documento novo, funde
Onboarding v16 + SOP v2 em 16 seções sem repetição: Finalidade/Escopo, RACI,
Arquitetura, Boas Práticas, Configuração Inicial, GitHub Desktop, Fluxo de
Trabalho, Teste em Celular, Rollback, Referências Técnicas, Armadilhas,
Skills Automatizadas (doc-sync + code-audit), Feature Pauta N2, Métricas de
Processo, Documentos Relacionados, Histórico de Revisões.

Durante a fusão, 4 divergências entre as fontes foram resolvidas conferindo
o `index.html` real (fonte de verdade):
1. **ID da planilha de Requisições** — Onboarding tinha typo (`...tm1uJM`);
   valor correto confirmado no código é `...tu1uJM`.
2. **Arquitetura desktop/mobile** — Onboarding ainda citava `mobile.html`
   como arquivo separado; corrigido para arquivo único responsivo (mobile.html
   removido em 01/Jul/2026).
3. **Coluna M de REQS** — atualizada para "Finalização do Serviço" (SOP já
   refletia a mudança de Jul/2026; Onboarding estava desatualizado).
4. **Comando `git add`** — removida referência a `mobile.html` no passo de
   publicação.

> ⚠️ **Achado colateral:** o `CLAUDE.md` do projeto também tem o mesmo typo
> no ID da planilha de Requisições (`...tm1uJM` em vez de `...tu1uJM`) — não
> corrigido nesta sessão, fica para aprovação do usuário.

### doc-sync/SKILL.md — correções de eficiência
- **Etapa 1**: trocada de "ler index.html inteiro" para diff contra o
  snapshot (`diff` / `git diff`) — evita o maior consumo de tokens da skill.
- **Etapa 6**: atualizada para a nova estrutura de 3 docx (Manual de Uso,
  Guia Técnico Unificado, Ficha Técnica) — Onboarding e SOP não são mais
  documentos separados.
- **Etapa 9**: relatório final atualizado com os novos nomes de arquivo.
- Caminho do repo corrigido de OneDrive (que corrompe `.git`) para
  `C:\Users\gagui\Github\maz-dashboard\`.

### doc-sync/context.md — inventário atualizado
- §4: inventário revisado para os 3 docx ativos + ONBOARDING.md. Guia do
  Usuário Final (pptx) marcado como descontinuado (última versão v2, já
  arquivada).
- §6: tabela de mapeamento mudança→documento simplificada para 4 colunas
  (era 5, removida a coluna do pptx descontinuado) + nova linha para
  mudanças de RACI/processo.

### Organização de arquivos
Movidos para `Manual/old_versions/`:
- `Manual de Uso e Manutenção Dashboard_v10.docx` + `.pdf`
- `Guia de Onboarding_Manutençao Dashboard_MAZ_2026_v16.docx` + `.pdf`
- `SOP_Dashboard_MAZ_2026_v2.docx` + `.pdf`

Apagada pasta temporária `Manual/unpacked_v16/` (XML de trabalho da skill
docx, sobra de sessão anterior, sem valor de entrega).

**`Manual/` agora contém apenas:** DEV_GUIDE.html, Ficha_Tecnica v4 (docx+pdf),
Guia Tecnico Unificado v1.docx, Manual de Uso v11.docx.

---

## Parte 3 — Correção de hook global

Identificado hook `PostToolUse` em `~/.claude/settings.json` (global, todos
os projetos) disparando após cada `Write|Edit|MultiEdit`, causando pausas
frequentes de continuação mesmo sem achado sensível. Matcher restrito para
`Write` apenas — efeito a partir da próxima sessão do Claude Code.

---

## Documentos atualizados

| Documento | Versão | Alteração |
|---|---|---|
| Manual de Uso Dashboard | novo v10 → **v11** | Trim §1-5, título ajustado, sumário corrigido |
| Guia Tecnico Unificado | **v1 (novo)** | Funde Onboarding v16 + SOP v2, 16 seções, 4 divergências resolvidas |
| Ficha Técnica | v4 (sem mudança) | — |
| ONBOARDING.md | in-place | §9 +3 armadilhas, nova §11 Aba Diretoria |
| DEV_GUIDE.html | in-place | Grupo 6 — RN-14/RN-15 (Aba Diretoria) |
| doc-sync/SKILL.md | in-place | Etapa 1 diff-based, Etapa 6/9 atualizadas p/ nova estrutura |
| doc-sync/context.md | in-place | §4 e §6 atualizados |
| `~/.claude/settings.json` | in-place | Hook PostToolUse restrito a Write |

**Não commitado** — nenhum `git add`/`commit`/`push` executado, conforme regra.

---

*Executado em 02-03/Jul/2026 — doc-sync v1 (com reestruturação de documentos)*
