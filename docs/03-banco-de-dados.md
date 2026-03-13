# 🗄️ Banco de Dados — Supabase/PostgreSQL

> ⚠️ **CRÍTICO:** O schema é fixo. NUNCA criar ou modificar tabelas pelo frontend.  
> Todas as migrações ficam em `supabase/migrations/` (arquivos READ-ONLY).

---

## Tabelas Principais

### `profiles`

Criado automaticamente via trigger `handle_new_user()` ao registrar um usuário no Auth.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | Espelha `auth.users.id` |
| `full_name` | text | Nome completo — obrigatório para acessar o app |
| `email` | text | E-mail |
| `avatar_url` | text | URL do avatar |
| `plan` | text | `'free'` ou `'pro'` (padrão: `'free'`) |
| `is_admin` | boolean | Flag de administrador (padrão: `false`) |
| `is_lifetime` | boolean | Acesso vitalício (padrão: `false`) |
| `onboarding_completed` | boolean | Controle do onboarding |
| `role` | text | Role textual (uso futuro) |
| `phone` | text | Telefone |
| `document` | text | CPF/CNPJ |
| `document_type` | text | Tipo do documento |
| `address_*` | text | Campos de endereço |

**RLS:** Usuário acessa apenas seu próprio registro (`id = auth.uid()`).

> ⚠️ **Type casting necessário:** Ao fazer `update` em `profiles`, usar `as any` para campos como `role` e `onboarding_completed` devido a discrepâncias com os tipos gerados.

---

### `establishments`

Estabelecimentos (restaurantes, consultorias) pertencentes a um usuário.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `owner_id` | uuid | FK → `profiles.id` |
| `name` | text | Nome do estabelecimento |
| `logo_url` | text | Logo (bucket `establishment-logos`) |
| `is_active` | boolean | Ativo/inativo (soft delete) |
| `type` | text | Tipo do negócio |
| `city` | text | Cidade |
| `state` | text | Estado |
| `description` | text | Descrição |

**RLS:** Acesso restrito ao `owner_id = auth.uid()`.

---

### `establishment_members`

Membros adicionais de um estabelecimento (para futura funcionalidade multiusuário).

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `establishment_id` | uuid | FK → `establishments` |
| `user_id` | uuid | FK → `profiles` |
| `role` | text | Role do membro |
| `invited_by` | uuid | FK → `profiles` (quem convidou) |

**RLS:** Usuário acessa apenas seus próprios registros de membro.

---

### `ingredients`

Catálogo de insumos do usuário. **Não existem ingredientes globais/template** — todos pertencem ao usuário que os criou.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `user_id` | uuid | Dono do ingrediente |
| `name` | text | Nome do insumo |
| `package_unit` | text | Unidade da embalagem (FK → `ingredient_units.code`) |
| `package_quantity` | numeric | Quantidade na embalagem |
| `fc` | numeric | Fator de Correção (padrão: 1.0) |
| `fcc` | numeric | Fator de Cocção (padrão: 1.0) |
| `category` | text | Categoria do insumo |
| `calories`, `proteins`, `carbs`, `fats`, `fiber` | numeric | Dados nutricionais resumidos |

**RLS:** Leitura e escrita apenas do próprio catálogo (`user_id = auth.uid()`).

> ⚠️ A função `seed_user_ingredients` **existe no banco mas não é chamada** pelo frontend. O usuário começa com catálogo vazio.

---

### `establishment_ingredients`

Preço de mercado de cada insumo por estabelecimento.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `establishment_id` | uuid | FK → `establishments` |
| `ingredient_id` | uuid | FK → `ingredients` |
| `package_price` | numeric | Preço da embalagem |
| `supplier` | text | Fornecedor |
| `notes` | text | Observações |

**RLS:** Apenas o dono do estabelecimento.

> ⚡ **Trigger:** `trg_recalc_on_price_change` — recalcula automaticamente os custos das receitas quando o preço de um insumo é alterado.

---

### `recipes`

Fichas técnicas (receitas).

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `establishment_id` | uuid | FK → `establishments` |
| `created_by` | uuid | FK → `profiles` |
| `name` | text | Nome da receita |
| `type` | text | `'receita'` ou `'ficha_tecnica_interna'` |
| `photo_url` | text | **OBRIGATÓRIO** — URL da foto no bucket |
| `yield_quantity` | numeric | Rendimento total |
| `yield_unit` | text | Unidade do rendimento (FK → `ingredient_units.code`) |
| `yield_portions` | numeric | Número de porções |
| `total_cost_cached` | numeric | Custo total (recalculado por trigger) |
| `cost_per_portion_cached` | numeric | Custo por porção (cache) |
| `selling_price` | numeric | Preço de venda sugerido |
| `markup_percent` | numeric | Markup (padrão: 30%) |
| `category` | text | Categoria |
| `notes` | text | Observações |
| `is_active` | boolean | Ativo/inativo |
| `portion_size` | numeric | Tamanho da porção |
| `nutritional_info_cached` | jsonb | Dados nutricionais agregados (cache) |

**RLS:** Acesso via `is_establishment_member(establishment_id)`.

---

### `recipe_ingredients`

