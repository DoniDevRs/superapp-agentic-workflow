---
name: ios-feature
description: Use ao implementar uma nova feature iOS neste projeto
---

Ao implementar uma nova feature:
1. Leia a spec correspondente em specs/<feature>/spec.md
2. Verifique se algum componente já existe no Design System antes de criar um novo
3. Siga Clean Architecture + MVVM-C
4. Toda navegação nova deve passar pelo Coordinator, nunca ser feita direto na View
5. Gere testes unitários do ViewModel junto com a implementação
6. Ao final, rode uma checagem de acessibilidade (ver skill accessibility-audit)