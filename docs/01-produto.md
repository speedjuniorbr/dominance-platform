# DOMINANCE Platform — especificação de produto (v0.2)

AxiosTech Solutions · CRTC.ai · novembro de 2026

## 1. Problema e proposta

Organizações já usam vários modelos de IA ao mesmo tempo (ChatGPT, Claude, Gemini, copilotos de código,
modelos internos), cada um com sua conta, sua fatura e nenhuma visão consolidada. Isso traz três dores:

- **Controle:** não se sabe quem usa qual modelo, nem com quais dados.
- **Custo:** a fatura de tokens cresce sem dono e sem limite.
- **Soberania:** dados sensíveis saem para nuvens estrangeiras sem que ninguém decida isso.

O DOMINANCE coloca um único ponto de controle entre as pessoas e os modelos. Ele decide quem acessa o quê,
mede o consumo por equipe e reduz o gasto, mandando o que pode para modelos locais e reaproveitando
respostas. Funciona na nuvem ou instalado no data center do cliente, sobre bases soberanas como o
MultiCortex OS.

## 2. Perfis

| Perfil | Quem é | O que faz no DOMINANCE |
|---|---|---|
| Administrador | TI / CISO / gestor da plataforma | Conectores, catálogo, políticas, orçamentos, acesso, implantação |
| Gestor | Líder de equipe | Vê consumo e orçamento da sua equipe, convida pessoas, pede liberação de modelos |
| Usuário | Colaborador | Usa o chat corporativo e os aplicativos; não acessa o console |
| Aplicativo | Sistema integrado | Chama o gateway com chave virtual |

## 3. Conceitos (glossário do domínio)

- **Organização:** o cliente (ex.: CRTC.ai). No MVP há uma organização por instalação.
- **Equipe:** agrupa pessoas e define os modelos permitidos.
- **Conector (provider):** uma base de inferência. Local (MultiCortex, NVIDIA NIM, Nutanix Enterprise AI,
  Ollama, vLLM) ou nuvem (OpenAI, Anthropic, Vertex, Bedrock, Azure).
- **Alias de modelo:** nome estável do catálogo (`dominance-rapido`) que aponta para um modelo real num
  conector. Aplicativos só conhecem aliases; trocar o modelo por trás não exige mudar o aplicativo.
- **Chave virtual:** credencial do DOMINANCE para aplicativos (`dmn_live_…`). Substitui a chave do provedor.
- **Política:** regra aplicada a toda requisição (mascarar PII, forçar rota local, cache, fallback…).
- **Orçamento:** limite mensal em US$ por equipe, com percentual de alerta e ação ao atingir 100%.
- **Evento de auditoria:** registro de uso ou de governança, sem o texto do prompt.

## 4. Pipeline de uma requisição

1. **Identidade:** valida a chave virtual ou o token SSO e resolve usuário e equipe.
2. **Permissão do modelo:** o alias existe no catálogo e é permitido para a equipe/chave? Senão, 403.
3. **Orçamento:** a equipe passou de 100% com ação "bloquear"? Então 429. Passou do alerta? Registra aviso.
4. **Política de dados:** detecta PII e termos sensíveis. Para equipes no escopo de "conteúdo sensível só
   local", troca a rota para o alias local. Para os demais, mascara PII antes de sair para a nuvem.
5. **Cache:** mesma combinação alias + conteúdo normalizado em até 24h devolve a resposta guardada, custo zero.
6. **Roteamento e inferência:** chama o conector. Se ele falhar ou passar do tempo limite, usa o fallback.
7. **Medição e auditoria:** grava tokens, custo, economia, latência, políticas aplicadas e request_id.

## 5. Telas do DOMINANCE Governance (referência: `prototypes/governance/index.html`)

Menu organizado em **Observar** (Visão geral, Consumo, Auditoria, Playground), **Controlar** (Conectores e
modelos, Políticas, Orçamentos, Acesso), **Proteger** (Guardrails), **Infraestrutura** (DOMINANCE Core) e
**Plataforma** (Implantação).

