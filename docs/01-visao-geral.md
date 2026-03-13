# 📋 Chefs Office — Visão Geral do Projeto

> Atualizado: Março 2026

---

## O que é o Chefs Office?

O **Chefs Office** é uma plataforma SaaS de gestão culinária profissional criada por **Conrado Nogueira**, Chef e Consultor. Nasceu da necessidade real de centralizar fichas técnicas que ficavam espalhadas em PDFs, planilhas e e-mails — sem padronização, sem agilidade, sem inteligência.

### Princípios do Produto

| Princípio | Descrição |
|-----------|-----------|
| 🤝 **De Chef para Chef** | Interface e fluxo pensados para a realidade da cozinha profissional, sem burocracia |
| ⚡ **Simplicidade** | Cadastros rápidos, navegação direta, zero telas desnecessárias |
| 🤖 **IA Integrada** | Importação automática de receitas a partir de textos, PDFs e planilhas via Gemini AI |
| 📋 **Organização Total** | Fichas técnicas, ingredientes, cardápios e múltiplos estabelecimentos centralizados |

---

## Status Atual das Funcionalidades

| Área | Status | Observações |
|------|--------|-------------|
| Autenticação | ✅ Funcional | Email/senha, Google OAuth, Apple OAuth |
| Onboarding | ✅ Funcional | Criação de perfil + primeiro estabelecimento |
| Dashboard | ✅ Funcional | Métricas de fichas e ingredientes |
| Fichas Técnicas | ✅ Funcional | CRUD completo + cálculo de custos automático |
| Ingredientes | ✅ Funcional | CRUD manual completo com FC/FCC |
| Importação de Fichas por IA | ✅ Funcional | Edge Function `process-recipe-etl` + Gemini API |
| Detecção de etapas inferidas | ✅ Funcional | Badge "IA" em etapas criadas pela IA no importer |
| Cardápios | ✅ Funcional | Gestão de menus por seções |
| Estabelecimentos | ✅ Funcional | Multi-estabelecimento |
| Exportação PDF | ✅ Funcional | Ficha técnica, gerencial e receita |
| Responsividade Mobile | ✅ Funcional | Hamburger menu, grids responsivos |
| Landing Page | ✅ Funcional | Hero + vídeo institucional + Diferenciais + Sobre + Planos |
| Painel Admin | ✅ Funcional | Listagem de usuários, stats, toggle lifetime |
| Importação de Ingredientes | ❌ Removido | Funcionalidade retirada — apenas cadastro manual |
| Plano Pro / Billing | ⏳ Em Breve | Cards de pricing exibidos, funcionalidade pendente |

---

## Stack Técnica

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18 + TypeScript + Vite |
| Estilização | Tailwind CSS + shadcn/ui |
| Backend | Supabase (PostgreSQL + Auth + Storage + Edge Functions) |
| IA | Google Gemini API via Edge Function (`process-recipe-etl`) |
| PDF | jsPDF + html2canvas |
| Deploy | Lovable Cloud |
| Testes | Vitest |

---

## Identidade Visual

| Token | Valor |
|-------|-------|
| Fundo principal | `hsl(0 0% 4%)` — `#0A0A0A` |
| Sidebar | `hsl(0 0% 8%)` — `#141414` |
| Cards | `hsl(0 0% 11%)` — `#1A1A1A` |
| Bordas | `hsl(0 0% 14%)` — `#242424` |
| Accent dourado (primary) | `hsl(43 84% 41%)` — `#D4A017` |
| Texto principal | `hsl(40 20% 92%)` |
| Texto secundário | `hsl(0 0% 50%)` |
| Badge IA (inferido) | `hsl(260 60% 50%)` — roxo |

**Fonte:** Plus Jakarta Sans (Google Fonts)

---

## URLs do Projeto

| Ambiente | URL |
|----------|-----|
| Preview (dev) | `[AMBIENTE DE PREVIEW INTERNO]` |
| Produção | `https://www.chefsoffice.com.br` |

---

## Estrutura de Pastas

```
src/
├── components/           # Componentes reutilizáveis
│   ├── fichas/           # Componentes de fichas técnicas (SmartImporter, PDF, etc.)
│   ├── cardapios/        # Componentes de cardápios
│   ├── ingredientes/     # Componentes de ingredientes (TacoCatalogTab)
│   └── ui/               # shadcn/ui + componentes base
├── contexts/             # Context API (EstablishmentContext)
├── hooks/                # Hooks customizados (useAuth, useEstablishment)
├── integrations/         # Cliente Supabase e tipos gerados
│   └── supabase/
│       ├── client.ts     # createClient configurado
│       └── types.ts      # Tipos gerados pelo schema (READ-ONLY)
├── pages/                # Páginas da aplicação
└── lib/                  # Utilitários (cn, formatters)

supabase/
├── functions/            # Edge Functions Deno
│   ├── process-recipe-etl/   # Importação de fichas por IA (Gemini)
│   ├── delete-user/          # Exclusão de conta
│   └── import-ingredients-etl/ # ⚠️ Deployada mas não utilizada
└── migrations/           # Histórico de migrações SQL (READ-ONLY)

public/
└── videos/
    └── intro-chefs-office.mp4  # Vídeo institucional da landing page

docs/                     # Esta documentação
```
