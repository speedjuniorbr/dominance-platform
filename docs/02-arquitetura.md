# DOMINANCE Platform — arquitetura de referência (v0.2)

## 1. Camadas

```
┌──────────────────────────────────────────────────────────────────────┐
│ CONSUMIDORES   apps e agentes · chat corporativo · IDEs e copilotos   │
└───────────────────────────────┬──────────────────────────────────────┘
                                │  API compatível com OpenAI + chave virtual
┌───────────────────────────────▼──────────────────────────────────────┐
│ PLANO DE GOVERNANÇA DOMINANCE                                         │
│  Gateway (data plane)                    Control plane                │
│  identidade → permissão → orçamento →    console web, API admin,      │
│  política de dados → cache → rota →      catálogo, políticas,         │
│  medição/auditoria                       orçamentos, auditoria        │
│  ─────────────────────────────────────────────────────────────────── │
│  Keycloak · PostgreSQL · Redis · Presidio · Vault · OTel/Prometheus   │
└───────────────┬───────────────────────────────────┬──────────────────┘
                │ adaptadores                        │ adaptadores
┌───────────────▼─────────────────┐   ┌──────────────▼──────────────────┐
│ BASES EM NUVEM                  │   │ BASES LOCAIS (SOBERANAS)         │
│ OpenAI · Anthropic · Vertex AI  │   │ MultiCortex OS (vLLM/Ollama)     │
│ AWS Bedrock · Azure OpenAI      │   │ NVIDIA NIM · Nutanix Enterprise AI│
└─────────────────────────────────┘   └──────────────────────────────────┘
```

O DOMINANCE Governance é a camada do meio. O DOMINANCE Core é o hardware: um rack air-gapped montado
com equipamentos de mercado (servidores de GPU NVIDIA, por exemplo Supermicro; nós hiperconvergentes,
por exemplo Nutanix; switches 400G InfiniBand/RoCE; NVMe; UPS de lítio), entregue com a marca DOMINANCE Core. O MultiCortex OS é a primeira base local homologada, não uma dependência:
o manual técnico do MultiCortex (v2.1) informa que não há API administrativa própria nem API nativa única;
a integração acontece pelo endpoint compatível com OpenAI de cada motor (vLLM, Ollama, llama.cpp).

## 2. Componentes

| Componente | Responsabilidade | Tecnologia |
|---|---|---|
| `apps/governance-web` | Landing e console do Governance | React + Vite + TS |
| `apps/workspace-web` | Interface do usuário (DOMINANCE Workspace) | React + Vite + TS |
| `apps/control-plane` | API `/v1/admin/*`, regras de negócio, agregações de uso | FastAPI + PostgreSQL |
| `apps/gateway` | `/v1/chat/completions`, `/v1/embeddings`, `/v1/models` e pipeline | LiteLLM Proxy + hooks DOMINANCE |
| `gateway/adapters/*` | Um adaptador por `ProviderType` | Configuração LiteLLM + código próprio quando necessário |
| Identidade | SSO, perfis, emissão de tokens do console | Keycloak |
| Dados | Configuração, catálogo, políticas, orçamentos, eventos | PostgreSQL (ClickHouse na V2 para volume) |
| Cache/quotas | Cache de respostas, contadores de orçamento em tempo real | Redis |
| PII | Detecção e mascaramento (CPF, CNPJ, e-mail, telefone) | Presidio + reconhecedores BR |
| Segredos | Chaves de provedores | Vault (local) / AWS Secrets Manager (SaaS) |
| Observabilidade | Métricas, traces sem conteúdo, painéis técnicos | OpenTelemetry, Prometheus, Grafana |

### Hooks do gateway

O LiteLLM é o motor de roteamento; as regras do DOMINANCE ficam em hooks próprios, para não depender de
recursos pagos e manter a lógica sob controle:

- `pre_call`: resolve identidade e equipe → permissão → orçamento (Redis) → política de dados (Presidio)
  → cache. Pode reescrever o alias (rota local) ou o conteúdo (mascaramento), ou abortar com 403/429.
- `post_call`: calcula tokens e custo pelo catálogo, grava evento de uso, atualiza contador do orçamento,
  grava no cache, emite métricas. Assíncrono, sem atrasar a resposta.
- Cabeçalhos de resposta `x-dominance-*` conforme o OpenAPI.

## 3. Modelo de dados (mínimo)

`organization`, `team`, `user`, `virtual_key` (hash), `provider`, `catalog_model`, `policy`,
`budget`, `usage_event` (particionada por mês), `audit_event`, `deployment_config`, `waitlist_entry`.
O `usage_event` guarda: request_id, timestamp, user_id, team_id, key_id, alias_requested, alias_routed,
provider_id, tokens_in, tokens_out, cost_usd, savings_usd, latency_ms, cache_hit, policies_applied[].
**Nunca** o conteúdo.

## 4. Modos de implantação

| Modo | Onde roda o control plane | Onde roda o gateway | Bases | Empacotamento |
|---|---|---|---|---|
| SaaS | AWS (sa-east-1), multi-tenant lógico | AWS | Nuvem + locais via túnel | Terraform + Helm (EKS) |
| Híbrido | AWS | Cliente, sobre MultiCortex OS | Locais + nuvem | Helm (K3s/RKE2) ou Compose |
| Soberano | Cliente | Cliente | Locais (nuvem opcional) | Helm/Compose + imagens assinadas, sem telemetria externa |

No modo híbrido, o gateway local envia ao control plane apenas metadados de uso (nunca prompts) e recebe
configuração. Se o link cair, o gateway continua aplicando a última configuração conhecida.

## 5. Segurança

- TLS em todo endpoint; mTLS entre gateway local e control plane no modo híbrido.
- Chaves virtuais com hash; rotação e expiração.
- Princípio do menor privilégio nos perfis; ações administrativas sempre auditadas.
- Logs e traces sem conteúdo de prompt; retenção configurável (30 dias a 5 anos).
- Serviços de inferência sem root e com portas restritas à rede do gateway (recomendação do manual MultiCortex).

## 6. Pontos a validar com a MultiCortex

1. Métricas expostas pelos motores (tokens/s, TTFT, uso de GPU) e em qual formato (Prometheus?).
2. Roadmap de API administrativa (carregar/descarregar modelos, inventário).
3. Telemetria e atualizações em ambiente desconectado.
4. Suporte a Kubernetes e passagem de GPU/NPU para contêineres.
5. Matriz de hardware homologada e imagem para nuvem (AMI), se existir.
6. Modelo comercial: OEM/white-label, redistribuição, licenças de drivers e modelos, SLA.
