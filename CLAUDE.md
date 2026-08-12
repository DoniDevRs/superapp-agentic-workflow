# SuperApp Agentic Lab — Contexto do Projeto

## O que é este projeto
Laboratório de estudo que redesenha a jornada de Transferência via Pix de um
super-app bancário fictício. O objetivo duplo é (1) entregar essa jornada e
(2) validar um fluxo de desenvolvimento orientado por especificação (SDD —
Spec-Driven Development) executado por agentes de IA especializados, descrito
na seção [Fluxo SDD](#fluxo-sdd-spec-driven-development) abaixo.

## Arquitetura
- Clean Architecture + MVVM-C
- Módulos:
  - **App** — shell da aplicação, composição de dependências, ponto de entrada
  - **Core** — utilitários, networking e modelos compartilhados entre módulos
  - **DesignSystem** — componentes visuais reutilizáveis (fonte única de verdade de UI)
  - **Pix** — feature module da jornada de Transferência via Pix
- SwiftUI para novas telas, UIKit para navegação (Coordinator pattern)

## Fluxo SDD (Spec-Driven Development)
Toda feature neste projeto passa pelas cinco etapas abaixo, nesta ordem. Cada
etapa produz um artefato revisável em `/specs`, e a etapa seguinte só começa
depois que o artefato anterior foi aprovado — nenhum agente deve pular etapas
ou implementar a partir de uma spec ainda em rascunho.

1. **Spec** — descreve o *quê* e o *porquê* da feature: problema do usuário,
   comportamento esperado, critérios de aceite e restrições de negócio. Sem
   detalhes de implementação. Vive em `/specs/<feature>/spec.md`.
2. **Plan** — traduz a spec em decisão técnica: quais módulos são afetados,
   contratos de API/DTOs, componentes de Design System a reutilizar (ou criar),
   pontos de navegação no Coordinator, e riscos técnicos. Vive em
   `/specs/<feature>/plan.md`.
3. **Tasks** — quebra o plan em unidades de trabalho pequenas, sequenciáveis e
   independentemente verificáveis (ex.: "criar ViewModel X", "adicionar rota Y
   no Coordinator"). Vive em `/specs/<feature>/tasks.md`.
4. **Implement** — agente(s) implementam as tasks uma a uma, seguindo as
   regras da seção abaixo (Coordinator, Design System, acessibilidade).
5. **Test** — testes de ViewModel e validação de acessibilidade (VoiceOver +
   Dynamic Type) para cada task implementada, antes de considerá-la concluída.

> Regra de ouro: se um agente encontrar ambiguidade em qualquer etapa, ele
> deve voltar para o artefato da etapa anterior e esclarecê-lo — nunca
> resolver a ambiguidade "no código" durante a implementação.

## Regras que todo agente deve seguir
- Seguir o [Fluxo SDD](#fluxo-sdd-spec-driven-development): nunca implementar uma feature sem spec, plan e tasks aprovados em `/specs`
- Nunca criar navegação fora do Coordinator
- Toda tela nova precisa ter: acessibilidade (VoiceOver + Dynamic Type), testes de ViewModel
- Não duplicar componentes visuais — sempre checar o Design System primeiro

## Como este repositório se relaciona com os outros
- **superapp-ios** — consome os specs e regras definidos aqui como fonte de verdade
- **superapp-design-system** — fonte dos componentes visuais referenciados no Design System
- **superapp-api** — backend Java consumido pelo app
