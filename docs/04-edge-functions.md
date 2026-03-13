# ⚡ Edge Functions — Supabase Deno

---

## Visão Geral

| Função | Auth | Status | Descrição |
|--------|------|--------|-----------|
| `process-recipe-etl` | ✅ JWT obrigatório | ✅ Em uso | Importação de fichas técnicas via Google Gemini AI |
| `delete-user` | ✅ JWT obrigatório | ✅ Em uso | Exclusão de conta do usuário autenticado |
| `import-ingredients-etl` | ✅ JWT obrigatório | ⚠️ Deployada, não utilizada | ETL de importação em massa de ingredientes |

---

## `process-recipe-etl`

**Arquivo:** `supabase/functions/process-recipe-etl/index.ts`

### Responsabilidade

Recebe um arquivo (PDF, DOCX, XLSX, imagem) via `multipart/form-data`, extrai o texto quando necessário, e chama a API do Google Gemini para estruturar os dados em formato de ficha técnica JSON.

### Fluxo de Processamento

```
Cliente → POST /functions/v1/process-recipe-etl
  │
  ├─ 1. Validação do JWT (getClaims)
  ├─ 2. Verifica GEMINI_API_KEY nos secrets
  ├─ 3. Recebe arquivo via multipart/form-data
  ├─ 4. Detecta tipo do arquivo:
  │     ├─ DOCX/XLSX → extrai texto via unzip do XML interno
  │     ├─ PDF/imagem → converte para base64 (inline_data para Gemini)
  │     └─ DOC (.doc legacy) → envia como base64
  ├─ 5. Chama Gemini API com fallback de modelos
  │     ├─ gemini-2.0-flash (tentativa 1)
  │     ├─ gemini-1.5-flash-8b (tentativa 2)
  │     └─ gemini-flash-latest (tentativa 3)
  ├─ 6. Parseia e valida o JSON retornado
  │     └─ Se truncado → tenta reparar via repairTruncatedJson()
  └─ 7. Retorna ExtractedRecipe como JSON
```

### Schema de Retorno

```typescript
interface ExtractedRecipe {
  name: string;
  type: "intermediaria" | "final";
  yield_quantity: number;
  yield_unit: "g" | "kg" | "ml" | "l" | "un" | "porcao" | "ft";
  ingredients: Array<{
    name: string;
    quantity: number;
    unit: string;
    fc: number;
  }>;
  preparation_steps: Array<{
    phase: "pre_preparo" | "preparo" | "finalizacao" | "empratamento";
    description: string;
    inferred: boolean;  // true = gerado pela IA, false = extraído do arquivo
  }>;
}
```

### Campo `inferred` nas Etapas

O campo `inferred` distingue etapas extraídas do arquivo original vs. etapas inferidas pela IA:

| Cenário | Comportamento |
|---------|---------------|
| Arquivo sem modo de preparo | Todas as etapas com `inferred: true` |
| Arquivo com modo de preparo completo | Todas as etapas com `inferred: false` |
| Arquivo com modo de preparo parcial | Apenas as etapas ausentes com `inferred: true` |

Na UI (`SmartRecipeImporter.tsx`), etapas com `inferred: true` exibem badge roxo **"IA"** e uma legenda de aviso.

### SYSTEM_PROMPT — Regras Principais

1. **Fases de preparo:** `pre_preparo`, `preparo`, `finalizacao`, `empratamento` — exatamente estas strings
2. **Passos atômicos:** cada passo = 1 ação simples. Prefere 10 passos simples a 3 compostos
3. **Normalização de ingredientes:** remove adjetivos comerciais, mantém nome genérico + qualificador técnico
4. **yield_unit:** apenas `g`, `kg`, `ml`, `l`, `un`, `porcao`, `ft` — sem variações com acento
5. **Outros:** `fc` padrão 1.0, `type` baseado no uso da receita
6. **inferred:** marca etapas deduzidas como `true`

### Autenticação

Usa `anonClient.auth.getClaims(token)` para validação local do JWT (sem chamada de rede ao Supabase Auth).

### Variáveis de Ambiente

| Secret | Descrição |
|--------|-----------|
| `GEMINI_API_KEY` | Chave da Google AI Studio. Configurar em: Supabase Dashboard → Edge Functions → Secrets |
| `SUPABASE_URL` | Injetado automaticamente pelo Supabase |
| `SUPABASE_ANON_KEY` | Injetado automaticamente pelo Supabase |

### Formatos Suportados

| Extensão | Tratamento |
|----------|-----------|
| `.pdf` | base64 → inline_data para Gemini |
| `.docx` | ZIP → extrai `word/document.xml` → texto limpo |
| `.doc` | base64 → inline_data para Gemini |
| `.xlsx` | ZIP → extrai `xl/sharedStrings.xml` + worksheets → texto limpo |
| `.xls` | base64 → inline_data para Gemini |
| `.png`, `.jpg`, `.jpeg`, `.webp` | base64 → inline_data para Gemini |

### Tratamento de Erros

| Código HTTP | Causa | Mensagem |
|-------------|-------|---------|
| 401 | JWT inválido ou ausente | `"Token inválido"` |
| 400 | Arquivo ausente ou formato inválido | Mensagem descritiva |
| 500 | Gemini indisponível / JSON inválido | Mensagem de erro interno |
| Gemini 401/403 | `GEMINI_API_KEY` inválida | Instrução de verificação |
| Gemini 429 | Cota atingida | Instrução de verificar billing |
| Gemini 404 | Modelo indisponível | Tenta próximo modelo no fallback |

---

## `delete-user`

**Arquivo:** `supabase/functions/delete-user/index.ts`

### Responsabilidade

Exclui permanentemente a conta do usuário autenticado. Chamada pela página `/perfil`.

### Fluxo

1. Valida JWT do usuário
2. Usa `supabaseAdmin` (service role) para deletar o usuário de `auth.users`
3. O cascade remove `profiles` e dados relacionados

---

## `import-ingredients-etl`

**Arquivo:** `supabase/functions/import-ingredients-etl/index.ts`

> ⚠️ **Status:** Deployada no Supabase mas **não utilizada** pelo frontend atual. A importação em massa de ingredientes foi pausada — apenas cadastro manual.

---

## Como Fazer Deploy das Edge Functions

```bash
# Via Supabase CLI
supabase functions deploy process-recipe-etl
supabase functions deploy delete-user

# Configurar secret da Gemini
supabase secrets set GEMINI_API_KEY=sua-chave-aqui
```

Ou via Lovable: as funções são deployadas automaticamente ao editar os arquivos.
