# Tasks: Pix Transfer Redesign

> **Update note (2026-08-13):** Phase 3 originally described 4 View/ViewModel
> pairs (one screen and one ViewModel per step). The approved prototype
> (`design/images/pix-depois.png`) consolidated "Amount" and "Review" into a
> single screen, and the implementation used a single shared `PixViewModel`
> across the 3 screens instead of one ViewModel per screen. Phase 3, Phase 4
> (Coordinator, now `PixCoordinator`), and Phase 5 (Accessibility) were
> renumbered and updated below to reflect this. See also the equivalent
> note in `plan.md`.

Recommended sequential order. Each task should be completed (and, where
applicable, tested) before starting the next.

## Phase 0 — Preparation
1. Survey the Design System for which components already cover the flow (text field, selection list, primary/secondary button, error/success banner) and list what's missing.
2. Align with the backend (`superapp-api`) on the contracts for: saved recipient lookup, Pix key validation, balance inquiry, and transfer confirmation.

## Phase 1 — Domain (Pix module)
3. Create the domain entities: `PixRecipient`, `PixKey`, `TransferAmount`, `AccountBalance`, `PixTransferRequest`, `PixTransferResult`.
4. Create the `PixRecipientRepository`, `AccountBalanceRepository`, `PixTransferRepository` protocols.
5. Implement `FetchSavedRecipientsUseCase`.
6. Implement `ValidatePixKeyUseCase`.
7. Implement `FetchAccountBalanceUseCase`.
8. Implement `ValidateTransferAmountUseCase` (amount > 0 and ≤ available balance).
9. Implement `ConfirmPixTransferUseCase`.

## Phase 2 — Data (Pix module)
10. Create DTOs and DTO → domain entity mappers for recipients, balance, and transfer.
11. Implement `PixRecipientRepositoryImpl` on top of Core's network client.
12. Implement `AccountBalanceRepositoryImpl`.
13. Implement `PixTransferRepositoryImpl`, including mapping network errors to domain errors.
14. Register repositories and use cases in the App module's DI container.

## Phase 3 — Presentation: single ViewModel + 3 screens
15. `PixViewModel` (single instance, shared across the 3 screens) + unit tests (saved recipient list, manual key entry/validation, balance display, amount validation, aggregating data for review, success and error states).
16. `SelectRecipientView` (SwiftUI) using Design System components.
17. `ReviewPaymentView` (SwiftUI) — consolidates amount + review into a single screen, with a reviewable summary before confirmation.
18. `ConfirmationView` (SwiftUI) with clear success/error messages.

## Phase 4 — Coordinator
19. Create `PixCoordinator` (UIKit) orchestrating the 3 screens via `UIHostingController`.
20. Define `start()` and the flow-completion callback/delegate; wire it to the super-app's existing entry point.
21. Connect `PixViewModel`'s callbacks to the Coordinator (advance, go back, cancel the flow).

## Phase 5 — Accessibility
22. Review VoiceOver (labels, hints, reading order) across the 3 screens.
23. Review Dynamic Type across the 3 screens, including accessibility font sizes.
24. Validate that errors/successes are communicated beyond color (WCAG 2.1 AA) in all feedback messages.

## Phase 6 — Closeout
25. Run the full `PixViewModel` unit test suite and check it against the spec's acceptance criteria.
26. Run a manual end-to-end test of the full flow (happy path, network error, insufficient balance) with VoiceOver enabled.
27. If any technical decision changes during implementation, update `spec.md`/`plan.md` before proceeding (golden rule of the SDD workflow).
