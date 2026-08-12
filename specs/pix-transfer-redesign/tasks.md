# Tasks: Pix Transfer Redesign

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

## Fase 3 — Presentation: tela a tela
15. `RecipientSelectionViewModel` + testes unitários (lista de salvos, entrada manual de chave, validação).
16. `RecipientSelectionView` (SwiftUI) usando componentes do Design System.
17. `AmountEntryViewModel` + testes unitários (exibição de saldo, validação de valor).
18. `AmountEntryView` (SwiftUI).
19. `TransferReviewViewModel` + testes unitários (agregação dos dados das etapas anteriores).
20. `TransferReviewView` (SwiftUI) com resumo revisável antes da confirmação.
21. `TransferResultViewModel` + testes unitários (estado de sucesso e de erro).
22. `TransferResultView` (SwiftUI) com mensagens claras de sucesso/erro.

## Fase 4 — Coordinator
23. Criar `PixTransferCoordinator` (UIKit) orquestrando as 4 telas via `UIHostingController`.
24. Definir `start()` e o callback/delegate de finalização do fluxo; integrar ao ponto de entrada existente do super-app.
25. Conectar os callbacks das ViewModels ao Coordinator (avançar, retroceder, cancelar o fluxo).

## Fase 5 — Acessibilidade
26. Revisar VoiceOver (labels, hints, ordem de leitura) nas 4 telas.
27. Revisar Dynamic Type nas 4 telas, incluindo tamanhos de fonte de acessibilidade.
28. Validar que erros/sucessos são comunicados além da cor (WCAG 2.1 AA) em todas as mensagens de feedback.

## Fase 6 — Fechamento
29. Rodar toda a suíte de testes unitários das ViewModels e conferir contra os critérios de aceite da spec.
30. Executar teste manual end-to-end do fluxo completo (caminho feliz, erro de rede, saldo insuficiente) com VoiceOver ativado.
31. Se alguma decisão técnica mudar durante a implementação, atualizar `spec.md`/`plan.md` antes de prosseguir (regra de ouro do fluxo SDD).
