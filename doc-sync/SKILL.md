---
name: doc-sync
description: >
  Skill de sincronização automática de documentação do Dashboard MAZ 2026.
  USAR SEMPRE que: o usuário fizer git push do dashboard, disser "doc-sync",
  "atualizar docs", "documentação", "sincronizar manuais", ou quando chamado
  pelo scheduled task diário. Compara o código atual do dashboard com o
  snapshot anterior, identifica mudanças relevantes, e atualiza os documentos
  afetados (Manual de Uso, Guia Técnico Unificado, Ficha Técnica) com aprovação
  do usuário antes de gravar.
---

# doc-sync — Sincronização de Documentação Dashboard MAZ 2026

## Contexto do projeto

Leia o arquivo `context.md` na mesma pasta desta skill antes de qualquer operação.
Ele contém: arquitetura técnica, estrutura WBS, paleta de cores, inventário de
documentos, regras de relevância e mapeamento mudança → documento.

Caminhos importantes:
- **Repo GitHub local (pasta única):** `C:\Users\gagui\Github\maz-dashboard\`
- **Manuais:** `C:\Users\gagui\Github\maz-dashboard\Manual\`
- **Snapshot de referência:** `C:\Users\gagui\Github\maz-dashboard\doc-sync\_snapshot_index.html`
- **Esta skill:** `C:\Users\gagui\Github\maz-dashboard\doc-sync\`
- **Relatórios de execução:** `C:\Users\gagui\Github\maz-dashboard\doc-sync\reports\`
- **Catálogo de erros de vibe coding (versionado, em Manual\):** `C:\Users\gagui\Github\maz-dashboard\Manual\Catalogo_Erros_Vibe_Coding_MAZ_v4.docx`
  (documento vivo — ver seção "Manutenção do Catálogo de Erros" abaixo; confirme a versão vigente com `ls Manual/*Catalogo_Erros*`)

> ⚠️ Não usar OneDrive como caminho do repo — o OneDrive corrompe a pasta `.git`
> ao sincronizar arquivos internos do git (ver `CLAUDE.md` do projeto).

> ⚠️ Regra crítica: NUNCA executar `git add`, `git commit` ou `git push` sem
> instrução explícita do usuário.

---

## Fluxo de execução

### ETAPA 1 — Diff contra o snapshot

**Nunca ler `index.html` inteiro.** Comparar contra o snapshot de referência via
diff — só os hunks alterados entram no contexto. Isso é o que evita o consumo
pesado de tokens da skill.

```bash
diff "doc-sync/_snapshot_index.html" "index.html"
# ou, se preferir granularidade de commit:
git diff -- index.html
```

Se o diff vier vazio → pular direto para ✅ "Commit limpo" (Etapa 3) sem ler
mais nada do arquivo.

---

### ETAPA 2 — Leitura do snapshot de referência

```python
SNAPSHOT_PATH = r"C:\Users\gagui\Github\maz-dashboard\doc-sync\_snapshot_index.html"

# Se não existir: avisar o usuário que é a primeira execução
if not exists(SNAPSHOT_PATH):
    avisar o usuário que é a primeira execução
```

---

### ETAPA 3 — Diff e classificação de mudanças

Comparar `index.html` atual com o snapshot linha a linha.
Para cada bloco diferente, classificar usando as regras do `context.md §5`:

**Classificações:**
- `RELEVANTE_UX` — novo botão, nova aba, novo filtro visível, mudança de navegação
- `RELEVANTE_TECH` — nova função JS relevante, novo campo Sheets, nova URL, nova armadilha
- `RELEVANTE_DADOS` — nova coluna parseada, novo status, ID de planilha alterado
- `IGNORAR` — refactor interno, CSS cosmético, dados embutidos atualizados, comentários

**Regra de ouro:** se um usuário final ou dev novo precisaria saber para usar ou
manter o dashboard → `RELEVANTE`. Dúvida → `RELEVANTE`.

Se **nenhuma mudança relevante** for encontrada:
```
✅ Commit limpo — nenhuma mudança que afete a documentação.
Snapshot atualizado. Nada a fazer.
```
Atualizar snapshot e encerrar.

---

### ETAPA 4 — Mapeamento de impacto

Para cada mudança `RELEVANTE_*`, usar a tabela do `context.md §6` para
determinar quais documentos e seções precisam ser atualizados.

Montar um relatório de impacto:

```
📋 MUDANÇAS ENCONTRADAS — [data]

1. [RELEVANTE_UX] Novo filtro de Responsável adicionado ao header
   → Afeta: Manual §3.3 | Onboarding §8

Total: 1 mudança relevante em 2 documentos.
```

---

### ETAPA 5 — Apresentação para aprovação (OBRIGATÓRIA)

Exibir para o usuário em linguagem de leigo:

```
🔔 doc-sync encontrou mudanças no dashboard que precisam ser documentadas.

Mudança 1: Foi adicionado um novo botão de filtro por Responsável.
  O que isso significa: os usuários agora podem filtrar pelo responsável.
  Vou atualizar: Manual (seção de filtros) + Guia de Onboarding

Posso prosseguir com as atualizações? [Sim / Não / Ver detalhes técnicos]
```

**Aguardar aprovação antes de qualquer escrita em arquivo.**

Se o usuário pedir detalhes técnicos, mostrar o diff bruto da mudança.
Se o usuário disser "não" em algum item específico, pular aquele item.

---

### ETAPA 6 — Atualização dos documentos

Para cada documento afetado, executar na ordem:

#### 6a. Manual de Uso (só UX — sem seções de manutenção técnica)
- Usar skill `docx` (unpack → edit XML → repack)
- Cobre §1-5: visão geral, filtros, Gantt, Status Report, Requisições
- Editar apenas as seções mapeadas — nunca reescrever o documento inteiro
- Incrementar versão ao salvar (ex: `_v11.docx` → `_v12.docx`)
- Mover versão anterior para `Manual/old_versions/`
- Tom: linguagem acessível, orientado a tarefa, sem jargão técnico

#### 6b. Guia Técnico Unificado (substitui Onboarding + SOP, fundidos em Jul/2026)
- Usar skill `docx`
- Cobre: finalidade/escopo, RACI, arquitetura, setup, fluxo de trabalho,
  rollback, referências técnicas (URLs, Sheets, colunas, funções JS, armadilhas),
  skills automatizadas (doc-sync, code-audit), métricas de processo
- Editar seções técnicas relevantes — não duplicar conteúdo entre seções
- Incrementar versão ao salvar (ex: `_v1.docx` → `_v2.docx`)
- Mover versão anterior para `Manual/old_versions/`
- Tom: técnico e preciso, comandos literais quando aplicável
- **Não recriar Onboarding e SOP como documentos separados** — desde Jul/2026
  esse conteúdo vive num documento só. Se uma mudança afeta RACI ou fluxo de
  processo, edita a seção correspondente aqui, não um arquivo novo.

#### 6c. Ficha Técnica (versão atual: `_v4.docx`)
- Usar skill `docx`
- Atualizar apenas URLs, IDs, dependências
- Incrementar versão ao salvar (ex: `_v4.docx` → `_v5.docx`)
- Mover versão anterior para `Manual/old_versions/`
- Tom: formal, tabular, conciso

#### 6d. ONBOARDING.md (`maz-dashboard/ONBOARDING.md`)
- Arquivo Markdown no repo git — editar in-place (sem versionamento numérico)
- **Critério de sincronização:** atualizar ONBOARDING.md **somente** quando a mudança
  altera comportamento técnico — nova função JS, novo índice de coluna, nova armadilha,
  novo fluxo de dados, nova estrutura de pastas. Explicações de UX humanas ficam
  apenas no docx. Regra prática: *"Um dev experiente precisaria saber isso para
  trabalhar corretamente?" → sim = atualizar; não = só docx.*
- Tom: técnico, direto, com tabelas de referência e comandos literais
- Após editar, commitar junto com as demais alterações do ciclo doc-sync

#### 6e. DEV_GUIDE.html (`Manual/DEV_GUIDE.html`)
- Arquivo HTML sem versionamento numérico — editar in-place
- Atualizar quando houver mudanças técnicas relevantes
- Após atualizar, commitar para publicar no GitHub Pages
- URL pública: `https://pmo-creator.github.io/maz-dashboard/Manual/DEV_GUIDE.html`
- Tom: técnico, orientado a desenvolvedor, com exemplos de código quando aplicável

#### 6f. Verificação de arquivamento (obrigatória para 6a–6c)

> Os números de versão citados em 6a/6b/6c (ex: `_v7.docx`) são só exemplos e
> podem estar desatualizados — a versão real vigente é sempre a que estiver
> fisicamente na pasta `Manual/`. Antes de editar, confirme com `ls Manual/` qual
> é a versão atual de cada documento; não confie no número escrito aqui.

Depois de salvar a nova versão de um documento (6a/6b/6c), antes de seguir para a
ETAPA 7, rode `ls Manual/*<nome-base-do-documento>*` e confirme que a pasta `Manual/`
raiz tem **exatamente uma** versão `.docx` daquele documento (a que
acabou de ser criada). Se aparecer mais de uma — a nova mais alguma antiga que
ficou pra trás de um ciclo anterior — mova TODAS as versões antigas para
`Manual/old_versions/` antes de encerrar o ciclo. Esse passo existe porque já
aconteceu de uma versão anterior ficar esquecida na raiz (ex: `_v9` não arquivado
quando o `_v10` foi criado) — o arquivamento faz parte do mesmo ato de salvar a
nova versão, não um passo separado que pode ser pulado.

> ⚠️ Regra crítica: NUNCA resumir ou parafrasear comentários vindos da
> planilha Google Sheets. Sempre exibir texto bruto.

---

### ETAPA 7 — PDF (não gerar)

> Decisão do usuário (Jul/2026): o doc-sync não gera mais PDF. Cada ciclo produz
> só o `.docx` atualizado — quem precisar do PDF exporta manualmente quando for
> consumir o documento. Não crie, não converta e não verifique PDFs nesta etapa.

---

### ETAPA 8 — Atualização do snapshot

Salvar o `index.html` atual como novo snapshot de referência:

```python
shutil.copy2(
    r"C:\Users\gagui\Github\maz-dashboard\index.html",
    r"C:\Users\gagui\Github\maz-dashboard\doc-sync\_snapshot_index.html"
)
```

---

### ETAPA 9 — Relatório final

Exibir resumo com links diretos:

```
✅ doc-sync concluído — [data]

Documentos atualizados:
• [Ver Manual de Uso vN](computer://C:\Users\gagui\Github\maz-dashboard\Manual\Manual de Uso Dashboard_vN.docx)
• [Ver Guia Técnico Unificado vN](computer://C:\Users\gagui\Github\maz-dashboard\Manual\Guia Tecnico Unificado_MAZ_2026_vN.docx)
• [Ver Ficha Técnica vN](computer://C:\Users\gagui\Github\maz-dashboard\Manual\Ficha_Tecnica_Dashboard_MAZ_2026_vN.docx)

Salvar relatório em: doc-sync/reports/doc-sync-relatorio-[data].md
```

---

## Manutenção do Catálogo de Erros

Além do fluxo de sincronização de documentação (Etapas 1-9, disparado por mudança
de código), o doc-sync também mantém vivo o **Catálogo de Erros de Vibe Coding**
— documento versionado em `Manual/`, no mesmo padrão de 6a-6c (Manual de Uso,
Guia Técnico Unificado, Ficha Técnica). Arquivo atual: `Manual/Catalogo_Erros_Vibe_Coding_MAZ_v4.docx`
— confirme sempre com `ls Manual/*Catalogo_Erros*` qual é a versão vigente antes
de editar (mesma regra da ETAPA 6f).

Cobre anti-padrões de vibe coding: Parte A (execução técnica — corrupção de
arquivo, commit ausente, substituição ambígua etc.) e Parte B (sessão
estratégica — over-engineering, scope creep, sycophancy etc., E01-E08).

**Gatilho — independente do diff de código:** sempre que, em qualquer sessão de
trabalho neste projeto (doc-sync ou não), um erro correspondente a um item já
catalogado se repetir, ou um padrão de erro novo (ainda não catalogado) for
identificado — seja por autodetecção ou apontado pelo usuário — registrar uma
nova entrada no catálogo:

- Usar a skill `docx` (unpack → editar XML → repack) — nunca reescrever o
  documento inteiro. Adicionar a entrada na seção correspondente (Parte A ou B),
  no mesmo formato dos itens existentes:
  - Título curto + severidade (Parte A) ou código EnN (Parte B)
  - **O que aconteceu:** descrição objetiva do incidente
  - **Como evitar:** regra prática, com comando/código se aplicável
  - **Caso real / Evidência:** data + sessão/feature onde ocorreu
- Numerar sequencialmente a partir do último item existente (Parte A: Erro 15+;
  Parte B: E09+).
- Atualizar também a seção "Log de novas ocorrências" no fim do documento.
- Incrementar versão ao salvar (ex: `_v4.docx` → `_v5.docx`) e mover a versão
  anterior para `Manual/old_versions/` — mesma verificação de arquivamento da
  ETAPA 6f (confirmar que só existe uma versão do catálogo na raiz de `Manual/`
  ao final).
- Este passo **não** precisa esperar um ciclo de doc-sync completo (Etapas 1-9)
  para acontecer — pode ser feito no momento em que o erro é identificado,
  dentro de qualquer sessão técnica ou estratégica do projeto.
- **Parte B ativa:** ao iniciar uma sessão de brainstorm/feature nova, execução/
  entregável, ou revisão de proposta neste projeto, aplicar as provocações
  E01-E08 relevantes (tabela "Provocações por tipo de tarefa" no catálogo) como
  autocheck — mesmo havendo sobreposição com as skills genéricas
  `chat-vibing-guard`/`code-vibing-guard`, elas ficam ativas aqui por decisão do
  usuário.
- A versão v3.1 original (pré-migração para docx versionado) fica arquivada em
  `Manual/old_versions/catalogo_erros_vibe_coding_v3.1_original.docx`, só como
  referência histórica.

---

## Regras de qualidade

1. **Nunca reescrever um documento inteiro** — editar cirurgicamente as seções afetadas
2. **Sempre pedir aprovação** antes de gravar qualquer arquivo
3. **Explicar em linguagem de leigo** — o usuário não precisa entender JS para aprovar
4. **Versionar sempre** — nunca sobrescrever sem incrementar versão
5. **Snapshot obrigatório** — sempre salvar novo snapshot ao final
6. **Git apenas com instrução explícita** — nunca rodar git add/commit/push automaticamente
7. **Catálogo de erros sempre atualizado** — todo erro técnico ou estratégico identificado numa sessão vira entrada nova em `catalogo_erros_vibe_coding.md`, sem esperar ciclo completo

---

## Para o próximo desenvolvedor

Esta skill vive em `maz-dashboard/doc-sync/SKILL.md`.
Para editar: abrir sessão Cowork com `maz-dashboard` montado → editar `doc-sync/SKILL.md`.

O doc-sync é executado **manualmente** — não há agendamento automático.
Para detectar se há mudanças pendentes, peça ao Claude: *"Checa se tem mudanças
no dashboard desde o último doc-sync"*.

Documentação completa em `Manual/Guia Tecnico Unificado_MAZ_2026_vXX.docx` (seção Skills Automatizadas).
