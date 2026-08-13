# Plan: Pix Transfer Redesign

> **Nota de atualização (2026-08-13):** este plano propunha originalmente 4
> telas, cada uma com seu próprio ViewModel. O protótipo aprovado
> (`design/images/pix-depois.png`) consolidou as etapas de **Valor** e
> **Revisão** em uma única tela, resultando em um fluxo de 3 telas. A
> implementação seguiu o protótipo (fonte de verdade de UX) em vez desta
> versão do plano — as seções abaixo foram atualizadas para refletir a
> implementação real. Ver também a nota equivalente em `tasks.md`.

## Visão geral do fluxo
A jornada é composta por 3 telas, navegadas sequencialmente por um Coordinator
dedicado, cobrindo as 4 user stories da spec (a etapa de "Valor" e a de
"Revisão" da spec original são atendidas por uma única tela, que reúne
digitação de valor e resumo revisável antes da confirmação):

1. **Seleção de destinatário** (`SelectRecipientView`) — lista de destinatários salvos ou entrada manual de chave Pix
2. **Valor + Revisão** (`ReviewPaymentView`) — informar o valor, visualizar o saldo disponível e revisar destinatário + valor antes de confirmar
3. **Confirmação** (`ConfirmationView`) — feedback de sucesso ou erro

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
Um único `PixViewModel`, compartilhado pelas 3 telas do fluxo (injetado pelo
Coordinator na mesma instância em cada View), em vez de um ViewModel por
tela — o estado (destinatário selecionado, valor, comprovante) atravessa a
navegação sem precisar ser retransmitido manualmente entre ViewModels:
- `SelectRecipientView`
- `ReviewPaymentView`
- `ConfirmationView`
- `PixViewModel` (único, `@MainActor`, `@Published`)

O `PixViewModel` só conversa com Use Cases (nunca diretamente com
repositórios ou rede) e expõe estado observável (`@Published`) consumido
pelas 3 Views.

## Coordinator
- `PixCoordinator` (UIKit, `UINavigationController`-based), cada SwiftUI View embrulhada em `UIHostingController`
- Responsável por: iniciar o fluxo (`start()`), avançar/retroceder entre as 3 telas, encerrar o fluxo e notificar o coordinator pai (delegate/closure) ao concluir ou cancelar
- A View não navega diretamente — comunica intenção ao Coordinator via closures (ex.: `onRecipientSelected`, `onChangeRecipient`, `onTransferConfirmed`, `onFinish`, `onRepeat`)
- Nenhuma navegação deve ser criada fora deste Coordinator (regra do CLAUDE.md)

## Módulos afetados
- **Pix** — todo o código novo de Domain/Data/Presentation da feature e o Coordinator
- **Core** — reaproveitar cliente de rede e tipos de erro compartilhados; adicionar apenas o que for genérico o suficiente para outras features (ex.: formatação de moeda, se ainda não existir)
- **DesignSystem** — reaproveitar componentes existentes (campo de texto, lista de seleção, botão primário/secundário, banner de erro/sucesso); qualquer componente novo necessário deve ser adicionado aqui, nunca criado localmente dentro do módulo Pix
- **App** — registrar o `PixCoordinator` e suas dependências no composition root/DI, e conectar o ponto de entrada existente do super-app à inicialização do fluxo

## Acessibilidade (WCAG 2.1 AA)
- VoiceOver: labels, hints e ordem de leitura definidos para cada View, com foco especial nas mensagens de erro/sucesso da tela de Confirmação
- Dynamic Type: layouts validados em tamanhos de fonte de acessibilidade (não só nos tamanhos padrão)
- Erros nunca comunicados apenas por cor — sempre acompanhados de texto/ícone com label acessível

## Testes
- Testes unitários obrigatórios para o `PixViewModel` compartilhado, cobrindo os cenários das 3 telas: estados de carregamento, validação (chave Pix, valor vs. saldo), sucesso e erro
- Fora do escopo desta fase: testes de UI automatizados (não exigidos pelos critérios de aceite da spec)

## Riscos técnicos
- Contratos de API (busca de destinatários, validação de chave, saldo, confirmação) ainda não definidos com o backend (`superapp-api`) — bloqueiam o início da camada Data
- Reuso de componentes do Design System depende de quais já existem hoje; falta de componente pode adicionar trabalho não estimado aqui
