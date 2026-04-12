# Contribuindo com a VibeCoding Security Skill

Obrigado por ajudar a tornar o desenvolvimento assistido por IA mais seguro! 🛡️

## Como Contribuir

### 1. Adicionando um Novo Padrão de Vulnerabilidade

Se você encontrou um padrão de vulnerabilidade que assistentes de IA introduzem frequentemente e que não está coberto, abra um PR com:

- **Qual arquivo atualizar**: Adicione ao arquivo `vulnerabilities/*.md` ou `stack/*.md` relevante
- **Exemplo real**: Mostre o padrão de código vulnerável
- **Cenário de exploração**: Explique o que um atacante poderia fazer (seja concreto, não abstrato)
- **Correção**: Mostre a versão segura como diff before/after  
- **Adicionar ao checklist**: Se justificar um novo check, adicione ao `checklists/*.md` e `core/audit-process.md` apropriados

### 2. Adicionando um Novo Módulo de Stack

Se há um framework ou serviço popular ainda não coberto (ex: PlanetScale, padrões específicos do Prisma, Clerk, etc.):

1. Crie `stack/<nome>.md` seguindo a estrutura existente
2. Atualize `SKILL.md` para referenciar o novo arquivo no Índice de Referências
3. Adicione a detecção de stack na tabela de detecção em `SKILL.md`
4. Adicione os checks relevantes ao `core/audit-process.md`

### 3. Melhorando um Módulo Existente

- Mantenha exemplos de código mínimos — mostre apenas o que ilustra a vulnerabilidade
- Sempre mostre o padrão ❌ vulnerável antes do padrão ✅ seguro
- Adicione labels de linguagem/framework nos code fences (ex: ` ```typescript `)
- Mantenha tabelas de técnicas de bypass precisas e baseadas em exploits reais

### 4. Corrigindo um Erro

Encontrou informação desatualizada, uma correção incorreta ou um padrão seguro melhor? Abra um PR explicando o que mudou e por quê.

---

## Idioma dos Arquivos

> ⚠️ **Importante**: Os arquivos da skill (`SKILL.md`, `core/`, `vulnerabilities/`, `stack/`, `production/`, `checklists/`) são escritos em **inglês** intencionalmente, pois é o idioma em que os modelos de linguagem têm melhor performance. Somente os arquivos voltados à comunidade (README, CONTRIBUTING, SECURITY, CHANGELOG) são em português.

---

## Convenções de Estrutura

```
vulnerabilities/     # Classes genéricas de vulnerabilidade (não específicas de framework)
stack/               # Padrões específicos de framework/serviço
production/          # Prontidão para produção e conformidade
checklists/          # Checklists executáveis que referenciam os acima
core/                # Orquestração central (não altere sem discussão)
```

## Checklist do Pull Request

- [ ] O exemplo vulnerável é efetivamente explorável (não teórico)
- [ ] A correção é efetivamente segura (não apenas levemente menos vulnerável)
- [ ] Exemplos de código possuem labels de linguagem
- [ ] Novos checks adicionados ao `core/audit-process.md` e ao `checklists/*.md` correto
- [ ] Nenhuma informação pessoal identificável nos exemplos
- [ ] Formatação consistente com os arquivos existentes (headers, tabelas, checklist no final)

## Código de Conduta

- Seja construtivo e específico — "este padrão é vulnerável porque X pode fazer Y", não "isso está errado"
- Dê crédito às fontes ao adicionar conteúdo de writeups de bug bounty ou pesquisas
- Não inclua payloads de exploit reais que possam prejudicar sistemas em produção

## Issues

Use GitHub Issues para:
- Reportar falsos positivos (um check que sinaliza código seguro)
- Reportar falsos negativos (um padrão de vulnerabilidade que estamos deixando passar)
- Sugerir novos módulos de stack
- Discutir mudanças arquiteturais na skill

Para problemas de segurança na própria skill, abra uma issue normal — esta é uma skill de documentação/prompt, não código executável.

---

Obrigado por contribuir! Cada adição torna apps vibe-coded um pouco mais seguros. 🚀
