# SuperApp Agentic Lab

Laboratório de estudo com objetivo duplo: (1) redesenhar a jornada de
**Transferência via Pix** de um super-app bancário fictício e (2) validar, na
prática, um fluxo de desenvolvimento orientado por especificação (SDD)
executado por agentes de IA especializados — do requisito de negócio até o
code review, passando por implementação, testes e auditoria de
acessibilidade.

Este repositório (`superapp-agentic-workflow`) concentra o contexto do
projeto, o fluxo SDD, as skills e os subagents. A implementação real vive em
três repositórios satélite:
[`superapp-ios`](https://github.com/DoniDevRs/superapp-ios),
[`superapp-design-system`](https://github.com/DoniDevRs/superapp-design-system)
e [`superapp-api`](https://github.com/DoniDevRs/superapp-api).

## O problema

O fluxo fictício original de Pix ("MeuBanco") tinha 6 telas e 11 campos para
completar uma transferência: menu → tipo de chave → destinatário (com a
chave Pix digitada duas vezes, para confirmação) → valor → confirmação (com
senha na mesma tela densa de revisão) → comprovante. O tipo de chave era
perguntado *antes* de saber quem era o destinatário, uma decisão redundante
já que o tipo pode ser inferido a partir do valor digitado.

Havia também problemas de acessibilidade estruturais: alvos de toque de
26–30px (abaixo dos 44×44pt recomendados), texto de 11px em cinza claro
sobre cinza (contraste abaixo de 3:1, quando o WCAG 2.1 AA exige 4.5:1 para
texto normal), e uma tela final que misturava a revisão dos dados com a
digitação da senha.

Esse cenário (documentado em [`design/images/pix-antes.png`](design/images/pix-antes.png))
foi o ponto de partida para a spec em
[`specs/pix-transfer-redesign/spec.md`](specs/pix-transfer-redesign/spec.md).

## Arquitetura

O fluxo de trabalho segue um pipeline linear, onde cada etapa produz um
artefato revisável antes da próxima começar:

```
Business Requirement
        │
        ▼
      SPEC        specs/pix-transfer-redesign/spec.md
        │          (o quê e por quê — sem detalhes de implementação)
        ▼
      PLAN        specs/pix-transfer-redesign/plan.md
        │          (módulos, contratos, componentes de Design System, riscos)
        ▼
      TASKS       specs/pix-transfer-redesign/tasks.md
        │          (unidades de trabalho pequenas e verificáveis)
        ▼
   Implementation  Domain → Data → Presentation → Coordinator
        │          (skill ios-feature)
        ▼
      Tests        testes unitários de ViewModel/UseCase
        │          (skill test-generation, subagent qa-engineer)
        ▼
Accessibility Audit VoiceOver, Dynamic Type, contraste, hit area
        │           (skill accessibility-audit + performAccessibilityAudit())
        ▼
   Code Review      retain cycles, threading, Clean Architecture, duplicação
                    (subagent ios-reviewer)
```

A regra de ouro do fluxo: se um agente encontra ambiguidade em qualquer
etapa, ele volta para o artefato da etapa anterior e a esclarece — nunca
resolve "no código" durante a implementação. Isso aconteceu de fato durante
o projeto: o protótipo aprovado consolidou as etapas de "Valor" e "Revisão"
da spec original em uma única tela, e `plan.md`/`tasks.md` foram atualizados
para refletir essa decisão antes de a implementação prosseguir, em vez de a
divergência ser resolvida silenciosamente no código.

**Clean Architecture + MVVM-C.** O módulo `Pix` é dividido em três camadas —
Domain (entidades e use cases, sem dependência de UIKit/SwiftUI/rede), Data
(implementações de repositório sobre o networking do Core, mapeando erros de
transporte para erros de domínio) e Presentation (SwiftUI + um único
`PixViewModel` compartilhado pelas três telas do fluxo, evitando
retransmissão manual de estado entre ViewModels). A navegação nunca é feita
diretamente pela View: um `PixCoordinator` (UIKit, `UINavigationController`)
recebe intenções via closures (`onRecipientSelected`, `onTransferConfirmed`,
etc.) e decide para onde ir — regra reforçada tanto no `CLAUDE.md` quanto na
checklist do subagent `ios-reviewer`.

## O que foi construído

- **UIKit + SwiftUI** — navegação via Coordinator pattern (UIKit) envolvendo
  telas construídas em SwiftUI (`UIHostingController`)
- **Multi-repo + SPM** — 4 repositórios (`superapp-agentic-workflow`,
  `superapp-ios`, `superapp-design-system`, `superapp-api`), com o app iOS
  organizado em pacotes Swift Package Manager (`App`, `Packages/Core`,
  `Packages/Pix`) consumindo o pacote `SuperAppDesignSystem` como
  dependência externa
- **Design System próprio** — componentes de campo de texto, lista de
  seleção, botões e banners reaproveitados do `superapp-design-system` em
  vez de recriados localmente no módulo Pix
- **Backend Java (Spring Boot)** — `superapp-api` expõe `PixController`,
  `AccountController` e `ContactController`, com DTOs, serviços e exceções
  de domínio (`InsufficientBalanceException`, `RecipientNotFoundException`)
  sobre dados mockados
- **Testes unitários** — cobertura do `PixViewModel` e das camadas de
  Domain do módulo Pix, mais os testes existentes do módulo Core
- **WCAG 2.1 AA** — VoiceOver, Dynamic Type, contraste e área de toque
  mínima como critérios de aceite, não como item opcional de polimento

## Workflow agêntico

**`CLAUDE.md` como contexto persistente.** Define a arquitetura do projeto,
o fluxo SDD obrigatório (spec → plan → tasks → implement → test) e as regras
que todo agente segue independente da task — nunca criar navegação fora do
Coordinator, nunca duplicar componente do Design System, sempre exigir
testes de ViewModel e acessibilidade em telas novas. É lido antes de
qualquer implementação, não apenas consultado sob demanda.

**3 skills, cada uma com um papel específico:**
- [`ios-feature`](skills/ios-feature/SKILL.md) — orquestra a sequência de
  implementação de uma feature nova: ler a spec, checar o Design System
  antes de criar componente, seguir Clean Architecture + MVVM-C, gerar
  testes junto com a implementação e fechar com checagem de acessibilidade
- [`test-generation`](skills/test-generation/SKILL.md) — padroniza os
  testes de ViewModel (mock do repositório, caminho feliz + 2 cenários de
  erro no mínimo, convenção de nome `test_<condição>_<resultadoEsperado>`)
- [`accessibility-audit`](skills/accessibility-audit/SKILL.md) — checklist
  de WCAG 2.1 AA (label acessível, Dynamic Type, contraste 4.5:1, área de
  toque 44×44pt, ordem de leitura do VoiceOver) rodada antes de considerar
  qualquer tela pronta

**2 subagents com escopo restrito, ambos somente leitura (exceto execução de
testes):**
- [`ios-reviewer`](.claude/agents/ios-reviewer.md) — revisa retain cycles,
  violações de threading, violações de Clean Architecture e duplicação de
  código no módulo Pix. Nunca edita arquivos nem sugere que outra ferramenta
  o faça em seu lugar — apenas relata achados por severidade
- [`qa-engineer`](.claude/agents/qa-engineer.md) — roda a suíte de testes
  existente, analisa cobertura por leitura de código e aponta edge cases não
  cobertos (valores negativos, chave Pix malformada, timeout de rede, saldo
  exatamente igual ao valor transferido). Tem acesso a `Bash` só para
  executar comandos de teste/inspeção, nunca para escrever arquivos de teste
  ou alterar dependências

**Integração com GitHub via MCP.** O fluxo gerou issues reais no repositório
`superapp-ios`, não apenas tasks internas em markdown:
- [Issue #1](https://github.com/DoniDevRs/superapp-ios/issues/1) — *"Levantamento
  de componentes existentes no Design System"*, a task planejada da Fase 0
  de `tasks.md`, aberta antes de qualquer código de Domain ser escrito, para
  evitar duplicar componente visual
- [Issue #2](https://github.com/DoniDevRs/superapp-ios/issues/2) — um bug
  real encontrado durante o desenvolvimento: toque nos destinatários
  recentes não navegava para `ReviewPaymentView`. A issue documenta a
  investigação (wiring `Button` → `PixViewModel` → `PixCoordinator`
  revisado e correto por leitura de código, um teste `XCUITest` que passa
  mas não reproduz o problema com toque sintético) e levanta hipótese de
  race condition entre o carregamento da lista e a instalação dos gesture
  recognizers — o tipo de achado que só aparece testando o app de verdade,
  não só lendo o código

## Antes / Depois

| | Antes | Depois |
|---|---|---|
| Telas | 6 | 3 |
| Campos/entradas | 11 | 2 |
| Alvo de toque | 26–30px | ≥48px |
| Contraste (texto principal) | <3:1 | 12:1 |
| Chave Pix | digitada duas vezes | inferida a partir de uma busca única |
| Confirmação | senha na mesma tela densa da revisão | estado dedicado, sem senha misturada ao resumo |

Ver [`design/images/pix-antes.png`](design/images/pix-antes.png) e
[`design/images/pix-depois.png`](design/images/pix-depois.png) para o
comparativo visual completo, e o protótipo navegável em
[`design/prototype/index.html`](design/prototype/index.html).

## Auditoria de Acessibilidade

Antes de fechar a feature, as 3 telas (`SelectRecipientView`,
`ReviewPaymentView`, `ConfirmationView`) passaram por dois passes: a
checklist estática da skill `accessibility-audit` e um teste de UI novo
(`AccessibilityAuditUITests.swift`) que roda
`XCUIApplication().performAccessibilityAudit()` navegando o fluxo completo
Select → Review → Confirmation. Juntos, os dois passes encontraram e
corrigiram:

- **Hit area insuficiente** — os botões de texto "Trocar" e "Repetir para
  {nome}" tinham `frame(minWidth:minHeight:)` mas nenhum `contentShape`, ou
  seja, o frame maior não expandia a região realmente tocável. Confirmado
  pelo próprio audit ("Hit area is too small").
- **Elementos de acessibilidade não agrupados** — o resumo do destinatário
  em `ReviewPaymentView` combinava um `Text` de iniciais do avatar
  (`accessibilityHidden`) com o nome/banco visíveis sem agrupá-los, o que o
  audit sinalizou como "potentially inaccessible text". Corrigido agrupando
  em um único `accessibilityElement(children: .combine)`, mantendo o botão
  "Trocar" como elemento separado e focável.
- **Contraste insuficiente em modo escuro** — `PixTheme.error` era uma cor
  fixa (~2.66:1 no modo escuro, abaixo dos 4.5:1 exigidos); tornada
  adaptativa claro/escuro.
- **Ícones decorativos sem `accessibilityHidden`** — busca, chevron e
  triângulo de erro geravam paradas de VoiceOver sem label útil.
- **Ordem de leitura do VoiceOver** — o banner de erro em
  `SelectRecipientView` é um `.overlay` e por padrão é lido por último pelo
  VoiceOver apesar de estar visualmente por cima; corrigido com
  `accessibilitySortPriority`.
- **Dynamic Type ignorado** — os ícones de status em `ConfirmationView`
  usavam `.font(.system(size: 56))` fixo; substituído por `@ScaledMetric`.

A causa raiz de um dos achados relacionados a Dynamic Type não estava no
módulo Pix, mas no **Design System**: `DSFont` era construído via
`UIFontMetrics.scaledFont(for:)` embrulhado em `Font(uiFont:)`, o que
renderizava no tamanho correto mas não carregava os metadados de associação
de text style que `performAccessibilityAudit()` inspeciona — resultado, todo
`Text` usando `DSFont` era sinalizado com "Dynamic Type font sizes are
unsupported". A correção exigiu um commit em `superapp-design-system`,
reconstruindo `DSFont` sobre `@ScaledMetric` (API pública inalterada), e não
apenas em `superapp-ios`.

**Resultado final:** reexecutando `performAccessibilityAudit()` nas 3 telas
do fluxo após as correções — **0 achados**, contra falha em todo elemento de
texto que usava `DSFont` antes do fix no Design System.

## Resultados

- **23 testes unitários** (21 no módulo Pix + 2 no módulo Core) + **3 testes
  de UI** (2 testes de regressão existentes, relabelados após a mudança de
  accessibilityLabel de "Trocar" para "Trocar destinatário", e o novo teste
  de auditoria de acessibilidade) — todos passando.
- O processo de ponta a ponta foi seguido sem pular etapa: spec → plan →
  tasks → implementação em camadas (Domain → Data → Presentation →
  Coordinator) → testes → auditoria de acessibilidade → correção → reteste
  limpo. A única divergência encontrada no caminho (consolidação de duas
  telas da spec original em uma só) foi resolvida atualizando `plan.md` e
  `tasks.md` antes de prosseguir, e não silenciosamente no código —
  exatamente o comportamento que a regra de ouro do SDD exige.
