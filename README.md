# chefs-office-showcase
Plataforma de gestão de fichas técnicas para chefs
# 🍳 Chefs Office

> SaaS para gestão técnica e financeira de cozinhas profissionais — desenvolvido por um chef com 8 anos de experiência e background em TI.

🔗 **[chefsoffice.com.br](https://www.chefsoffice.com.br)** — acesso gratuito disponível

---

## Sobre o projeto

Chefs Office é um sistema web completo para gestão de fichas técnicas gastronômicas. O produto resolve um problema real do mercado: chefs e donos de restaurante gerenciam receitas, custos e precificação em planilhas e papel — um processo lento, propenso a erros e sem escala.

O sistema foi concebido, projetado e desenvolvido por mim como produto solo, da arquitetura ao deploy, combinando minha experiência em cozinha profissional com background em infraestrutura de TI e desenvolvimento com IA.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | React 18 + TypeScript + Tailwind CSS + shadcn/ui |
| Estado | TanStack Query v5 |
| Backend | Supabase (PostgreSQL + Auth + Storage + Edge Functions) |
| IA | Gemini 2.0 Flash via Edge Function (Deno) |
| PDF | jsPDF + html2canvas |
| Deploy | Lovable (CI/CD automatizado) |
| Automações | n8n (self-hosted, zero custo) |
| Workflow de dev | Claude Code + Gemini + Lovable |

---

## Funcionalidades

### Fichas Técnicas
- CRUD completo de receitas com foto obrigatória
- Tipos: Ficha Técnica de Produção (FTP) e Ficha Técnica de Ingrediente (FTI)
- Sub-receitas encadeadas: FTI como ingrediente de FTP com propagação automática de custo
- Exportação em PDF profissional (PDF Receita e PDF Gerencial)
- Etapas de preparo com 4 fases: pré-preparo, preparo, finalização e empratamento

### Importação com IA
- Upload de fichas em PDF, DOCX, XLSX, PNG ou JPG
- Extração automática de ingredientes, quantidades e etapas via Gemini 2.0 Flash
- Fallback automático para modelos alternativos em caso de indisponibilidade
- Etapas inferidas pela IA identificadas com badge visual para revisão do usuário
- SmartImporter com resolução em 4 camadas: catálogo do usuário → catálogo global → sub-receitas → criação inline

### Ingredientes
- Catálogo global com **320 ingredientes** baseados na Tabela TACO (UNICAMP) + USDA
- FC (Fator de Correção) e FCC (Fator de Cocção) aplicados por categoria
- Dados nutricionais completos: calorias, proteínas, carboidratos, gorduras e fibras
- Catálogo por estabelecimento com preço de embalagem e fornecedor
- Importação com 1 clique do catálogo TACO para o catálogo do usuário

### Custos e Precificação
- Cálculo automático de CMV%
- Modelo de markup por fator direto (custo × fator)
- Análise financeira completa: custo por porção, preço de venda, margem
- Comparativo de cenários no PDF Gerencial

### Multi-estabelecimento
- Um usuário pode gerenciar múltiplos estabelecimentos
- Preços de ingredientes independentes por estabelecimento
- Cardápios com seções e receitas vinculadas

### Autenticação e Onboarding
- Auth completo com Supabase (email/senha + Google OAuth)
- Fluxo de onboarding guiado com criação de estabelecimento
- Planos: Free / Pro / Enterprise

---

## Arquitetura de banco de dados

```
profiles
establishments ──── establishment_members
                └── establishment_ingredients ──── ingredients (catálogo global: user_id IS NULL)
                                                └── ingredients (catálogo usuário: user_id = X)
recipes ──── recipe_ingredients ──── ingredients
         └── preparation_steps      └── sub_recipe_id → recipes (FTI encadeado)
menus ──── menu_sections ──── menu_recipes ──── recipes
```

Principais decisões de arquitetura:
- Catálogo global (`user_id IS NULL`) é somente leitura — importado via RPC para o catálogo do usuário antes de ser utilizado em fichas
- Constraint de banco garante que `recipe_ingredients` nunca referencie ingrediente sem dono
- RPCs com similaridade de texto (`pg_trgm`) para busca inteligente no catálogo
- Edge Functions em Deno para processamento de IA sem expor chave no cliente

---

## Workflow de desenvolvimento

Este projeto foi desenvolvido com um workflow **AI-first** deliberadamente estruturado:

- **Gemini** → arquitetura, infraestrutura, decisões de schema
- **Claude / Claude Code** → lógica complexa, SQL, RPCs, Edge Functions, CLAUDE.md
- **Lovable** → geração de UI via prompt (React/Tailwind), CI/CD automatizado
- **n8n** (self-hosted) → automações de notificação e alertas, zero custo operacional
- **Notion** → hub de documentação e gestão do produto

---

## Custo de infraestrutura (produção)

| Serviço | Plano | Custo |
|---|---|---|
| Supabase | Free tier | R$ 0 |
| Lovable | Starter | ~R$ 100/mês |
| Gemini API | Pay-as-you-go | ~R$ 7/mês |
| Domínio | chefsoffice.com.br | ~R$ 50/ano |
| n8n | Self-hosted (VPS própria) | R$ 0 |
| **Total** | | **~R$ 110/mês** |

---

## Sobre o desenvolvedor

**Conrado Nogueira** — [github.com/JConradoN](https://github.com/JConradoN)

Background multidisciplinar: 8 anos como chef profissional + carreira em TI (infraestrutura, redes, segurança) + desenvolvimento com IA.

Especialidades: Python · SQL · Linux · Docker · n8n · RAG · Supabase · React · Claude Code · Lovable · Infraestrutura (Squid, Snort, Apache, DNS, LDAP, FreeBSD)

Disponível para projetos freelance remotos — preferencialmente em USD.

---

*Repositório de vitrine — código fonte completo disponível mediante solicitação para avaliação técnica.*
