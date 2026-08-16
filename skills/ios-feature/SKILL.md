---
name: ios-feature
description: Use when implementing a new iOS feature in this project
---

When implementing a new feature:
1. Read the corresponding spec at specs/<feature>/spec.md
2. Check whether a component already exists in the Design System before creating a new one
3. Follow Clean Architecture + MVVM-C
4. All new navigation must go through the Coordinator, never done directly in the View
5. Generate ViewModel unit tests alongside the implementation
6. At the end, run an accessibility check (see the accessibility-audit skill)
