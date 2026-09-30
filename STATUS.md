# STATUS — Projeto Qualivida Serviços Médicos LTDA

> **Atualizado em:** 2026-09-01 · **Por:** Eduard / Fábio
> O painel do projeto: fase atual, progresso e o que precisa de atenção.

## Onde estamos

- **Fase atual:** 1 — Registro operacional e governança de responsáveis · aberta em 2026-08-19
- **Objetivo desta fase:** validar o registro mínimo de produção/prontuário, seus responsáveis, regras de acesso e o contrato de saída para a central financeira, usando fixtures controladas.
- **No prazo?** em preparação — execução técnica após pré-condições.

## Progresso da fase

- **Tasks:** 5/12 (42%)
- **Tasks concluídas:** F1-T001 — pré-condições da governança; F1-T002 — fixture e política mínima de escopo; F1-T003 — negações e privacidade; F1-T004 — auditoria e rollback de RLS; F1-T005 — contrato de campos mínimos.
- **Próxima task do champion:** F1-T006 — criar registro operacional válido (SPEC-1-002, usa o contrato F1-FIELDS-BASELINE).

## Contexto operacional confirmado

- **Empresa:** Qualivida Serviços Médicos LTDA — entidade principal no sistema.
- **Champion:** Fábio Schneider — CEO, líder do projeto e único responsável.
- **Unidades:** 5 no total; 2 com nome “Qualivida”; 3 com outros nomes, ainda não informados.
- **Usuários:** Fábio é o único usuário com acesso no momento.
- **Superfície técnica:** ETHOS.
- **Runner de testes:** ETHOS.

## Travas ativas

| Trava | Desde | Quem resolve | Ação em curso |
|---|---|---|---|
| Nomes das 3 unidades restantes | 2026-08-31 | Fábio | Confirmar quando forem necessários para cadastro real |

## Entregas concluídas

| Fase | O que foi entregue | Fechada em |
|---|---|---|
| Preparação | Escopo, SPECs, tasks e pasta operacional da Fase 1 | 2026-08-19 |
| F1-T001 | Pré-condições formalizadas; identidade, responsável, superfície e runner registrados | 2026-08-31 |
| F1-T002 | Fixture sintética e política mínima RN-1-001; cenário RLS-ALLOW demonstrado no Skip, sem dados reais | 2026-08-31 |
| F1-T003 | Negações de escopo (entidade, unidade, profissional) e rejeição de payload clínico demonstradas no Skip | 2026-08-31 |
| F1-T004 | Auditoria de alteração de escopo com versionamento e rollback sem apagar auditoria, demonstrados no Skip | 2026-09-01 |
| F1-T005 | Contrato F1-FIELDS-BASELINE: catálogo próprio por clínica (27 serviços, 21 pagadores), guia individual + record_id, retenção permanente, RN-1-005/006/007, código TUSS opcional | 2026-09-01 |

## Próxima reunião

A definir — análise da F1-T006 sobre criação de registro operacional válido com o contrato fechado.