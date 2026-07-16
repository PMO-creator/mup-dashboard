# doc-sync — Relatório 22/Jun/2026

**Snapshot anterior:** 10/Jun/2026 (259.630 bytes, 3.756 linhas)
**Snapshot atual:** 22/Jun/2026 (264.766 bytes, 3.786 linhas)
**Delta:** +5.136 bytes, +30 linhas, 22 hunks

---

## Mudanças identificadas

### 1. Feature N2: seleção migrou de MARCO para TAREFA (impacto alto)
- `toggleN2Marco(gi,mi,ti,ev)` → `toggleN2Task(domId,ev)`
- IDs de seleção: `gi:mi:ti` (3 partes) → `gi:mi:ti:si` (4 partes)
- Checkboxes saíram do `marco-band` e foram para `taskCard(t, n2Id)`
- `applyN2Filter()` agora filtra `taskcard-*` além de `mband-*`
- `clearN2Selection()` agora também limpa `taskcard-*`

### 2. Aba Áreas: toggle Semanal/Mensal removido (impacto médio)
- Botões `btn-areas-semanal` / `btn-areas-mensal` removidos do header da aba Áreas

### 3. Aba Áreas Gantt: labels de data nas barras (impacto médio)
- Barras do Gantt de Áreas agora exibem data de início (esquerda) e fim (direita) diretamente no SVG

### 4. Aba Áreas: auto-expand ao trocar modo de visualização (impacto baixo)
- `setAreasViewLevel()` agora expande todas as seções automaticamente ao trocar Marcos/Tarefas

---

## Documentos atualizados

| Documento | Versão | Alteração |
|---|---|---|
| Manual de Uso e Manutenção | v9 → v10 | §4.5 Pauta N2: "marcos" → "tarefas" em 5 ocorrências |
| Guia de Onboarding | v15 → v16 | §PPT export: IDs 4 partes; URL format: nota 4 partes; footer: 22/Jun/2026 |
| ONBOARDING.md | in-place (v16) | §10 completo: UX, estrutura técnica, IDs, funções, armadilhas |
| Memória project_pauta_n2.md | atualizada | Estrutura completa com IDs 4 partes e toggleN2Task |

**Ficha Técnica:** não afetada.

---

*Executado em 22/Jun/2026 — doc-sync v1*
