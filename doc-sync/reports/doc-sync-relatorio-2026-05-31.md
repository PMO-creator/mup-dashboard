# doc-sync — Relatório de Execução

**Data:** 31/Mai/2026
**Commit analisado:** `8d3268f` — feat: Pauta N2 — compartilhamento via URL+PIN com popup de link
**Snapshot anterior:** gerado em 22/Mai/2026

---

## Mudanças detectadas

### [RELEVANTE_UX] Botão "Publicar Pauta N2" + modo view-only
Adicionado botão azul "📤 Publicar pauta" (n2-publish-fab) e botão roxo "🔓 Desbloquear" (n2-lock-fab). Ao publicar, o usuário define um PIN e recebe um link compartilhável. Quem abre o link vê a pauta em modo view-only com checkboxes desabilitados.

### [RELEVANTE_TECH] Funções JS de compartilhamento via URL+PIN
Novas funções: `initN2FromURL()`, `publishN2Pauta()`, `unlockN2Edit()`, `_n2Hash()`.
Novas variáveis globais: `_n2ViewMode`, `_n2Unlocked`.
Formato da URL: `?n2=IDs&ph=HASH`.

---

## Documentos atualizados

| Documento | Versão | O que mudou |
|---|---|---|
| Manual de Uso e Manutenção | v7 → v8 | Nova seção §4.5 Pauta N2 — Compartilhamento de Pauta |
| Guia de Onboarding | v13 → v14 | Nova seção §11 com funções JS, variáveis de estado e FABs; tabela de funções da §10 expandida |
| ONBOARDING.md | in-place (v14) | §10 tabela expandida com novas funções e FABs; novo bloco §Compartilhamento via URL+PIN |

**Ficha Técnica:** sem alterações necessárias.

---

## Snapshot atualizado

Novo snapshot salvo em `doc-sync/_snapshot_index.html` (3094 linhas, commit 8d3268f).

---

## Arquivos movidos para old_versions/
- `Manual de Uso e Manutenção Dashboard_v7.docx` + `.pdf`
- `Guia de Onboarding_Manutençao Dashboard_MAZ_2026_v13.docx` + `.pdf`
