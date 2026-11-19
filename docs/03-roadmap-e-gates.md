# DOMINANCE Platform — roadmap de construção, entregas e gates

> Este roadmap é da construção do produto. O modelo de implantação comercial nos clientes ainda está em
> definição e não faz parte deste documento.

## Fase 0 — Bases da parceria (1 a 2 semanas, em paralelo à Fase 1)

| Entrega | Responsável |
|---|---|
| Memorando ou acordo de sócios AxiosTech × CRTC.ai (participação, titularidade do código e da marca, papéis) | Sócios + jurídico |
| NDA com a MultiCortex | AxiosTech |
| Busca e pedido de registro da marca DOMINANCE no INPI | AxiosTech |
| Hipótese de preço (por usuário, por organização ou % sobre consumo) | Sócios |

**Gate 0:** documentos assinados antes da primeira reunião com investidores.

## Fase 1 — Protótipo navegável reaproveitável (3 a 4 semanas)

Objetivo: ter algo para mostrar a sócios e investidores, sem descartar nada na Fase 2.

| Semana | Entrega |
|---|---|
| 1 | Monorepo, contrato `api/openapi.yaml` revisado, tipos gerados, mock Prism no ar, design tokens e layout do console |
| 2 | Landing com lista de espera; Visão geral; Consumo; Conectores e modelos |
| 3 | Políticas; Orçamentos; Acesso (chaves com segredo exibido uma vez); Auditoria; Implantação |
| 4 | Playground com os 5 cenários (MSW), fatia real opcional, testes Playwright, roteiro de demonstração |

**Fatia real opcional (recomendada):** um LiteLLM local com 1 provedor em nuvem e 1 modelo Ollama, e o
Playground chamando de verdade, com tokens e custo reais na tela. Deixe claro na demo o que é real e o que é simulado.

**Gate 1 (objetivo):**
- Conceito e escopo aprovados pelo CRTC.ai.
- Hipótese de preço definida.
- 3 a 5 conversas com investidores ou potenciais clientes com retorno registrado.
- Inscrições na lista de espera, como sinal de interesse.

## Fase 2 — MVP funcional (cerca de 12 semanas)

Objetivo: demonstrar o funcionamento real e conquistar o cliente lançador.

| Semanas | Entrega |
|---|---|
| 1–2 | Infra AWS (Terraform), Keycloak, PostgreSQL, Redis; servidor de laboratório com MultiCortex OS + vLLM |
| 3–6 | Gateway com hooks (identidade, permissão, orçamento, PII, cache, fallback, medição); conectores OpenAI, Anthropic, Bedrock ou Vertex, e MultiCortex |
| 7–9 | Control plane implementando todo o OpenAPI; frontend da Fase 1 apontado para o backend real; chat corporativo |
| 10 | Pacote instalável (Helm/Compose) sobre MultiCortex; instância SaaS; testes de carga, segurança e LGPD |
| 11–12 | Piloto com 20 a 50 usuários reais; relatório de resultados |

**Busca do cliente lançador começa na semana 1 da Fase 2, não depois do gate.**

**Gate 2 (objetivo):**
- 100% do uso de IA do grupo piloto passando pelo DOMINANCE.
- Consumo visível por usuário e por equipe, com custo.
- Economia mensurável com o desvio para o modelo local, comparada a uma linha de base medida antes do piloto.
- Instalação local concluída em menos de 1 dia seguindo o roteiro.
- O mesmo pacote funcionando em SaaS e sobre MultiCortex.
- Revisão de segurança sem achados críticos em aberto.

## Equipe sugerida para a Fase 2

Tech lead/arquiteto, 2 backend, 1 frontend, 1 DevOps/MLOps; PO em tempo parcial (CRTC.ai ou AxiosTech);
apoio técnico da MultiCortex; revisão de segurança e LGPD antes do piloto.
