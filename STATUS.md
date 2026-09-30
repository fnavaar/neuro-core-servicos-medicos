# STATUS — Projeto Qualivida Serviços Médicos LTDA

> **Atualizado em:** 2026-09-30 · **Por:** Eduard / Fábio
> O painel do projeto: fase atual, progresso e o que precisa de atenção.

## Onde estamos

- **Fase atual:** 1 — Registro operacional e governança de responsáveis · aberta em 2026-08-19
- **Objetivo desta fase:** validar o registro mínimo de produção/prontuário, seus responsáveis, regras de acesso e o contrato de saída para a central financeira, usando fixtures controladas.
- **No prazo?** em preparação — execução técnica após pré-condições.

## Progresso da fase

- **Tasks:** 10/12 (83%)
- **Tasks concluídas:** F1-T001 — pré-condições da governança; F1-T002 — fixture e política mínima de escopo; F1-T003 — negações e privacidade; F1-T004 — auditoria e rollback de RLS; F1-T005 — contrato de campos mínimos; F1-T006 — registro operacional válido; F1-T007 — bloqueios, bordas e versionamento; F1-T008 — evidência do registro para o contrato; F1-T009 — aprovação do contrato de saída; F1-T010 — envelope válido versionado.
- **Próxima task do champion:** F1-T011 — exercitar rejeição, idempotência e privacidade (SPEC-1-003, dono: Ethos).

## Contexto operacional confirmado

- **Empresa:** Qualivida Serviços Médicos LTDA — entidade principal no sistema.
- **Champion:** Fábio Schneider — CEO, líder do projeto e único responsável.
- **Unidades:** 5 no total; 2 com nome “Qualivida”; 3 com outros nomes, ainda não informados.
- **Usuários:** Fábio é o único usuário com acesso no momento.
- **Superfície técnica:** ETHOS.
- **Runner de testes:** ETHOS.
- **Central consumidora:** este próprio sistema (nova aplicação de prontuário/produção) — confirmado em 2026-09-29.

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
| F1-T006 | Registro operacional válido: módulo com dimensões mínimas da SPEC-1-002, validação VALIDO/BLOQUEADO, RLS reaproveitada, auditoria e versionamento; demonstrado no Skip v0.0.27/v0.0.28 | 2026-09-29 |
| F1-T007 | Bloqueios, bordas e versionamento: BLOQUEADO persistido com missing_fields, status/data/competência validados, conteúdo clínico rejeitado, correção de qualquer registro por nova versão, elegibilidade para a central (VALIDO + ATENDIDO); demonstrado no Skip v0.0.36 | 2026-09-29 |
| F1-T008 | Separação de evidência para o contrato: `separateForContract` divide elegíveis (VALIDO + ATENDIDO) de excluídos com motivo (BLOQUEADO, STATUS_SEM_PRODUCAO, RASCUNHO); fixture F1-BLOCKED-001; demonstrado no Skip v0.0.40 | 2026-09-29 |
| F1-T009 | Contrato de saída `production-record.v1` APROVADO pelo responsável financeiro: idempotência por `sistema:record_id:versão`, origem preservada, valores nulos aceitos nesta fase, privacidade na fronteira, versionamento sem emenda silenciosa; central consumidora = este próprio sistema; evidência em `05_entregas/F1-T009-evidencia.md` | 2026-09-29 |
| F1-T010 | Envelope `production-record.v1` determinístico: `buildProductionEnvelope` com origem, dimensões, `service_ref` do catálogo, audit e `idempotency_key`; determinismo provado por comparação dupla na tela; demonstrado no Skip v0.0.42 | 2026-09-30 |

## Próxima reunião

A definir — testes de rejeição, idempotência e privacidade (F1-T011) e handoff para a Fase 2 (F1-T012).
