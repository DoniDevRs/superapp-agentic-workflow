# SuperApp Agentic Lab — Project Context

## What this project is
A study lab that redesigns the Pix Transfer journey of a fictional banking
super-app. The dual goal is (1) to deliver that journey and (2) to validate
a specification-driven development workflow (SDD — Spec-Driven Development)
executed by specialized AI agents, described in the
[SDD Workflow](#sdd-workflow-spec-driven-development) section below.

## Architecture
- Clean Architecture + MVVM-C
- Modules:
  - **App** — application shell, dependency composition, entry point
  - **Core** — utilities, networking, and models shared across modules
  - **DesignSystem** — reusable UI components (single source of truth for UI)
  - **Pix** — feature module for the Pix Transfer journey
- SwiftUI for new screens, UIKit for navigation (Coordinator pattern)

## SDD Workflow (Spec-Driven Development)
Every feature in this project goes through the five stages below, in this
order. Each stage produces a reviewable artifact in `/specs`, and the next
stage only begins after the previous artifact has been approved — no agent
should skip stages or implement from a spec that's still a draft.

1. **Spec** — describes the *what* and *why* of the feature: user problem,
   expected behavior, acceptance criteria, and business constraints. No
   implementation details. Lives in `/specs/<feature>/spec.md`.
2. **Plan** — translates the spec into a technical decision: which modules
   are affected, API/DTO contracts, Design System components to reuse (or
   create), Coordinator navigation points, and technical risks. Lives in
   `/specs/<feature>/plan.md`.
3. **Tasks** — breaks the plan into small, sequenceable, independently
   verifiable units of work (e.g., "create ViewModel X", "add route Y to
   the Coordinator"). Lives in `/specs/<feature>/tasks.md`.
4. **Implement** — agent(s) implement the tasks one by one, following the
   rules in the section below (Coordinator, Design System, accessibility).
5. **Test** — ViewModel tests and accessibility validation (VoiceOver +
   Dynamic Type) for each implemented task, before considering it done.

> Golden rule: if an agent finds ambiguity at any stage, it must go back to
> the previous stage's artifact and clarify it there — never resolve the
> ambiguity "in the code" during implementation.

## Rules every agent must follow
- Follow the [SDD Workflow](#sdd-workflow-spec-driven-development): never implement a feature without an approved spec, plan, and tasks in `/specs`
- Never create navigation outside the Coordinator
- Every new screen needs: accessibility (VoiceOver + Dynamic Type), ViewModel tests
- Don't duplicate visual components — always check the Design System first

## How this repository relates to the others
- **superapp-ios** — consumes the specs and rules defined here as the source of truth
- **superapp-design-system** — source of the visual components referenced by the Design System
- **superapp-api** — Java backend consumed by the app