| Rota | Tela | Conteúdo principal |
|---|---|---|
| `/` | Landing | Proposta, como funciona, bases suportadas, modos de implantação, lista de espera (formulário validado) |
| `/console` | Visão geral | 5 indicadores (tokens, custo, % local, economia, barradas), tokens/dia local × nuvem, custo por equipe, eventos recentes, alertas |
| `/console/playground` | Playground | Chat com seleção de usuário e alias, pipeline animado das 7 etapas, resultado (tokens, custo, latência, texto que saiu), snippet de integração, 5 cenários de exemplo |
| `/console/consumo` | Consumo | Agrupar por equipe/usuário/modelo/provedor, período 7/14/30 dias, % local, economia, exportar CSV |
| `/console/conectores` | Conectores e modelos | Cartões por conector (local/nuvem/planejado, status, latência), catálogo de aliases com preço, formulário "Adicionar conector" |
| `/console/politicas` | Políticas | Lista com liga/desliga, escopo e tipo; formulário "Nova política" |
| `/console/orcamentos` | Orçamentos | Barra de uso por equipe, limite, alerta e ação; edição em modal |
| `/console/acesso` | Acesso | Pessoas (convidar), chaves virtuais (criar com segredo exibido uma vez, revogar), permissões por equipe |
| `/console/auditoria` | Auditoria | Tabela filtrável por nível; exportar CSV |
| `/console/implantacao` | Implantação | Modo SaaS/híbrido/soberano, base local padrão, retenção, configuração do SSO |

| `/console/guardrails` | Guardrails | Observar/Controlar/Proteger, regras (prompt injection na entrada, vazamento na saída, IA não autorizada, aprovação humana de agentes), detecções recentes, simulação de ataque |
| `/console/core` | DOMINANCE Core | Telemetria das GPUs (uso, temperatura, watts), consumo do rack, EADRA, fila de tarefas com prioridade e preempção com checkpoint |

## 5b. Telas do DOMINANCE Workspace (referência: `prototypes/workspace/index.html`)

| Rota | Tela | Conteúdo principal |
|---|---|---|
| `/` | Conversas | Chat com modo Automático ou escolha de modelo permitido, aviso de governança antes de enviar, selo em cada resposta (local/nuvem, regras aplicadas, CPU/GPU/NPU, tokens, custo, "Ver caminho") |
| `/agentes` | Agentes e conectores | Agentes configurados no Governance e conectores com indexação local |
| `/meu-uso` | Meu uso e regras | Consumo do usuário, orçamento da equipe e políticas que se aplicam a ele |

## 6. Cenários de demonstração (devem funcionar na Fase 1 e na Fase 2)

| # | Cenário | Entrada | Resultado esperado |
|---|---|---|---|
| 1 | Pergunta simples | Atendimento + `dominance-rapido` | Roda local, custo quase zero; repetido, vem do cache |
| 2 | Dado pessoal | Comercial + `dominance-geral` + CPF e e-mail | CPF e e-mail mascarados antes de sair para a nuvem |
| 3 | Conteúdo sensível | Jurídico + `dominance-raciocinio` + "contrato confidencial" | Rota forçada para `dominance-local-pro`; nada sai do ambiente |
| 4 | Sem permissão | Atendimento + `dominance-raciocinio` | 403 com mensagem clara |
| 5 | Orçamento esgotado | Diretoria (acima de 100%) | 429 com orientação para o gestor |

## 7. Regras de validação (iguais no cliente e no servidor)

- E-mail válido; na lista de espera, recusar domínios pessoais (gmail, hotmail, outlook, yahoo).
- Endpoint de conector começa com `https://`.
- Referência de segredo começa com `vault://`, `aws-sm://` ou `env://`.
- Nome de chave virtual: `^[a-z0-9-]{3,40}$`.
- Orçamento: 0 < limite ≤ 100.000; alerta entre 10% e 99%.
- Alias de catálogo: `^dominance-[a-z0-9-]+$`.

## 8. Fora do escopo do MVP

Cache semântico, roteamento automático por complexidade, RAG governado, DLP avançado, multi-organização
com faturamento, white-label, pacotes de compliance (ISO 42001, EU AI Act), atualização offline assinada,
conectores NVIDIA NIM e Nutanix Enterprise AI em produção (o tipo já existe no contrato).
Telemetria real do Core e preempção de tarefas dependem das APIs dos fabricantes e da MultiCortex (validar).
