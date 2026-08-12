# Plan: Pix Transfer Redesign

## Visão geral do fluxo
A jornada é composta por 4 telas, navegadas sequencialmente por um Coordinator
dedicado, cobrindo as 4 user stories da spec:

1. **Seleção de destinatário** — lista de destinatários salvos ou entrada manual de chave Pix
2. **Valor** — informar o valor e visualizar o saldo disponível
3. **Revisão** — revisar destinatário + valor antes de confirmar
4. **Resultado** — feedback de sucesso ou erro

## Camadas (Clean Architecture)

### Domain (módulo Pix)
Regras de negócio, sem dependência de SwiftUI/UIKit/rede.
- **Entidades**: `PixRecipient`, `PixKey`, `TransferAmount`, `AccountBalance`, `PixTransferRequest`, `PixTransferResult`
- **Protocolos de repositório**: `PixRecipientRepository`, `AccountBalanceRepository`, `PixTransferRepository`
- **Use cases**: `FetchSavedRecipientsUseCase`, `ValidatePixKeyUseCase`, `FetchAccountBalanceUseCase`, `ValidateTransferAmountUseCase` (valor > 0 e ≤ saldo disponível), `ConfirmPixTransferUseCase`

### Data (módulo Pix, usando networking do Core)
Implementações concretas dos protocolos de repositório.
- DTOs e mappers DTO → Entidade de domínio
- `PixRecipientRepositoryImpl`, `AccountBalanceRepositoryImpl`, `PixTransferRepositoryImpl`
- Tratamento de erros de rede mapeado para tipos de erro de domínio (não expor erros de transporte para a Presentation)

### Presentation (módulo Pix, SwiftUI + MVVM)
Uma View + um ViewModel por tela, cada ViewModel testável isoladamente (sem SwiftUI/UIKit):
- `RecipientSelectionView` / `RecipientSelectionViewModel`
- `AmountEntryView` / `AmountEntryViewModel`
- `TransferReviewView` / `TransferReviewViewModel`
- `TransferResultView` / `TransferResultViewModel`

ViewModels só conversam com Use Cases (nunca diretamente com repositórios ou rede) e expõem estado observável (`@Published`) consumido pelas Views.

## Coordinator
- `PixTransferCoordinator` (UIKit, `UINavigationController`-based), cada SwiftUI View embrulhada em `UIHostingController`
- Responsável por: iniciar o fluxo (`start()`), avançar/retroceder entre as 4 telas, encerrar o fluxo e notificar o coordinator pai (delegate/closure) ao concluir ou cancelar
- ViewModels não navegam diretamente — comunicam intenção ao Coordinator via closures/delegate (ex.: `onRecipientSelected`, `onAmountConfirmed`, `onReviewConfirmed`, `onResultDismissed`)
- Nenhuma navegação deve ser criada fora deste Coordinator (regra do CLAUDE.md)

## Módulos afetados
- **Pix** — todo o código novo de Domain/Data/Presentation da feature e o Coordinator
- **Core** — reaproveitar cliente de rede e tipos de erro compartilhados; adicionar apenas o que for genérico o suficiente para outras features (ex.: formatação de moeda, se ainda não existir)
- **DesignSystem** — reaproveitar componentes existentes (campo de texto, lista de seleção, botão primário/secundário, banner de erro/sucesso); qualquer componente novo necessário deve ser adicionado aqui, nunca criado localmente dentro do módulo Pix
- **App** — registrar o `PixTransferCoordinator` e suas dependências no composition root/DI, e conectar o ponto de entrada existente do super-app à inicialização do fluxo

## Acessibilidade (WCAG 2.1 AA)
- VoiceOver: labels, hints e ordem de leitura definidos para cada View, com foco especial nas mensagens de erro/sucesso da tela de Resultado
- Dynamic Type: layouts validados em tamanhos de fonte de acessibilidade (não só nos tamanhos padrão)
- Erros nunca comunicados apenas por cor — sempre acompanhados de texto/ícone com label acessível

## Testes
- Testes unitários obrigatórios para as 4 ViewModels, cobrindo: estados de carregamento, validação (chave Pix, valor vs. saldo), sucesso e erro
- Fora do escopo desta fase: testes de UI automatizados (não exigidos pelos critérios de aceite da spec)

## Riscos técnicos
- Contratos de API (busca de destinatários, validação de chave, saldo, confirmação) ainda não definidos com o backend (`superapp-api`) — bloqueiam o início da camada Data
- Reuso de componentes do Design System depende de quais já existem hoje; falta de componente pode adicionar trabalho não estimado aqui
