# Feature: Pix Transfer Redesign

## Objective
Redesign the Pix transfer journey to reduce the number of steps and improve
clarity and accessibility.

## User stories
- As a user, I want to select a saved recipient or type in a Pix key.
- As a user, I want to enter the amount and see my available balance.
- As a user, I want to review the details before confirming.
- As a user, I want clear feedback of success or failure after confirming.

## Constraints
- Navigation via UIKit + Coordinator
- Screens built in SwiftUI
- MVVM-C architecture
- WCAG 2.1 AA
- Unit tests required on the ViewModel

## Acceptance Criteria
- The flow works 100% with VoiceOver
- Supports Dynamic Type
- ViewModel covered by unit tests
- Network/balance errors handled with clear messages
