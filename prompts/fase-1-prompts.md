# Fase 1 — sequência de prompts para o Claude Code

Como usar:
- Abra o Claude Code na raiz deste kit (onde está o `CLAUDE.md`).
- Rode um prompt por vez, na ordem. Só avance quando o critério de pronto do prompt for atendido.
- Ao fim de cada prompt, peça: "rode lint, typecheck e testes e me mostre o resultado".
- Faça commit ao fim de cada etapa.

---

## P1 — Estrutura do monorepo

```
Leia CLAUDE.md, docs/01-produto.md, docs/02-arquitetura.md e api/openapi.yaml.
Crie o monorepo pnpm com:
- apps/governance-web e apps/workspace-web (React 18 + Vite + TypeScript + Tailwind + shadcn/ui + React Router + TanStack Query + React Hook Form + Zod + Recharts)
- packages/api-types (tipos gerados de api/openapi.yaml com openapi-typescript; script "pnpm gen:api")
- api/ mantendo o openapi.yaml atual
- scripts na raiz: dev, build, lint, typecheck, test, gen:api, mock
Configure ESLint, Prettier, Vitest e Playwright. Não crie telas ainda.
Pronto quando: pnpm install, pnpm gen:api, pnpm lint, pnpm typecheck e pnpm test passam.
```

## P2 — Mock da API a partir do contrato

```
Configure o mock da Fase 1:
- "pnpm mock" sobe o Prism em http://localhost:4010 servindo api/openapi.yaml em modo dinâmico.
- Adicione exemplos (examples) ao OpenAPI para que o mock devolva dados coerentes com o protótipo:
  organização CRTC.ai, equipes Jurídico/Engenharia/Comercial/Atendimento/Diretoria, os 7 conectores,
  os 7 aliases do catálogo, as 7 políticas e 30 dias de uso. Use prototypes/governance/index.html (objeto DB) como fonte dos dados.
- Em packages/api-types, crie um cliente HTTP tipado (packages/api-types) cuja URL base vem de VITE_API_BASE_URL.
Pronto quando: GET /v1/admin/usage/summary, /v1/admin/providers e /v1/admin/policies respondem dados coerentes
e o cliente tipado compila.
```

## P3 — Design system e layout

```
Implemente a identidade visual do protótipo (prototypes/governance/index.html, bloco :root):
- Tokens de cor claro e escuro (cobalto = governança/nuvem, dourado = local/soberano, semânticos ok/warn/crit).
- Fontes Archivo (títulos), IBM Plex Sans (texto) e IBM Plex Mono (código e números).
- Componentes: Button, Pill/Badge (ok, warn, crit, local, cloud, neutral), Card/Panel, KPI, Table, ProgressBar,
  Switch, Modal, Toast, FormField com erro ao lado do campo.
- Layout do console: barra lateral escura com grupos Operação / Governança / Plataforma, faixa "Protótipo · dados simulados".
- Rotas: / (landing) e /console/* conforme a tabela da seção 5 de docs/01-produto.md.
Pronto quando: navegação entre todas as rotas funciona, com páginas vazias e tema claro/escuro corretos.
```

## P4 — Landing page

```
Implemente a landing (/) seguindo o protótipo: hero com o cartão "Uma requisição, sete verificações",
"O que o DOMINANCE resolve", "Qualquer base de IA" (chips local/nuvem), modos de implantação e lista de espera.
O formulário da lista de espera usa React Hook Form + Zod com as regras de docs/01-produto.md seção 7 e envia
POST /v1/public/waitlist. Mostre sucesso inline, sem alert.
Pronto quando: as validações aparecem ao lado de cada campo e o envio funciona contra o mock.
```

## P5 — Visão geral e Consumo

```
Implemente /console (Visão geral) e /console/consumo consumindo apenas a API:
- 5 KPIs, gráfico de barras empilhadas local × nuvem por dia (Recharts), custo por equipe, últimos eventos,
  painel "Precisa de atenção" (orçamentos acima do alerta, conectores degradados).
- Consumo: agrupar por equipe/usuário/modelo/provedor, período 7/14/30 dias, coluna % local com barra,
  linha de total, botão "Exportar CSV" (GET /v1/admin/usage?format=csv).
Inclua estados de carregando, vazio e erro.
Pronto quando: os números batem com o mock e os testes de componente passam.
```

## P6 — Conectores e catálogo

