# Fase 2 — sequência de prompts para o Claude Code (MVP)

Pré-requisitos: Gate 1 aprovado, conta AWS, servidor de laboratório com MultiCortex OS e um motor
(vLLM ou Ollama) publicando endpoint compatível com OpenAI, contas de teste nos provedores em nuvem.

---

## M1 — Infraestrutura base

```
Crie deploy/terraform (AWS sa-east-1): VPC, EKS (ou ECS se preferirmos simplicidade), RDS PostgreSQL 16,
ElastiCache Redis 7, Secrets Manager, S3 + CloudFront para o frontend, ACM para TLS.
Crie deploy/compose/docker-compose.yml para nó único (laboratório MultiCortex): postgres, redis, keycloak,
control-plane, gateway, web, prometheus, grafana.
Nada de segredo versionado: .env.example documentado.
```

## M2 — Control plane

```
Implemente apps/control-plane (FastAPI + SQLAlchemy 2 + Alembic) cobrindo TODOS os endpoints /v1/admin/* e
/v1/public/waitlist de api/openapi.yaml, com o modelo de dados de docs/02-arquitetura.md seção 3.
Autenticação OIDC via Keycloak; autorização por perfil (admin, manager, user) — gestor só vê a própria equipe.
Toda ação administrativa gera audit_event. Chaves virtuais guardadas como hash.
Gere testes de contrato (schemathesis) contra o OpenAPI.
```

## M3 — Gateway e hooks

```
Implemente apps/gateway usando LiteLLM Proxy como motor e hooks próprios (docs/02-arquitetura.md seção 2):
pre_call: chave virtual → equipe → permissão (catálogo/equipe) → orçamento (Redis) → Presidio com reconhecedores
CPF/CNPJ/telefone BR → política "conteúdo sensível só local" (reescreve alias) → cache Redis (TTL da política).
post_call (assíncrono): tokens, custo pelo catálogo, economia vs. alias de referência, usage_event, contador de
orçamento, cabeçalhos x-dominance-*. Fallback por conector com timeout configurável.
Nunca gravar conteúdo de prompt. Testes de integração reproduzindo os 5 cenários de docs/01-produto.md.
```

## M4 — Adaptadores

```
Implemente gateway/adapters para: multicortex (vLLM/Ollama, compatível OpenAI), openai, anthropic, bedrock, vertex.
Deixe nvidia_nim e nutanix_ai como adaptadores "openai_compatible" configuráveis (sem homologação ainda).
Cada adaptador implementa health(): latência, modelos carregados e, quando exposto, uso do acelerador.
Para MultiCortex, leia métricas Prometheus do vLLM se disponíveis.
```

## M5 — Frontend no backend real

```
Aponte apps/governance-web e apps/workspace-web para o control plane e o gateway reais (VITE_API_BASE_URL), substitua MSW no Playground pelo
gateway real, mantenha o modo mock para demos. Login via Keycloak (OIDC PKCE) nos dois apps.
O apps/workspace-web passa a usar o gateway real, mostrando só os aliases permitidos para o usuário.
```

## M6 — Pacote instalável e piloto

```
Crie deploy/helm/dominance (values para saas, hybrid e sovereign) e um roteiro de instalação sobre MultiCortex OS
em menos de 1 dia (pré-requisitos, comandos, checklist de aceite inspirado na seção 16 do manual MultiCortex).
Modo soberano: sem telemetria externa, imagens assinadas (cosign), atualização por pacote offline.
Prepare painel de resultados do piloto: linha de base de custo antes × depois, % local, economia, bloqueios.
```
