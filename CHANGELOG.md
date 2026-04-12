# Changelog

Todas as mudanças notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

## [1.0.0] - 2026-04-12

### Adicionado
- **SKILL.md** — Manifesto principal com operação dual-mode (PREVENTIVO + AUDITORIA), detecção automática de stack e índice de referências
- **core/principles.md** — 7 pilares de segurança e catálogo de anti-patterns introduzidos por IA
- **core/audit-process.md** — Tabela de roteamento do checklist de 50 pontos em três níveis de severidade
- **core/output-format.md** — Template de relatório padronizado com score 0-100, diffs before/after, seção Quick Wins
- **vulnerabilities/injection.md** — SQL injection, XSS (com fontes indiretas), Command injection, XXE (multi-linguagem), Prototype pollution
- **vulnerabilities/access-control.md** — IDOR, Escalação de privilégio, Mass assignment
- **vulnerabilities/authentication.md** — JWT (alg:none, confusão de algoritmo), OAuth 2.0 / PKCE, Hashing de senhas, Gerenciamento de sessão, Prefixos de cookies, Autenticação WebSocket
- **vulnerabilities/csrf-ssrf.md** — CSRF, SSRF com 12 técnicas de bypass de IP e DNS rebinding, Open redirect com 11 técnicas de bypass
- **vulnerabilities/file-upload.md** — Tabela de referência de magic bytes, arquivos polyglot, ZIP slip, path traversal, handler de upload seguro
- **vulnerabilities/cryptography.md** — Criptografia AES-GCM, algoritmos de hashing, gestão de chaves, reutilização de nonce, comparação timing-safe
- **vulnerabilities/race-conditions.md** — TOCTOU, double-spend, chaves de idempotência, locks distribuídos (Redis)
- **stack/supabase.md** — Padrões de políticas RLS (perigosos vs seguros), exposição de chave service_role, getUser vs getSession
- **stack/firebase.md** — Regras de segurança Firestore, regras de Storage, verificação de ID token server-side
- **stack/nextjs.md** — Server Actions (validação Zod + auth), Route Handlers, Middleware, escopo de variáveis de ambiente, configuração de headers
- **stack/stripe-payments.md** — Manipulação de preço client-side, verificação de assinatura de webhook, validação de status de assinatura
- **stack/react-native.md** — SecureStore vs AsyncStorage, proxy de chave de API, variáveis de ambiente Expo, validação de deep link
- **stack/ai-llm.md** — Proteção de chave de API server-side, rate limiting + caps de uso, prevenção de prompt injection, sanitização de output
- **stack/graphql.md** — Introspection, limitação de profundidade, análise de complexidade, prevenção de batching, autorização em resolvers
- **stack/docker-cicd.md** — Containers não-root, secrets em Dockerfile, pinning de imagens, permissões de GitHub Actions, pinning de Actions por SHA
- **production/security-headers.md** — Conjunto completo de headers, CSP com nonces (exemplo Next.js), configuração de allowlist CORS
- **production/deployment.md** — Source maps, endpoints debug, separação de ambientes, exposição de .git, segurança de email (SPF/DKIM/DMARC)
- **production/compliance.md** — Fluxo completo de exclusão de conta LGPD/GDPR, exportação de dados, gestão de consentimento, estratégia de backup
- **production/observability.md** — Sanitização de PII em logs, prevenção de log injection, logging estruturado com redação, monitoramento de eventos de segurança
- **production/dependency-audit.md** — Detecção de typosquatting, scanning de CVE, ataques de supply chain, integridade de lockfile, conformidade de licenças
- **checklists/critical-15.md** — 15 checks críticos com padrões regex e critérios de aprovação
- **checklists/standard-25.md** — 25 checks padrão com referências cruzadas para arquivos de módulo
- **checklists/production-10.md** — 10 checks de prontidão para produção

### Fontes
- Sintetizado a partir de [LadyKerr/Vibe-Security-Skill](https://github.com/LadyKerr/Vibe-Security-Skill), [BehiSecc/VibeSec-Skill](https://github.com/BehiSecc/VibeSec-Skill) e [raroque/vibe-security-skill](https://github.com/raroque/vibe-security-skill)
- Ampliado com nova cobertura: Criptografia, Race conditions, Docker/CI-CD, OAuth/PKCE, Autenticação WebSocket, Segurança de logging, Prefixos de cookies, Prototype pollution
