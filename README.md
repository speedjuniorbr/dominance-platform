# DOMINANCE® Platform

**Soberania em todas as camadas.**

Plataforma de IA soberana da AxiosTech Solutions com o CRTC.ai. Três camadas, uma marca:

| Camada | Tipo | O que faz |
|---|---|---|
| **DOMINANCE Workspace** | Software | Interface do usuário final: chat privado, agentes e conectores, com a governança visível em cada resposta |
| **DOMINANCE Governance** | Software | Plano de controle: gateway único compatível com OpenAI, acesso, políticas, roteamento semântico, orçamentos, guardrails e auditoria |
| **DOMINANCE Core** | Hardware | Rack air-gapped montado com equipamentos de mercado (GPUs NVIDIA, servidores Supermicro, HCI Nutanix etc.), entregue com a marca DOMINANCE Core |

A base de inferência local de referência é o **MultiCortex OS**. Outras bases (NVIDIA NIM, Nutanix
Enterprise AI, Ollama, vLLM) e os provedores em nuvem entram como adaptadores do Governance: sem lock-in.

## Estrutura do repositório

```
.
├── CLAUDE.md                 Instruções permanentes para o Claude Code
├── api/openapi.yaml          Contrato da API (gateway + administração): fonte da verdade
├── docs/
│   ├── 01-produto.md         Perfis, glossário, pipeline, telas, cenários de demonstração
│   ├── 02-arquitetura.md     Componentes, adaptadores, modos de implantação, segurança
│   ├── 03-roadmap-e-gates.md Fases de construção e critérios de aceite
│   └── 04-identidade-visual.md Marcas, cores, tipografia e regras de uso
├── prompts/
│   ├── fase-1-prompts.md     Sequência para gerar os apps reaproveitáveis com o Claude Code
│   └── fase-2-prompts.md     Sequência para o MVP real (AWS, control plane, gateway, pacote local)
├── prototypes/
│   ├── governance/index.html Protótipo navegável do DOMINANCE Governance
│   └── workspace/index.html  Protótipo navegável do DOMINANCE Workspace
├── brand/
│   ├── svg/                  Símbolos vetoriais (plataforma, Workspace, Governance, Core)
│   ├── png/                  Símbolos e assinaturas em PNG
│   └── tokens.json           Cores, fontes e raios da marca
├── assets/                   Ilustrações (rack Core 400/800, capa com as três camadas)
└── presentation/             Apresentação de conceito (.pptx, roteiro nas notas)
```

## Ver os protótipos

Abra `prototypes/governance/index.html` ou `prototypes/workspace/index.html` no navegador. Não há
dependências: os dados e as respostas são simulados no próprio arquivo. Os dois se ligam pelos links
"DOMINANCE Workspace" e "Abrir o console de governança".

Cenários para demonstrar no Governance (menu Observar → Playground): pergunta simples, dado pessoal,
conteúdo sensível, sem permissão e orçamento esgotado. No Workspace, troque o usuário no canto inferior
esquerdo (Jurídico, Comercial, Atendimento) para ver o comportamento mudar.

## Começar a construir

1. Leia `CLAUDE.md` e `docs/`.
2. Abra o Claude Code na raiz do repositório.
3. Execute os prompts de `prompts/fase-1-prompts.md`, um por vez, com um commit ao fim de cada um.

## Situação

- Identidade visual: aprovada (família de marcas e marca-mãe D completo).
- Protótipos: navegáveis, com dados simulados.
- Modelo de implantação comercial: em definição.
- Pendências com a MultiCortex: métricas dos motores, API administrativa, operação desconectada,
  Kubernetes, matriz de hardware e modelo comercial (ver `docs/02-arquitetura.md`, seção 6).

---
© 2026 AxiosTech Solutions e CRTC.ai. Todos os direitos reservados. Repositório privado.
