# 🛠️ Guia de Desenvolvimento

---

## Setup Local

```bash
# 1. Clone o repositório
git clone <YOUR_GIT_URL>
cd chefs-office

# 2. Instale as dependências
npm install

# 3. Configure as variáveis de ambiente
cp .env.example .env
# Preencha com suas chaves do Supabase

# 4. Inicie o servidor de desenvolvimento
npm run dev
# → http://localhost:8080
```

### Variáveis de Ambiente (.env)

```env
VITE_SUPABASE_URL=https://seu-projeto.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sua-anon-key
```

---

## Scripts Disponíveis

| Comando | Descrição |
|---------|-----------|
| `npm run dev` | Servidor de desenvolvimento (porta 8080) |
| `npm run build` | Build de produção |
| `npm run preview` | Preview do build de produção |
| `npm run lint` | ESLint |
| `npm test` | Vitest (testes unitários) |

---

## Regras Obrigatórias de Desenvolvimento

### 1. Schema do Banco é Imutável

> ⚠️ **NUNCA** criar ou modificar tabelas pelo frontend. O schema já existe no Supabase. Use sempre os nomes exatos de tabelas e colunas documentados em `docs/03-banco-de-dados.md`.

### 2. Idioma da Interface

Toda a UI deve estar em **Português do Brasil**. Isso inclui:
- Mensagens de erro e sucesso
- Labels de formulários
- Toasts e modais
- Estados de loading (`"Carregando..."`, `"Salvando..."`, etc.)

### 3. `photo_url` é Obrigatória

Receitas **não podem ser salvas** sem foto. Bloquear o submit do formulário enquanto `photo_url` estiver vazio.

### 4. Design System — Tokens Semânticos

**NUNCA** usar cores diretas nos componentes. Sempre usar tokens do design system via `index.css` e `tailwind.config.ts`:

```tsx
// ❌ ERRADO
<div className="bg-amber-600 text-white">

// ✅ CORRETO
<div className="bg-primary text-primary-foreground">

// ✅ CORRETO com HSL inline (quando necessário)
<div style={{ background: 'hsl(43 84% 41%)' }}>
```

### 5. Type Casting em Operações Supabase

Algumas tabelas apresentam discrepâncias entre os tipos gerados e o schema real:

```typescript
// profiles — campos como role, onboarding_completed
await supabase
  .from('profiles')
  .update({ onboarding_completed: true } as any)
  .eq('id', userId);

// menu_recipes — chef_responsible_id obrigatório
await supabase
  .from('menu_recipes')
  .insert({ ...data, chef_responsible_id: user.id } as any);
```

### 6. Proteção de Rotas

- Usar `<ProtectedRoute>` em todas as rotas que requerem autenticação
- Usar `<AdminRoute>` para rotas administrativas
- Nunca verificar `is_admin` via localStorage — sempre buscar do Supabase

---

## Padrões de Código

### Fetch de Dados

```typescript
// Padrão com React Query
const { data, isLoading, error } = useQuery({
  queryKey: ['fichas', establishmentId],
  queryFn: async () => {
    const { data, error } = await supabase
      .from('recipes')
      .select('*')
      .eq('establishment_id', establishmentId);
    if (error) throw error;
    return data;
  },
  enabled: !!establishmentId,
});
```

### Componentes de Página

```typescript
// Estrutura padrão de uma página protegida
export default function MinhaPage() {
  const { activeEstablishment } = useEstablishment();
  
  // Loading state
  if (!activeEstablishment) {
    return <div>Selecione um estabelecimento</div>;
  }
  
  return (
    <AppLayout>
      {/* conteúdo */}
    </AppLayout>
  );
}
```

### Tratamento de Erros

```typescript
try {
  const { error } = await supabase.from('recipes').insert(data);
  if (error) throw error;
  toast.success('Ficha salva com sucesso!');
} catch (err) {
  console.error(err);
  toast.error('Erro ao salvar ficha. Tente novamente.');
}
```

---

## Configuração de Ferramentas

### Vite (`vite.config.ts`)

- Porta: `8080`
- HMR overlay desabilitado
- Alias `@/` → `src/`
- Plugin `componentTagger` ativo apenas em dev

### TypeScript (`tsconfig.app.json`)

- `strict: false` — TypeScript menos rígido para agilidade
- `noImplicitAny: false`
- `skipLibCheck: true`
- Target: `ES2020`

### Tailwind (`tailwind.config.ts`)

- Dark mode via classe
- Todas as cores customizadas devem ser adicionadas aqui como tokens HSL
- Animações: `accordion-down`, `accordion-up`

### shadcn/ui (`components.json`)

- Estilo: `default`
- BaseColor: `slate`
- CSS variables habilitadas

---

## Testes

**Framework:** Vitest

```bash
npm test
```

**Arquivos:**
- `src/test/setup.ts` — configuração global
- `src/test/example.test.ts` — exemplo de teste
- `supabase/functions/process-recipe-etl/index_test.ts` — testes da edge function

---

## Arquivos READ-ONLY (nunca editar)

| Arquivo | Motivo |
|---------|--------|
| `src/integrations/supabase/types.ts` | Gerado automaticamente do schema Supabase |
| `.gitignore` | Configuração de versionamento |
| `supabase/migrations/` | Histórico imutável de migrações SQL |
| `bun.lock` / `package-lock.json` | Lockfiles de dependências |

---

## Decisões Técnicas Registradas

| Decisão | Motivo |
|---------|--------|
| Ingredientes sem seed automático | Evita poluição da base durante testes; dá controle total ao usuário |
| Importação de ingredientes removida do frontend | Funcionalidade pausada — apenas cadastro manual |
| `photo_url` obrigatória em receitas | Padrão de qualidade — impede fichas incompletas |
| Upsert em fichas via IA (`name + establishment_id`) | Evita duplicatas ao reimportar a mesma receita |
| `getClaims()` em vez de `getUser()` na Edge Function | Evita chamada de rede; valida JWT localmente — mais confiável |
| Badge "IA" em etapas inferidas | Transparência para o chef sobre o que foi gerado vs. extraído |
| Dois sistemas de toast (sonner + shadcn) | Legado; padronizar para sonner em implementações futuras |

---

## Pendências e Próximos Passos

- [ ] **Plano Pro / Billing** — Integração com gateway de pagamento (Stripe)
- [ ] **Storage seguro** — Tornar buckets privados + signed URLs
- [ ] **Validação de uploads** — Checar MIME type e tamanho no cliente antes do upload
- [ ] **Notificações in-app** — Sistema de alertas e novidades
- [ ] **Multiusuário por estabelecimento** — Convite de colaboradores (schema já modelado)
- [ ] **Empty state de ingredientes** — Tela visual quando catálogo está vazio
- [ ] **Padronizar toast** — Migrar todos os `useToast` para `sonner`
- [ ] **Modelos Gemini** — Revisar lista de fallback (gemini-2.0-flash, gemini-1.5-flash-8b podem retornar 404)
- [ ] **Testes E2E** — Cobertura de fluxos críticos (login, criar ficha, importar por IA)
