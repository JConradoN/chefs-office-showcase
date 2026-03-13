# 🏗️ Arquitetura Frontend

> Chefs Office — React 18 + TypeScript + Vite

---

## Ponto de Entrada

**`src/main.tsx`** → monta `<App />` no `#root`

**`src/App.tsx`** — componente raiz com:
- `QueryClientProvider` (React Query) — cache de dados
- `TooltipProvider` + `Toaster` / `Sonner` — notificações
- `AppWithAuth` — lê sessão do usuário via `useAuth()`
- `EstablishmentProvider` — contexto global de estabelecimento
- `BrowserRouter` + `Routes` — roteamento declarativo

---

## Fluxo de Autenticação

```
Usuário acessa URL
     │
     ├─ Não autenticado → LandingPage (/)
     │         └─ Clica "Entrar" → /auth (Auth.tsx)
     │                   └─ Login/Cadastro → DashboardRedirect
     │
     └─ Autenticado
               └─ DashboardRedirect verifica profiles.full_name
                         ├─ full_name preenchido → /dashboard
                         └─ full_name vazio → /onboarding
```

### Componentes de Proteção de Rota

| Componente | Arquivo | Função |
|-----------|---------|--------|
| `ProtectedRoute` | `src/components/ProtectedRoute.tsx` | Verifica sessão Supabase ativa; redireciona para `/auth` se não autenticado |
| `AdminRoute` | `src/components/AdminRoute.tsx` | Verifica `profiles.is_admin = true`; redireciona para `/dashboard` se não admin |
| `DashboardRedirect` | `src/App.tsx` | Após login, verifica `full_name` e direciona para `/onboarding` ou `/dashboard` |

---

## Rotas da Aplicação

### Públicas

| Rota | Componente | Descrição |
|------|-----------|-----------|
| `/` | `LandingPage` | Página de entrada pública com vídeo institucional |
| `/auth` | `Auth` | Login (email/senha, Google, Apple) e cadastro |
| `/login` | `Navigate → /auth` | Alias de compatibilidade |
| `/termos` | `Termos` | Termos de uso |
| `/reset-password` | `ResetPassword` | Redefinição de senha via e-mail |
| `/onboarding` | `Onboarding` | Configuração inicial: nome + primeiro estabelecimento |

### Protegidas (requer sessão)

| Rota | Componente | Descrição |
|------|-----------|-----------|
| `/dashboard` | `Dashboard` | Visão geral com métricas (fichas, ingredientes) |
| `/fichas` | `Fichas` | Listagem de fichas técnicas do estabelecimento ativo |
| `/fichas/nova` | `NovaFicha` | Formulário de criação de nova ficha |
| `/fichas/:id` | `FichaDetalhe` | Detalhe, edição, cálculo de custos e exportação PDF |
| `/ingredientes` | `Ingredientes` | Catálogo de insumos com FC/FCC e preços |
| `/cardapios` | `Cardapios` | Gestão de cardápios e seções |
| `/estabelecimentos` | `Estabelecimentos` | CRUD de estabelecimentos |
| `/perfil` | `Perfil` | Perfil do usuário, dados pessoais, exclusão de conta |

### Admin (requer `is_admin = true`)

| Rota | Componente | Descrição |
|------|-----------|-----------|
| `/admin` | `Admin` | Listagem de usuários, stats gerais, toggle lifetime |

---

## Context: EstablishmentContext

**Arquivo:** `src/contexts/EstablishmentContext.tsx`

Gerencia qual estabelecimento está ativo globalmente. Persiste a escolha no `localStorage`.

```typescript
interface EstablishmentContextType {
  establishments: Establishment[];        // lista completa do usuário
  activeEstablishment: Establishment | null; // estabelecimento selecionado
  selectEstablishment: (est) => void;     // troca o ativo + persiste
  loading: boolean;
  refetch: () => void;                    // recarrega a lista
}
```

**Uso:**
```typescript
import { useEstablishment } from '@/contexts/EstablishmentContext';
const { activeEstablishment } = useEstablishment();
```

---

## Hooks Customizados

### `useAuth` — `src/hooks/useAuth.ts`

Lê e observa a sessão do Supabase Auth em tempo real.

```typescript
const { user, session, loading } = useAuth();
```

### `useEstablishment` — `src/hooks/useEstablishment.ts`

Re-exporta o contexto de estabelecimento com validação de provider.

### `use-mobile` — `src/hooks/use-mobile.tsx`

Detecta viewport mobile (`< 768px`) para renderização condicional.

---

## Layout da Aplicação

**`src/components/AppLayout.tsx`** — wrapper de todas as páginas protegidas:
- Sidebar fixa 240px à esquerda (desktop)
- Hamburger menu (mobile) via `AppSidebar` + `HamburgerButton`
- Área de conteúdo com `overflow-y-auto`

**`src/components/AppSidebar.tsx`** — sidebar com:
- Logo + branding
- Seletor de estabelecimento (dropdown)
- Links de navegação com highlight de rota ativa
- Avatar + nome do usuário
- Menu de perfil e logout

---

## Componentes Principais

### Fichas Técnicas

| Componente | Função |
|-----------|--------|
| `fichas/SmartRecipeImporter.tsx` | Upload de arquivo → Gemini AI → tela de revisão com badges IA |
| `fichas/AnaliseFinanceira.tsx` | Cálculo de custos, markup e precificação |
| `fichas/PrecificacaoSection.tsx` | Seção de preço de venda e markup |
| `fichas/ExportarFichaModal.tsx` | Modal de escolha de tipo de exportação PDF |
| `fichas/FichaPrintable.tsx` | Layout completo para impressão/PDF |
| `fichas/FichaPrintableGerencial.tsx` | Layout gerencial (custos) para PDF |
| `fichas/FichaPrintableReceita.tsx` | Layout de receita limpa para PDF |

### Ingredientes

| Componente | Função |
|-----------|--------|
| `IngredientModal.tsx` | Modal de criação de novo ingrediente |
| `IngredientEditModal.tsx` | Modal de edição de ingrediente existente |
| `ingredientes/TacoCatalogTab.tsx` | Catálogo TACO para importar dados nutricionais |
| `RecipeIngredients.tsx` | Lista de ingredientes dentro de uma ficha |

### Cardápios

| Componente | Função |
|-----------|--------|
| `cardapios/MenuRecipesSheet.tsx` | Sheet lateral com receitas de um cardápio |

### Utilidades

| Componente | Função |
|-----------|--------|
| `CategorySelect.tsx` | Select de categoria de ingrediente |
| `PreparationSteps.tsx` | Editor de etapas de preparo |
| `StatsBar.tsx` | Barra de estatísticas do dashboard |
| `WelcomeModal.tsx` | Modal de boas-vindas pós-onboarding |
| `OnboardingChecklist.tsx` | Checklist de primeiros passos |
| `NavLink.tsx` | Link de navegação estilizado |

---

## Gerenciamento de Estado

- **React Query** (`@tanstack/react-query`) — cache e fetch de dados do Supabase
- **useState / useEffect** — estado local de componentes
- **Context API** — estabelecimento ativo (global)
- **localStorage** — persistência do estabelecimento ativo entre sessões

---

## Tratamento de Notificações

Dois sistemas em paralelo:
1. **`sonner`** — toasts via `toast()` do pacote sonner
2. **shadcn `useToast`** — toasts legados em alguns componentes

> Padronizar para `sonner` em implementações futuras.
