# Feature: Pix Transfer Redesign

## Objective
Redesenhar a jornada de transferência via Pix para reduzir o número de passos
e melhorar clareza e acessibilidade.

## User stories
- Como usuário, quero selecionar um destinatário salvo ou digitar uma chave Pix.
- Como usuário, quero informar o valor e ver o saldo disponível.
- Como usuário, quero revisar os dados antes de confirmar.
- Como usuário, quero feedback claro de sucesso ou erro após a confirmação.

## Constraints
- Navegação via UIKit + Coordinator
- Telas construídas em SwiftUI
- Arquitetura MVVM-C
- WCAG 2.1 AA
- Testes unitários obrigatórios no ViewModel

## Acceptance Criteria
- Fluxo funciona 100% com VoiceOver
- Suporta Dynamic Type
- ViewModel coberto por testes unitários
- Erros de rede/saldo tratados com mensagens claras