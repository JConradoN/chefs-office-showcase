# 🍳 Funcionalidades Detalhadas

---

## 1. Fichas Técnicas

### Criação Manual (`/fichas/nova`)

**Componente:** `src/pages/NovaFicha.tsx`

Fluxo:
1. Selecionar tipo: **Receita** (prato final) ou **Ficha Técnica Interna** (base/molho)
2. Definir nome, categoria, rendimento (quantidade + unidade + porções)
3. Fazer upload da foto (**obrigatório**)
4. Adicionar ingredientes com quantidade, unidade e FC
5. Adicionar etapas de preparo por fase
6. Salvar → custo calculado automaticamente pelo banco

### Edição e Detalhe (`/fichas/:id`)

**Componente:** `src/pages/FichaDetalhe.tsx`

Funcionalidades:
- Edição inline de todos os campos
- Visualização do custo total e custo por porção
- Seção de precificação (markup + preço de venda sugerido)
- Análise financeira
- Exportação em PDF (3 modelos)
- Vinculação de Sub-receitas (FTI)

### Importação por IA (`SmartRecipeImporter`)

**Componente:** `src/components/fichas/SmartRecipeImporter.tsx`

Fluxo:
1. Chef faz upload de arquivo (PDF, DOCX, XLSX, imagem)
2. Arquivo é enviado para Edge Function `process-recipe-etl`
3. Gemini extrai nome, rendimento, ingredientes e etapas
4. Tela de revisão exibe os dados extraídos:
   - Etapas com badge **"IA"** (roxo) indicam conteúdo inferido
   - Legenda de aviso aparece quando há etapas inferidas
5. Chef revisa, edita e confirma
6. Ficha é criada via upsert (`name + establishment_id`)

### Exportação PDF

3 modelos disponíveis:
- **Ficha Técnica Completa** (`FichaPrintable`) — todos os dados
- **Gerencial** (`FichaPrintableGerencial`) — foco em custos e financeiro
- **Receita** (`FichaPrintableReceita`) — modo de preparo limpo para a cozinha

---

## 2. Ingredientes (`/ingredientes`)

**Componente:** `src/pages/Ingredientes.tsx`

Funcionalidades:
- Listagem com busca e filtro por categoria
- Criação manual via modal (`IngredientModal`)
- Edição via modal (`IngredientEditModal`)
- Exclusão com confirmação
- Cadastro de preço por estabelecimento (`establishment_ingredients`)
- FC (Fator de Correção) e FCC (Fator de Cocção) configuráveis
- Aba de Catálogo TACO (`TacoCatalogTab`) para importar dados nutricionais

### Fator de Correção (FC)

FC = Peso Bruto / Peso Líquido

Exemplo: 1 kg de cebola rende 800g após descascar → FC = 1,25

### Fator de Cocção (FCC)

FCC = Peso Cozido / Peso Cru

Exemplo: 100g de frango cru rende 72g cozido → FCC = 0,72

---

## 3. Cardápios (`/cardapios`)

**Componente:** `src/pages/Cardapios.tsx`

Estrutura:
```
Cardápio
└── Seção (ex: "Entradas", "Pratos Principais", "Sobremesas")
    └── Receitas (com preço de venda e custo)
```

Funcionalidades:
- CRUD de cardápios
- Gerenciamento de seções por estabelecimento
- Adição de receitas às seções
- Visualização de custo vs. preço de venda por item

---

## 4. Estabelecimentos (`/estabelecimentos`)

**Componente:** `src/pages/Estabelecimentos.tsx`

Funcionalidades:
- CRUD de estabelecimentos
- Upload de logo (bucket `establishment-logos`)
- Campos: nome, tipo, cidade, estado, descrição
- Ativação/desativação (soft delete via `is_active`)
- Seletor na sidebar para trocar o estabelecimento ativo

---

## 5. Dashboard (`/dashboard`)

**Componente:** `src/pages/Dashboard.tsx`

Exibe métricas em tempo real:
- Total de fichas técnicas
- Total de ingredientes cadastrados
- Fichas por tipo (receita vs. FTI)
- Custo médio por porção
- Checklist de onboarding (`OnboardingChecklist`)

---

## 6. Autenticação (`/auth`)

**Componente:** `src/pages/Auth.tsx`

Métodos disponíveis:
- **E-mail + Senha** — cadastro e login local
- **Google OAuth** — login social
- **Apple OAuth** — login social

Fluxo pós-login:
1. `DashboardRedirect` verifica `profiles.full_name`
2. Se vazio → `/onboarding`
3. Se preenchido → `/dashboard`

---

## 7. Onboarding (`/onboarding`)

**Componente:** `src/pages/Onboarding.tsx`

Passos:
1. Informar nome completo (salva em `profiles.full_name`)
2. Criar primeiro estabelecimento
3. Marcar `onboarding_completed = true`
4. Redirecionar para `/dashboard`

---

## 8. Perfil (`/perfil`)

**Componente:** `src/pages/Perfil.tsx`

Funcionalidades:
- Edição de dados pessoais (nome, telefone, documento, endereço)
- Upload de avatar (bucket `avatars`)
- Visualização do plano atual
- Exclusão permanente de conta (via Edge Function `delete-user`)

---

## 9. Painel Admin (`/admin`)

**Componente:** `src/pages/Admin.tsx`

**Acesso:** Requer `profiles.is_admin = true`

Funcionalidades:
- Listagem de todos os usuários com email, nome, plano e data de cadastro
- Contagem de receitas por usuário
- Toggle de acesso lifetime (`admin_set_lifetime`)
- Estatísticas gerais via `get_admin_dashboard_stats()`

---

## 10. Landing Page (`/`)

**Componente:** `src/pages/LandingPage.tsx`

Seções:
1. **Navbar** — logo, links de navegação, botões de CTA
2. **Hero** — headline principal + vídeo institucional (`/videos/intro-chefs-office.mp4`)
3. **Diferenciais** — 3 cards de features principais
4. **A Origem** — lista de benefícios com checkmarks
5. **CTA** — chamada para ação
6. **Roadmap** — 4 features "Em breve" (Simulações Financeiras, Inteligência de Insumos, Relatórios Nutricionais, Assistente Criativo IA)
7. **Planos** — Free (ativo) e Chef Pro (em breve)
8. **Footer** — copyright e links legais

Usuários já autenticados são redirecionados automaticamente para `/dashboard`.