Ingredientes vinculados a uma receita.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `recipe_id` | uuid | FK → `recipes` |
| `ingredient_id` | uuid | FK → `ingredients` (nullable) |
| `sub_recipe_id` | uuid | FK → `recipes` (para FTI — Ficha Técnica Interna) |
| `name` | text | Nome do ingrediente (desnormalizado) |
| `quantity` | numeric | Quantidade na receita |
| `unit` | text | Unidade (FK → `ingredient_units.code`) |
| `fc` | numeric | FC aplicado |
| `fcc` | numeric | FCC aplicado |
| `unit_cost` | numeric | Custo calculado da linha |
| `position` | integer | Ordem na listagem |
| `preparation` | text | Forma de preparo do ingrediente |
| `notes` | text | Observações |

---

### `preparation_steps`

Etapas de preparo de uma receita.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `recipe_id` | uuid | FK → `recipes` |
| `phase` | text | `'pre_preparo'`, `'preparo'`, `'finalizacao'`, `'empratamento'` |
| `position` | integer | Ordem de exibição |
| `description` | text | Descrição da etapa |
| `duration_min` | integer | Duração estimada em minutos |
| `notes` | text | Observações adicionais |

---

### `menus` / `menu_sections` / `menu_recipes`

Sistema de cardápios com seções configuráveis.

**`menus`** — Cardápio vinculado a um estabelecimento:
- `id`, `establishment_id`, `chef_responsible_id`, `name`, `type`, `description`, `is_active`

**`menu_sections`** — Seções do cardápio (Entrada, Prato Principal, etc.):
- `id`, `establishment_id`, `name`, `position`, `is_active`

**`menu_recipes`** — Receitas dentro de cada seção:
- `id`, `menu_id`, `menu_section_id`, `recipe_id`, `chef_responsible_id`, `price_sale`, `price_cost`, `position`, `is_active`

> ⚠️ **Type casting:** `menu_recipes` requer `chef_responsible_id` obrigatório. Usar `as any` ao inserir se necessário.

---

### `ingredient_units`

Tabela de referência de unidades de medida (somente leitura para usuários autenticados).

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `code` | text (PK) | Ex: `'g'`, `'kg'`, `'ml'`, `'l'`, `'un'`, `'porcao'`, `'ft'` |
| `name` | text | Nome legível |
| `type` | text | `'massa'`, `'volume'`, `'unidade'` |
| `base_unit` | text | Unidade base para conversão |
| `multiplier` | numeric | Fator de conversão para a base |

---

### `nutritional_data`

Dados nutricionais detalhados por ingrediente.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `ingredient_id` | uuid | FK → `ingredients` |
| `per_quantity` | numeric | Quantidade de referência |
| `per_unit` | text | Unidade de referência (FK → `ingredient_units`) |
| `calories_kcal`, `proteins_g`, `carbohydrates_g` | numeric | Macronutrientes |
| `fats_g`, `saturated_fats_g`, `fiber_g`, `sodium_mg`, `sugar_g` | numeric | Outros nutrientes |
| `source` | text | Fonte (ex: `'TACO'`) |

---

### `recipe_categories`

Categorias customizadas de receitas por estabelecimento.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid (PK) | |
| `establishment_id` | uuid | FK → `establishments` |
| `name` | text | Nome da categoria |

---

## Funções de Banco (SECURITY DEFINER)

| Função | Descrição |
|--------|-----------|
| `is_establishment_member(id)` | Verifica se o usuário é membro ou dono do estabelecimento — usada nas policies RLS |
| `refresh_recipe_total_cost(id)` | Recalcula `total_cost_cached` e `cost_per_portion_cached` de uma receita |
| `calc_ingredient_line_cost(...)` | Calcula o custo de uma linha de ingrediente com conversão de unidades |
| `get_unit_base(code)` | Retorna a unidade base de uma unidade (ex: `kg` → `g`) |
| `get_unit_factor(code)` | Retorna o fator de conversão (ex: `kg` → `1000`) |
| `propagate_fti_cost_changes(est_id)` | Propaga alterações de custo de FTIs para fichas-mãe |
| `get_admin_dashboard_stats()` | Retorna estatísticas gerais para o painel admin |
| `get_admin_users_list()` | Retorna lista completa de usuários para o painel admin |
| `admin_set_lifetime(user_id, value)` | Alterna o acesso vitalício de um usuário |
| `seed_user_ingredients(user_id)` | ⚠️ Existe mas **não é chamada** pelo frontend |
| `create_default_menu_sections(est_id)` | Cria seções padrão ao criar um cardápio |
| `import_taco_ingredient(taco_id, user_id)` | Copia um ingrediente TACO para o catálogo do usuário |
| `search_ingredients_with_taco_fallback(...)` | Busca fuzzy no catálogo do usuário com fallback TACO |
| `search_similar_ingredients(...)` | Busca fuzzy apenas no catálogo do usuário |

---

## Triggers Automáticos

| Trigger | Tabela | Ação |
|---------|--------|------|
| `handle_new_user` | `auth.users` | Cria registro em `profiles` ao registrar usuário |
| `trg_recalc_on_price_change` | `establishment_ingredients` | Recalcula custos das receitas ao alterar preço de insumo |

---

## Storage Buckets

| Bucket | Público | Uso |
|--------|---------|-----|
| `recipe-photos` | Sim | Fotos das fichas técnicas |
| `avatars` | Sim | Avatares dos usuários |
| `establishment-logos` | Sim | Logos dos estabelecimentos |

> ⚠️ **Pendência de segurança:** Os buckets são públicos com políticas de escrita permissivas. Recomendado tornar privados e usar `createSignedUrl()` para exibição futura.
