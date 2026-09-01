# STATUS — Projeto Qualivida Serviços Médicos LTDA

> **Atualizado em:** 2026-09-01 · **Por:** Eduard / Fábio
> O painel do projeto: fase atual, progresso e o que precisa de atenção.

## Onde estamos

- **Fase atual:** 1 — Registro operacional e governança de responsáveis · aberta em 2026-08-19
- **Objetivo desta fase:** validar o registro mínimo de produção/prontuário, seus responsáveis, regras de acesso e o contrato de saída para a central financeira, usando fixtures controladas.
- **No prazo?** em preparação — execução técnica após pré-condições.

## Progresso da fase

- **Tasks:** 4/12 (33%)
- **Tasks concluídas:** F1-T001 — pré-condições da governança; F1-T002 — fixture e política mínima de escopo; F1-T003 — negações e privacidade; F1-T004 — auditoria de alteração e rollback de RLS.
- **Próxima task do champion:** F1-T005 — registrar contrato de campos mínimos (dono: cliente/administrador, SPEC-1-002).

## Contexto operacional confirmado

- **Empresa:** Qualivida Serviços Médicos LTDA — entidade principal no sistema.
- **Champion:** Fábio Schneider — CEO, líder do projeto e único responsável.
- **Unidades:** 5 no total; 2 com nome “Qualivida”; 3 com outros nomes, ainda não informados.
- **Usuários:** Fíbio é o único usuário com acesso no momento.
- **Superfície técnica:** ETHOS.
- **Runner de testes:** ETHOS.

## Travas ativas

| Trava | Desde | Quem resolve | Ação em curso |
|---|---|---|---|
| Nomes das 3 unidades restantes | 2026-08-31 | Fábio | Confirmar quando forem necessários para cadastro real |
| Campos mínimos e retenção da aplicação | 2026-08-19 | Fábio + Adapta | Definir antes de qualquer dado real — pré-condição direta da F1-T005 |

## Entregas concluídas

| Fase | O que foi entregue | Fechada em |
|---|---|---|
| Preparação | Escopo, SPECs, tasks e pasta operacional da Fase 1 | 2026-08-19 |
| F1-T001 | Pré-condições formalizadas; identidade, responsável, superfície e runner registrados | 2026-08-31 |
| F1-T002 | Fixture sintética e política mínima RN-1-001; cenário RLS-ALLOW demonstrado no Skip, sem dados reais | 2026-08-31 |
| F1-T003 | Negações de escopo (entidade, unidade, profissional) e rejeição de payload clínico demonstradas no Skip | 2026-08-31 |
| F1-T004 | Auditoria de alteração de escopo com versionamento e rollback sem apagar auditoria, demonstrados no Skip | 2026-09-01 |

## Próxima reunião

A definir — análise da F1-T005 sobre contrato de campos mínimos (SPEC-1-002), que exige decisões do cliente.