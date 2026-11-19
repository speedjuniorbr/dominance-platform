# DOMINANCE — instruções do projeto para o Claude Code

Leia este arquivo inteiro antes de qualquer tarefa. Ele vale para todas as fases.

## O que é o DOMINANCE

Plataforma de IA soberana da AxiosTech Solutions com o CRTC.ai, organizada em três camadas sob a
marca-mãe **DOMINANCE® Platform**:

| Camada | Tipo | O que é |
|---|---|---|
| DOMINANCE Workspace | Software | Interface do usuário final: chat privado, agentes e conectores |
| DOMINANCE Governance | Software | Plano de controle: gateway único, políticas, orçamentos, guardrails, auditoria |
| DOMINANCE Core | Hardware | Rack air-gapped montado com equipamentos de mercado (NVIDIA, Supermicro, Nutanix etc.) |

Todo pedido de qualquer usuário ou aplicativo a qualquer modelo passa pelo gateway do Governance, que
aplica identidade, permissão, orçamento, políticas de dados, cache, roteamento semântico e medição.
O software roda como SaaS (AWS), híbrido ou soberano (instalado no cliente, inclusive sem internet);
o modelo de implantação comercial ainda está em definição. A base de inferência local de referência é
o MultiCortex OS.

Documentos de referência, nesta ordem:
1. `docs/01-produto.md`: escopo, perfis, telas, regras de negócio.
2. `docs/02-arquitetura.md`: componentes, adaptadores, modos de implantação.
3. `docs/03-roadmap-e-gates.md`: fases, entregas e critérios de aceite.
4. `api/openapi.yaml`: contrato da API. **Fonte da verdade.**
5. `prototypes/governance/index.html` e `prototypes/workspace/index.html`: protótipos navegáveis aprovados
   (referência de UX, textos e fluxos).
6. `docs/04-identidade-visual.md` e `brand/`: marcas, cores, tipografia e regras de uso.

## Princípios inegociáveis

1. **Contrato primeiro.** Toda tela consome a API descrita em `api/openapi.yaml`. Nada de chamar dados
   "por fora". Mudou a necessidade? Altere primeiro o OpenAPI, depois o código. Tipos do frontend são
   gerados a partir do contrato (`openapi-typescript`), nunca escritos à mão.
2. **Sem lock-in de base de inferência.** O gateway fala com qualquer base por adaptadores
   (`ProviderType`). MultiCortex, NVIDIA NIM, Nutanix Enterprise AI, Ollama e vLLM são bases locais
   compatíveis com OpenAI. Nenhum código fora de `gateway/adapters/` pode conhecer um provedor específico.
3. **Um produto, três implantações.** O mesmo código e as mesmas imagens de contêiner rodam em SaaS,
   híbrido e soberano. Diferenças só por configuração (variáveis de ambiente / Helm values).
4. **Segredos nunca em texto.** Chaves de provedores são referências (`vault://`, `aws-sm://`, `env://`).
   Chaves virtuais são guardadas como hash (argon2 ou SHA-256 com pepper); o segredo só aparece uma vez.
   Nada de segredo em log, em fixture versionada ou em mensagem de erro.
5. **Privacidade por padrão (LGPD).** A auditoria guarda metadados (quem, quando, alias, tokens, custo,
   políticas), nunca o texto do prompt, a menos que uma política explícita diga o contrário.
6. **Português do Brasil na interface.** Textos curtos, voz ativa, botões dizem exatamente o que fazem,
   erros dizem o que houve e como resolver. Código, nomes de variáveis e commits em inglês.
7. **Fase 1 é reaproveitável.** O frontend da Fase 1 roda contra um mock gerado do OpenAPI. Na Fase 2,
   só a URL base muda. Não escreva lógica de negócio no frontend que deveria estar no backend.

## Stack

| Camada | Escolha |
|---|---|
| Monorepo | pnpm workspaces (`apps/governance-web`, `apps/workspace-web`, `apps/control-plane`, `apps/gateway`, `packages/api-types`, `packages/ui`, `deploy/`) |
| Frontend | React 18 + Vite + TypeScript + Tailwind + shadcn/ui; TanStack Query; React Router; React Hook Form + Zod; Recharts |
| Mock da Fase 1 | Prism (`@stoplight/prism-cli`) servindo `api/openapi.yaml`, mais MSW para cenários dinâmicos do playground |
| Control plane (Fase 2) | Python 3.12 + FastAPI + SQLAlchemy 2 + Alembic + PostgreSQL 16 |
| Gateway (Fase 2) | LiteLLM Proxy como motor de roteamento, com hooks próprios do DOMINANCE (pre/post call) para políticas, orçamento e medição. Confirmar a licença dos recursos usados. |
| Identidade | Keycloak (OIDC), federado com Entra ID / Google |
| Cache e quotas | Redis 7 |
| Mascaramento de PII | Microsoft Presidio + reconhecedores brasileiros (CPF, CNPJ, telefone) |
| Observabilidade | OpenTelemetry → Prometheus + Grafana; Langfuse opcional |
| Infra | Docker Compose (dev e nó único) e Helm (Kubernetes/RKE2/K3s); Terraform para AWS |

## Convenções

- Tema claro e escuro, com os tokens de `brand/tokens.json` (cobalto = ações/nuvem, dourado = local/soberano;
  laranja, azul e verde só para identificar as camadas Workspace, Governance e Core).
- Cada app usa o símbolo do seu produto e a linha "Parte da DOMINANCE® Platform".
- Toda rota do console tem estado de carregando, vazio e erro.
- Formulários validam no cliente (Zod, mesmas regras do OpenAPI) e mostram o erro ao lado do campo.
- Testes: Vitest + Testing Library no frontend; pytest no backend; Playwright para os 5 cenários do playground.
- Commits pequenos (Conventional Commits). Uma tarefa do roadmap por branch.
- Antes de concluir qualquer tarefa: `pnpm lint && pnpm typecheck && pnpm test` sem erros.

## O que NÃO fazer

- Não acoplar o código ao MultiCortex ou a qualquer provedor fora dos adaptadores.
- Não inventar endpoints que não estão no OpenAPI.
- Não guardar texto de prompt em banco, log ou trace.
- Não usar `localStorage` para dados de negócio; só preferências de interface.
- Não chamar provedores reais na Fase 1.
