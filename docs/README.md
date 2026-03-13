# 📚 Documentação — Chefs Office

> Documentação técnica completa do projeto. Gerada em: Março 2026.

---

## Índice

| Arquivo | Conteúdo |
|---------|---------|
| [01-visao-geral.md](./01-visao-geral.md) | Status do projeto, stack, identidade visual, estrutura de pastas |
| [02-arquitetura-frontend.md](./02-arquitetura-frontend.md) | Fluxo de auth, rotas, context, hooks, componentes, layout |
| [03-banco-de-dados.md](./03-banco-de-dados.md) | Todas as tabelas, colunas, RLS, triggers, funções, buckets |
| [04-edge-functions.md](./04-edge-functions.md) | process-recipe-etl (Gemini AI), delete-user, deploy |
| [05-guia-desenvolvimento.md](./05-guia-desenvolvimento.md) | Setup local, regras obrigatórias, padrões de código, pendências |
| [06-funcionalidades-detalhadas.md](./06-funcionalidades-detalhadas.md) | Fichas, ingredientes, cardápios, auth, admin, landing page |

---

## Quick Reference

### Comandos

```bash
npm run dev      # desenvolvimento (porta 8080)
npm run build    # build de produção
npm test         # testes Vitest
```

### Variáveis de Ambiente

```env
VITE_SUPABASE_URL=...
VITE_SUPABASE_PUBLISHABLE_KEY=...
```

### Regras de Ouro

1. ❌ **Nunca** criar/modificar tabelas no banco
2. 🇧🇷 Interface sempre em **Português do Brasil**
3. 📸 `photo_url` é **obrigatória** em receitas
4. 🎨 Usar apenas **tokens semânticos** do design system — sem cores diretas
5. 🔒 `is_admin` sempre validado no servidor — nunca via localStorage
