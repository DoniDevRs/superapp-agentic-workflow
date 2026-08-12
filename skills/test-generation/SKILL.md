---
name: test-generation
description: Use para gerar testes unitários de ViewModels seguindo o padrão do projeto
---

Ao gerar testes de um ViewModel:
- Use um Mock do repositório/serviço correspondente
- Cubra o caminho feliz e pelo menos 2 cenários de erro
- Nomeie os testes no formato test_<condição>_<resultadoEsperado>
- Não teste detalhes de UI, só estado e lógica do ViewModel