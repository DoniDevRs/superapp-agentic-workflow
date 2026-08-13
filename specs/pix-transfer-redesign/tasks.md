# Tasks: Pix Transfer Redesign

> **Nota de atualização (2026-08-13):** a Fase 3 originalmente descrevia 4
> pares View/ViewModel (uma tela e um ViewModel por etapa). O protótipo
> aprovado (`design/images/pix-depois.png`) consolidou "Valor" e "Revisão"
> em uma única tela, e a implementação usou um único `PixViewModel`
> compartilhado pelas 3 telas em vez de um ViewModel por tela. A Fase 3, a
> Fase 4 (Coordinator, agora `PixCoordinator`) e a Fase 5 (Acessibilidade)
> foram renumeradas e atualizadas abaixo para refletir isso. Ver também a
> nota equivalente em `plan.md`.

Ordem sequencial recomendada. Cada task deve ser concluída (e, quando aplicável,
testada) antes de iniciar a próxima.

## Fase 0 — Preparação
1. Levantar no Design System quais componentes já atendem ao fluxo (campo de texto, lista de seleção, botão primário/secundário, banner de erro/sucesso) e listar o que falta criar lá.
2. Alinhar com o backend (`superapp-api`) os contratos de: busca de destinatários salvos, validação de chave Pix, consulta de saldo e confirmação de transferência.

## Fase 1 — Domain (módulo Pix)
3. Criar as entidades de domínio: `PixRecipient`, `PixKey`, `TransferAmount`, `AccountBalance`, `PixTransferRequest`, `PixTransferResult`.
4. Criar os protocolos `PixRecipientRepository`, `AccountBalanceRepository`, `PixTransferRepository`.
5. Implementar `FetchSavedRecipientsUseCase`.
6. Implementar `ValidatePixKeyUseCase`.
7. Implementar `FetchAccountBalanceUseCase`.
8. Implementar `ValidateTransferAmountUseCase` (valor > 0 e ≤ saldo disponível).
9. Implementar `ConfirmPixTransferUseCase`.

## Fase 2 — Data (módulo Pix)
10. Criar DTOs e mappers DTO → entidade de domínio para destinatários, saldo e transferência.
11. Implementar `PixRecipientRepositoryImpl` sobre o cliente de rede do Core.
12. Implementar `AccountBalanceRepositoryImpl`.
13. Implementar `PixTransferRepositoryImpl`, incluindo mapeamento de erros de rede para erros de domínio.
14. Registrar repositórios e use cases no container de DI do módulo App.

## Fase 3 — Presentation: ViewModel único + 3 telas
15. `PixViewModel` (único, compartilhado pelas 3 telas) + testes unitários (lista de salvos, entrada manual de chave/validação, exibição de saldo, validação de valor, agregação dos dados para revisão, estado de sucesso e de erro).
16. `SelectRecipientView` (SwiftUI) usando componentes do Design System.
17. `ReviewPaymentView` (SwiftUI) — consolida valor + revisão numa única tela, com resumo revisável antes da confirmação.
18. `ConfirmationView` (SwiftUI) com mensagens claras de sucesso/erro.

## Fase 4 — Coordinator
19. Criar `PixCoordinator` (UIKit) orquestrando as 3 telas via `UIHostingController`.
20. Definir `start()` e o callback/delegate de finalização do fluxo; integrar ao ponto de entrada existente do super-app.
21. Conectar os callbacks do `PixViewModel` ao Coordinator (avançar, retroceder, cancelar o fluxo).

## Fase 5 — Acessibilidade
22. Revisar VoiceOver (labels, hints, ordem de leitura) nas 3 telas.
23. Revisar Dynamic Type nas 3 telas, incluindo tamanhos de fonte de acessibilidade.
24. Validar que erros/sucessos são comunicados além da cor (WCAG 2.1 AA) em todas as mensagens de feedback.

## Fase 6 — Fechamento
25. Rodar toda a suíte de testes unitários do `PixViewModel` e conferir contra os critérios de aceite da spec.
26. Executar teste manual end-to-end do fluxo completo (caminho feliz, erro de rede, saldo insuficiente) com VoiceOver ativado.
27. Se alguma decisão técnica mudar durante a implementação, atualizar `spec.md`/`plan.md` antes de prosseguir (regra de ouro do fluxo SDD).
