# 🛡️ Ultra Thermo Nuclear VibeCoding Security Review

> Uma skill de segurança abrangente para assistentes de IA — combinando orientação preventiva e auditoria automatizada para projetos "vibe-coded".

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/versão-1.0-blue)](SKILL.md)
[![Checks](https://img.shields.io/badge/checks%20de%20segurança-50-green)](#cobertura-do-checklist)

---

## O Que É Isso?

Assistentes de IA (Copilot, Cursor, Claude, Codex) são incrivelmente produtivos — mas introduzem consistentemente as mesmas vulnerabilidades de segurança. Esta skill ensina seu agente de IA a **pensar como um engenheiro de segurança**, tanto enquanto escreve código quanto ao revisá-lo.

Esta skill foi sintetizada a partir das melhores ideias de três skills públicas de vibe-security ([LadyKerr](https://github.com/LadyKerr/Vibe-Security-Skill), [BehiSecc](https://github.com/BehiSecc/VibeSec-Skill), [raroque](https://github.com/raroque/vibe-security-skill)) e ampliada com novas áreas de cobertura.

---

## Dois Modos de Operação

| Modo | Quando | O Que Faz |
|------|--------|-----------|
| 🛡️ **PREVENTIVO** | Enquanto escreve código | Consulta a base de vulnerabilidades antes de gerar código que toque em auth, pagamentos, banco, secrets |
| 🔍 **AUDITORIA** | Em código existente | Executa checklist de 50 pontos, pontua o projeto de 0-100, gera relatório estruturado com diffs before/after |

---

## Cobertura do Checklist

**50 verificações de segurança** em três níveis de severidade:

### 🚨 Crítico (15) — Corrigir Antes de Implantar
Secrets expostos, bypass de autenticação, SQL injection, XSS, IDOR, CORS wildcard, RLS ausente, preços client-side, JWT alg:none, upload sem validação, SSRF, chave service_role exposta, containers como root, diretório .git acessível

### 🔍 Padrão (25) — Corrigir em 1 Semana
CSRF, rate limiting, mass assignment, open redirect, path traversal, XXE, validação de input, hashing de senhas, gerenciamento de sessão, introspection GraphQL, prompt injection, caps de uso de IA, verificação de webhooks, armazenamento seguro mobile, deep links, OAuth/PKCE, auth por mensagem em WebSocket, race conditions, prototype pollution, CVEs em dependências, vazamento de erros, CSP, validação de Server Actions, prefixos de cookies

### 🚀 Produção (10) — Antes de Ir ao Ar
Headers de segurança, source maps, endpoints debug, exclusão de conta (GDPR/LGPD), estratégia de backup, PII em logs, separação de ambientes, hardening de CI/CD, segurança de email (SPF/DKIM/DMARC), documentação da superfície de ataque

---

## Mapa de Cobertura

| Domínio | Cobertura |
|---------|-----------|
| 🔐 Autenticação (JWT, OAuth, PKCE, Sessões) | ✅ Profunda |
| 💉 Injeção (SQL, XSS, Command, XXE, Prototype Pollution) | ✅ Profunda |
| 🔑 Gestão de Secrets | ✅ Profunda |
| 🗄️ Supabase RLS | ✅ Profunda |
| 🔥 Firebase Security Rules | ✅ Profunda |
| ▲ Next.js (Server Actions, Middleware, App Router) | ✅ Profunda |
| 💳 Pagamentos Stripe | ✅ Profunda |
| 📱 React Native / Expo | ✅ Profunda |
| 🤖 Integração IA / LLM | ✅ Profunda |
| 🌐 GraphQL | ✅ Profunda |
| 🐳 Docker / CI-CD | ✅ Profunda |
| 🔒 Criptografia | ✅ Profunda |
| ⚡ Race Conditions | ✅ Profunda |
| 🏗️ CORS, CSP, Headers de Segurança | ✅ Profunda |
| 📋 Conformidade LGPD/GDPR | ✅ Profunda |
| 📊 Logging / Observabilidade | ✅ Profunda |
| 📦 Supply Chain / Dependências | ✅ Profunda |
| 🔗 CSRF, SSRF, Open Redirect | ✅ Profunda (com tabelas de bypass) |
| 📂 Segurança de Upload | ✅ Profunda (tabela de magic bytes) |
| 🌍 Controle de Acesso (IDOR, Escalação de Privilégio) | ✅ Profunda |

---

## Estrutura de Arquivos

```
ultra-thermo-nuclear-vibecoding-security-review/
├── SKILL.md                         # Manifesto principal — detecção de stack + roteamento
├── core/
│   ├── principles.md                # 7 pilares de segurança + anti-patterns de vibe coding
│   ├── audit-process.md             # Tabela de roteamento do checklist de 50 pontos
│   └── output-format.md             # Template de relatório com pontuação
├── vulnerabilities/
│   ├── injection.md                 # SQLi, XSS, Command Injection, XXE, Prototype Pollution
│   ├── access-control.md            # IDOR, Escalação de Privilégio, Mass Assignment
│   ├── authentication.md            # JWT, OAuth/PKCE, Senhas, Sessões, WebSocket
│   ├── csrf-ssrf.md                 # CSRF, SSRF (12 bypasses), Open Redirect (11 bypasses)
│   ├── file-upload.md               # Magic bytes, Polyglot, ZIP Slip, Path Traversal
│   ├── cryptography.md              # AES-GCM, Hashing, Gestão de Chaves, Reutilização de Nonce
│   └── race-conditions.md           # TOCTOU, Double-spend, Idempotência, Locks Distribuídos
├── stack/
│   ├── supabase.md                  # Políticas RLS, chave service_role, getUser vs getSession
│   ├── firebase.md                  # Regras Firestore, regras de Storage, verificação de ID token
│   ├── nextjs.md                    # Server Actions, Route Handlers, Middleware, variáveis de ambiente
│   ├── stripe-payments.md           # Manipulação de preço, verificação de assinatura de webhook
│   ├── react-native.md              # SecureStore, proxy de API, Deep links, env vars do Expo
│   ├── ai-llm.md                    # Proteção de chave de API, Prompt injection, Caps de uso
│   ├── graphql.md                   # Introspection, Limitação de profundidade, Complexidade, Batching
│   └── docker-cicd.md               # Containers não-root, permissões de pipeline, pinning de Actions
├── production/
│   ├── security-headers.md          # CSP com nonces, CORS, HSTS, conjunto completo de headers
│   ├── deployment.md                # Source maps, endpoints debug, .git, SPF/DKIM/DMARC
│   ├── compliance.md                # Exclusão de conta LGPD/GDPR, exportação de dados, backup
│   ├── observability.md             # Sanitização de PII, log injection, logging estruturado
│   └── dependency-audit.md          # Typosquatting, supply chain, lockfile, licenças
└── checklists/
    ├── critical-15.md               # 15 checks críticos com padrões regex
    ├── standard-25.md               # 25 checks padrão com referências cruzadas
    └── production-10.md             # 10 checks pré-deploy
```

---

## Instalação

### Antigravity (Gemini AI)
A skill é lida automaticamente de `~/.gemini/antigravity/skills/ultra-thermo-nuclear-vibecoding-security-review/`.

```bash
# Clone para o diretório de skills do Antigravity
git clone https://github.com/antoniojvrf/ultra-thermo-nuclear-vibecoding-security-review \
  ~/.gemini/antigravity/skills/ultra-thermo-nuclear-vibecoding-security-review
```

### Outros Agentes (Claude, Cursor, Copilot)
Copie o conteúdo do `SKILL.md` para o prompt de sistema ou arquivo de regras do seu agente (`.cursorrules`, `CLAUDE.md`, `.github/copilot-instructions.md`). Para a experiência modular completa, inclua os arquivos referenciados no contexto do agente.

---

## Como Usar

A skill ativa automaticamente quando o contexto envolve código relacionado à segurança. Você também pode ativá-la explicitamente:

```
"Faça uma auditoria de segurança neste projeto"
"Esse código é seguro para deploy?"
"Verifique meu setup do Supabase para vulnerabilidades"
"Revise este fluxo de pagamento quanto à segurança"
"Alguém pode hackear isso?"
```

---

## Exemplo de Output

```markdown
# 🛡️ VibeSec Security Report

**Score**: 64/100 — 🟡 Regular — precisa de atenção
**Problemas Encontrados**: 5 (2 Críticos, 2 Altos, 1 Médio)

## ⚡ Quick Wins (corrigíveis em < 10 minutos)
| # | Problema          | Severidade | Tempo | Correção                           |
|---|-------------------|------------|-------|------------------------------------|
| 1 | .env não ignorado | 10/10      | 1 min | echo ".env*" >> .gitignore         |

## 🚨 Problemas Críticos

### C1. `lib/supabase.ts:3` — Chave service_role exposta no bundle do cliente
**Impacto**: Qualquer pessoa pode ler, modificar ou deletar todos os dados do banco.

\`\`\`diff
- const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_SERVICE_KEY!)
+ const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!)
\`\`\`
> ⚠️ Rotacione a chave imediatamente em Supabase Dashboard → Settings → API.
```

---

## Contribuindo

Veja [CONTRIBUTING.md](CONTRIBUTING.md) para diretrizes sobre como adicionar novos padrões de vulnerabilidade, módulos de stack ou melhorar checks existentes.

## Créditos

Construído sobre os ombros de:
- [LadyKerr/Vibe-Security-Skill](https://github.com/LadyKerr/Vibe-Security-Skill) — formato de auditoria estruturada e pontuação
- [BehiSecc/VibeSec-Skill](https://github.com/BehiSecc/VibeSec-Skill) — profundidade ofensiva e tabelas de técnicas de bypass
- [raroque/vibe-security-skill](https://github.com/raroque/vibe-security-skill) — arquitetura modular e módulos específicos por stack

## Licença

[MIT](LICENSE) — Use livremente, contribua de volta.
