# Fase 1 — Tarefas gerais

**Estado da fase:** F1-T001 a F1-T010 concluídas; F1-T011 é a próxima task, ainda não iniciada.  
**Regra de execução:** uma task por vez, somente após a pré-condição da linha estar comprovada; nenhum dado real, integração ou decisão ausente pode ser inventado.

## Tasks

| ID | Task | Dono | SPEC | Critério binário | Pré-condições | Status |
|---|---|---|---|---|---|---|
| F1-T001 | Registrar pré-condições da governança | Cliente — administrador | SPEC-1-001 | Matriz RLS, identidade oficial, superfície técnica e runner registrados ou ausência formalizada com dono e prazo | Gate de escopo aprovado; participação do cliente | concluída — 2026-08-31 |
| F1-T002 | Criar fixture e política mínima de escopo | Ethos | SPEC-1-001 | Fixture sintética contém entidade, unidade, profissional, usuários e escopos; usuário autorizado lê o registro-alvo | F1-T001 concluída; ambiente de teste autorizado | concluída — 2026-08-31 |
| F1-T003 | Exercitar negações e privacidade | Ethos | SPEC-1-001 | Usuários sem entidade, unidade ou profissional recebem negação e payload clínico não é persistido | F1-T002 concluída | concluída — 2026-08-31 |
| F1-T004 | Auditar alteração e demonstrar rollback de RLS | Ethos | SPEC-1-001 | Alteração sensível registra versões e rollback restaura política anterior sem apagar auditoria | F1-T002 e F1-T003 concluídas | concluída — 2026-09-01 |
| F1-T005 | Registrar contrato de campos mínimos | Cliente — administrador | SPEC-1-002 | Campos mínimos, identidade oficial, fronteira clínico/administrativo e retenção registrados | F1-T001 concluída; SPEC-1-001 desbloqueada | concluída — 2026-09-01 |
| F1-T006 | Criar registro operacional válido | Ethos | SPEC-1-002 | Fixture completa produz registro `VALIDO` com dimensões, responsável, evidência, auditoria e versão | F1-T005 e F1-T002 concluídas | concluída — 2026-09-29 |
| F1-T007 | Exercitar bloqueios, bordas e versionamento | Ethos | SPEC-1-002 | Ausência/ambiguidade/status desconhecido bloqueia; correção cria nova versão; conteúdo clínico é rejeitado | F1-T006 concluída | concluída — 2026-09-29 |
| F1-T008 | Entregar evidência do registro para o contrato | Analista financeiro | SPEC-1-002 | Somente registro `VALIDO` é separado para SPEC-1-003 e registros `BLOQUEADO` permanecem fora da saída | F1-T006 e F1-T007 concluídas | concluída — 2026-09-29 |
| F1-T009 | Registrar aprovação do contrato de saída | Responsável financeiro | SPEC-1-003 | A versão `production-record.v1`, campos obrigatórios e regra de idempotência estão aprovados ou rejeitados com motivo | F1-T001 concluída; central consumidora identificada | concluída — 2026-09-29 |
| F1-T010 | Gerar envelope válido versionado | Ethos | SPEC-1-003 | Registro `VALIDO` gera envelope completo, determinístico e com `idempotency_key` | F1-T008 e F1-T009 concluídas; runner disponível | concluída — 2026-09-30 |
| F1-T011 | Exercitar rejeição, idempotência e privacidade | Ethos | SPEC-1-003 | Bloqueado/duplicado/identidade ambígua não gera saída; repetição não duplica; campo clínico é rejeitado | F1-T010 concluída | bloqueada |
| F1-T012 | Fechar evidências e handoff para a Fase 2 | Analista financeiro | SPEC-1-003 | Pacote contém contrato, fixtures, rejeições, idempotência, aprovação e pendências; nenhuma API/PDF foi usada | F1-T009, F1-T010 e F1-T011 concluídas | bloqueada |

## Contexto confirmado para a próxima task

- Qualivida é a entidade principal.
- Fábio Schneider é CEO, líder do projeto e único responsável.
- Existem 5 unidades: 2 chamadas Qualivida e 3 com nomes ainda pendentes.
- Fábio é o único usuário com acesso no momento.
- Superfície e runner: ETHOS.
- Contrato F1-FIELDS-BASELINE fechado na F1-T005: catálogo próprio por clínica (27 serviços, 21 pagadores), `guide_number` + `record_id`, retenção permanente, RN-1-005 (vínculo = agendável), RN-1-006 (catálogo por clínica), RN-1-007 (vigência imediata/futura/retroativa), código TUSS opcional (pacote contratual aceito).
- Registro operacional da F1-T006/F1-T007/F1-T008: módulo `src/lib/operational-record.ts` com validação `VALIDO`/`BLOQUEADO`, rejeição de conteúdo clínico, correção versionada, gate `isEligibleForCentral` (VALIDO + ATENDIDO) e `separateForContract` (elegíveis vs excluídos com motivo).
- **Contrato de saída aprovado (F1-T009):** `production-record.v1` APROVADO pelo responsável financeiro (2026-09-29 22:10); central consumidora = este próprio sistema; idempotência por `sistema:record_id:versão`; valores nulos aceitos nesta fase.
- **Envelope implementado (F1-T010):** `buildProductionEnvelope` determinístico com `idempotency_key = nova-aplicacao-prontuario-producao:record_id:versão`, `service_ref` resolvido do catálogo e determinismo provado por comparação dupla na tela; demonstrado no Skip v0.0.42.

## Próximo passo

Analisar F1-T011 em uma nova rodada autorizada: exercitar rejeição (F1-BLOCKED-001), duplicidade (F1-DUPLICATE-001), nulos (F1-NULL-VALUE-001) e privacidade (F1-PRIVACY-001) no contrato. Não exercitar cenários fora do recorte antes da análise e autorização específicas da task.
