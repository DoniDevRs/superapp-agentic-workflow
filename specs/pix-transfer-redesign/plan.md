# Plan: Pix Transfer Redesign

> **Update note (2026-08-13):** this plan originally proposed 4 screens, each
> with its own ViewModel. The approved prototype
> (`design/images/pix-depois.png`) consolidated the **Amount** and
> **Review** steps into a single screen, resulting in a 3-screen flow. The
> implementation followed the prototype (source of truth for UX) instead of
> this version of the plan — the sections below have been updated to reflect
> the actual implementation. See also the equivalent note in `tasks.md`.

## Flow overview
The journey consists of 3 screens, navigated sequentially by a dedicated
Coordinator, covering the spec's 4 user stories (the spec's original
"Amount" and "Review" steps are handled by a single screen, which combines
amount entry and a reviewable summary before confirmation):

1. **Recipient selection** (`SelectRecipientView`) — list of saved recipients or manual Pix key entry
2. **Amount + Review** (`ReviewPaymentView`) — enter the amount, view the available balance, and review recipient + amount before confirming
3. **Confirmation** (`ConfirmationView`) — success or error feedback

## Layers (Clean Architecture)

### Domain (Pix module)
Business rules, with no dependency on SwiftUI/UIKit/networking.
- **Entities**: `PixRecipient`, `PixKey`, `TransferAmount`, `AccountBalance`, `PixTransferRequest`, `PixTransferResult`
- **Repository protocols**: `PixRecipientRepository`, `AccountBalanceRepository`, `PixTransferRepository`
- **Use cases**: `FetchSavedRecipientsUseCase`, `ValidatePixKeyUseCase`, `FetchAccountBalanceUseCase`, `ValidateTransferAmountUseCase` (amount > 0 and ≤ available balance), `ConfirmPixTransferUseCase`

### Data (Pix module, using Core's networking)
Concrete implementations of the repository protocols.
- DTOs and DTO → domain entity mappers
- `PixRecipientRepositoryImpl`, `AccountBalanceRepositoryImpl`, `PixTransferRepositoryImpl`
- Network error handling mapped to domain error types (transport errors not exposed to Presentation)

### Presentation (Pix module, SwiftUI + MVVM)
A single `PixViewModel`, shared across the flow's 3 screens (injected by the
Coordinator as the same instance into each View), instead of one ViewModel
per screen — state (selected recipient, amount, receipt) flows across
navigation without needing to be manually relayed between ViewModels:
- `SelectRecipientView`
- `ReviewPaymentView`
- `ConfirmationView`
- `PixViewModel` (single instance, `@MainActor`, `@Published`)

`PixViewModel` only talks to Use Cases (never directly to repositories or
the network) and exposes observable state (`@Published`) consumed by the 3
Views.

## Coordinator
- `PixCoordinator` (UIKit, `UINavigationController`-based), each SwiftUI View wrapped in a `UIHostingController`
- Responsible for: starting the flow (`start()`), moving forward/back between the 3 screens, ending the flow, and notifying the parent coordinator (delegate/closure) on completion or cancellation
- The View doesn't navigate directly — it communicates intent to the Coordinator via closures (e.g., `onRecipientSelected`, `onChangeRecipient`, `onTransferConfirmed`, `onFinish`, `onRepeat`)
- No navigation should be created outside this Coordinator (rule from CLAUDE.md)

## Affected modules
- **Pix** — all new Domain/Data/Presentation code for the feature and the Coordinator
- **Core** — reuse the shared network client and error types; add only what's generic enough for other features (e.g., currency formatting, if it doesn't already exist)
- **DesignSystem** — reuse existing components (text field, selection list, primary/secondary button, error/success banner); any new component needed must be added here, never created locally inside the Pix module
- **App** — register `PixCoordinator` and its dependencies in the composition root/DI, and wire the super-app's existing entry point to the flow's initialization

## Accessibility (WCAG 2.1 AA)
- VoiceOver: labels, hints, and reading order defined for each View, with special attention to the error/success messages on the Confirmation screen
- Dynamic Type: layouts validated at accessibility font sizes (not just the default sizes)
- Errors never communicated by color alone — always paired with text/icon with an accessible label

## Tests
- Unit tests required for the shared `PixViewModel`, covering the scenarios across the 3 screens: loading states, validation (Pix key, amount vs. balance), success, and error
- Out of scope for this phase: automated UI tests (not required by the spec's acceptance criteria)

## Technical risks
- API contracts (recipient lookup, key validation, balance, confirmation) not yet defined with the backend (`superapp-api`) — blocks the start of the Data layer
- Reuse of Design System components depends on what already exists today; a missing component may add unestimated work here