```
Implemente /console/conectores: cartões por conector (borda dourada = local, cobalto = nuvem, tracejada = planejado),
status, latência, endpoint, motor; catálogo de aliases com preço e equipes.
Modal "Adicionar conector" com os campos e validações do protótipo (https://, vault:// | aws-sm:// | env://),
tipos do enum ProviderType (inclui multicortex, nvidia_nim, nutanix_ai). Após criar, mostrar status "checking"
e consultar GET /providers/{id}/health.
Pronto quando: criar conector atualiza a lista e erros aparecem ao lado do campo.
```

## P7 — Políticas, Orçamentos, Acesso, Auditoria e Implantação

```
Implemente as telas restantes do console conforme o protótipo e o OpenAPI:
- Políticas: lista com Switch (PATCH), modal "Nova política".
- Orçamentos: tabela com barra (cor por estado), modal de edição (PUT), regras de validação.
- Acesso: pessoas (convidar), chaves virtuais (criar: modal mostra o segredo uma única vez com botão Copiar;
  revogar com confirmação), permissões por equipe.
- Auditoria: filtro por nível, paginação por cursor, exportar CSV.
- Implantação: modo SaaS/híbrido/soberano (cartões de rádio), base local padrão, retenção, SSO (validação https).
Pronto quando: todos os formulários validam no cliente e persistem no mock.
```

## P8 — Playground com os 5 cenários

```
Implemente /console/playground. Use MSW para simular o pipeline do gateway no navegador (Prism não tem estado),
reproduzindo exatamente a lógica da função gateway() de prototypes/governance/index.html:
identidade → permissão → orçamento → política de dados (mascarar PII / rota local) → cache → rota/fallback → medição.
- Painel "Caminho do pedido" com as 7 etapas animadas (ok, ajustado, barrado, pulado).
- Resultado: alias final, provedor, tokens, latência, custo, economia, "texto que saiu do ambiente".
- Snippet curl com o alias escolhido.
- Botões dos 5 cenários de docs/01-produto.md seção 6.
- A chamada usa o mesmo cliente e o mesmo contrato de POST /v1/chat/completions, com cabeçalhos x-dominance-*.
Escreva testes Playwright para os 5 cenários e para o acerto de cache.
Pronto quando: os 6 testes Playwright passam.
```

## P8b — DOMINANCE Workspace

```
Implemente apps/workspace-web seguindo prototypes/workspace/index.html e docs/01-produto.md seção 5b:
conversas com modo Automático, aviso de governança enquanto o usuário digita, selo em cada resposta
(local/nuvem, regras aplicadas, CPU/GPU/NPU, tokens, custo, "Ver caminho"), Agentes e conectores,
Meu uso e regras. Use o símbolo Portal e a linha "Parte da DOMINANCE® Platform" (brand/).
O chat chama POST /v1/chat/completions do mesmo contrato; no modo mock, reutilize os handlers MSW do P8.
Pronto quando: os três usuários de demonstração (Jurídico, Comercial, Atendimento) reproduzem os
comportamentos do protótipo e há testes Playwright para eles.
```

## P9 (opcional, recomendado) — Fatia real

```
Crie deploy/fatia-real/docker-compose.yml com:
- LiteLLM Proxy configurado com 1 provedor em nuvem (chave via variável de ambiente) e 1 modelo Ollama local,
  expostos com os aliases dominance-geral e dominance-rapido.
- Um serviço mínimo (FastAPI) que implementa POST /v1/chat/completions repassando ao LiteLLM, medindo tokens
  e custo e devolvendo os cabeçalhos x-dominance-*.
Nos dois apps, uma variável VITE_PLAYGROUND_MODE=real|mock define se o Playground usa MSW ou a fatia real.
Pronto quando: no modo real, uma pergunta retorna resposta verdadeira e os tokens/custo reais aparecem na tela.
```

## P10 — Empacotamento da demonstração

```
Prepare a demo:
- "pnpm demo" sobe mock (ou fatia real) e frontend com um comando.
- README com passo a passo, credenciais de demonstração e roteiro de 10 minutos
  (landing → visão geral → playground cenários 1 a 5 → políticas desliga PII e repete cenário 2 → orçamentos → auditoria).
- Build estático do frontend para hospedar em S3 + CloudFront.
Pronto quando: outra pessoa consegue rodar a demo do zero seguindo só o README.
```
