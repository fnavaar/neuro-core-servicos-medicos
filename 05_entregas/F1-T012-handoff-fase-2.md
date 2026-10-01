# F1-T012 — Handoff: fechamento de evidências da Fase 1 e preparação da Fase 2

**Data:** 2026-10-01 · **SPEC:** SPEC-1-003 · **Dono da task:** analista financeiro · **Execução:** Ethos (Eduard), aprovada pelo champion Fábio Schneider
**ID do pacote:** `F1-HANDOFF-001` · **Versão do sistema na data:** Skip v0.0.44 (pipeline verde)

## 1. O que este pacote é

Pacote de fechamento da Fase 1 (Registro operacional e governança de responsáveis). Ele consolida o que a Fase 1 provou — contrato aprovado, fixtures, rejeições, idempotência, aprovação — e o que a Fase 2 herda como pendência. Nada aqui é integração: a Fase 1 não usou API, PDF/pastas, credencial ou escrita na central (seção 8).

## 2. Contrato aprovado

- **Contrato:** `production-record.v1` — **APROVADO** pelo responsável financeiro em 2026-09-29 22:10 (evidência `F1-CONTRACT-APPROVAL`, em `05_entregas/F1-T009-evidencia.md`).
- **Central consumidora:** este próprio sistema (nova aplicação de prontuário/produção) — decisão do champion em 2026-09-29.
- **Idempotência:** `idempotency_key = sistema:record_id:versão` (ex.: `nova-aplicacao-prontuario-producao:F1-VALID-001:1`). Repetição da mesma versão devolve o MESMO envelope como `DUPLICADO`, sem nova saída; correção gera versão nova, que gera chave nova.
- **Valores:** `quantity`, `unit_value` e `gross_value` nulos são preservados como nulos — nunca estimados (CA-1-015).
- **Versionamento do contrato:** mudança no contrato = nova versão, mantendo a anterior; nenhuma emenda silenciosa no mesmo `contract_version` (RN-1-008).

### Envelope de referência (forma aprovada)

```json
{
  "contract_version": "production-record.v1",
  "record_id": "REC-TEST-001",
  "source": {
    "system": "nova-aplicacao-prontuario-producao",
    "source_record_id": "REC-TEST-001",
    "version": 1,
    "evidence_ref": "fixture://fase-1/REC-TEST-001"
  },
  "entity_id": "ENTITY-TEST-001",
  "unit_id": "UNIT-TEST-001",
  "professional_id": "PROF-TEST-001",
  "service_ref": {"id": "SERVICE-TEST-001", "label": "servico-controlado"},
  "event_date": "2026-08-01",
  "competency": "2026-08",
  "quantity": null,
  "unit_value": null,
  "gross_value": null,
  "status_source": "ATENDIDO",
  "validation_state": "VALIDO",
  "responsible_user_id": "USER-RESP-001",
  "audit": {"created_by": "USER-CREATE-001", "updated_by": "USER-CREATE-001", "version": 1},
  "idempotency_key": "nova-aplicacao-prontuario-producao:REC-TEST-001:1"
}
```

## 3. Implementação de referência (onde o contrato vive)

| Componente | Local | Papel |
|---|---|---|
| Módulo do contrato | `src/lib/operational-record.ts` (Skip v0.0.44) | `validateRecord`, `createOperationalRecord`, `reviseOperationalRecord`, `isEligibleForCentral` (VALIDO + ATENDIDO), `separateForContract`, `buildProductionEnvelope`, `issueEnvelope`, registro de idempotência em memória |
| Catálogo da clínica | `src/lib/service-catalog.ts` | Catálogo próprio por clínica da F1-T005 (27 serviços, 21 pagadores, RN-1-005/006/007) |
| Identidade e RLS | `src/lib/rls-fixture.ts` | Fixture sintética da F1-T002; base das negações e da identidade oficial |
| Demonstração | `src/pages/Index.tsx` | Seções F1-T006 a F1-T011 com todos os cenários executáveis |

## 4. Mapa de fixtures e provas

| Fixture | Cenário | Prova observada | Onde vive |
|---|---|---|---|
| `F1-VALID-001` | Positivo (GREEN) | Envelope determinístico; duas gerações idênticas, mesma chave | `recordFixtures.valid` (módulo); tela F1-T010; evidência F1-T010 |
| `F1-INVALID-001` | Identidade não confirmada + serviço sem vínculo + status desconhecido | Nasce `BLOQUEADO` na criação; nunca gera envelope (invariante end-to-end) | `f1t007Fixtures.invalid`; tela F1-T007; evidência F1-T007 |
| `F1-EDGE-001` | Condicionais ausentes | Nulos preservados, nunca convertidos em zero | `f1t007Fixtures.edgeNull`; evidência F1-T007 |
| `F1-EDGE-002` | AGENDADO pós-data | Status preservado sem reinterpretação; pendência mantida; não elegível | `f1t007Fixtures.edgeLate`; tela F1-T008; evidências F1-T007/F1-T008 |
| `F1-EDGE-003` / `F1-PRIVACY-001` | Conteúdo clínico na entrada | Rejeitado sem persistir — na criação e na fronteira do contrato | `f1t007Fixtures.edgeClinical`; tela F1-T011; evidências F1-T007/F1-T011 |
| `F1-BLOCKED-001` | Registro bloqueado | Fora da saída com motivo `BLOQUEADO`; no contrato, `REJEITADO (NAO_ELEGIVEL)` | `f1t007Fixtures.blocked`; tela F1-T008/F1-T011; evidências F1-T008/F1-T011 |
| `F1-DUPLICATE-001` | Idempotência | 1ª emissão `EMITIDO`; 2ª `DUPLICADO` com o mesmo envelope; contador fixo em 1 | Tela F1-T011 (label do cenário); registro de chaves em memória; evidência F1-T011 |
| `F1-NULL-VALUE-001` | Nulos no envelope | `EMITIDO` com `null` preservados (CA-1-015) | `f1t007Fixtures.nullValue`; tela F1-T011; evidência F1-T011 |
| `F1-CONTRACT-APPROVAL` | Aprovação do contrato | Versão, campos e idempotência aprovados pelo responsável financeiro | `05_entregas/F1-T009-evidencia.md` |

Todas as fixtures são sintéticas, sem dado pessoal real, referenciadas por `fixture://fase-1/`.

## 5. Rejeições e regras do contrato

| Código/resultado | Quando ocorre | Garantia |
|---|---|---|
| `NAO_ELEGIVEL` | Registro `BLOQUEADO` ou status sem produção (`AGENDADO`/`CANCELADO`/`FALTOU`) | Nunca há saída parcial (RN-1-006) |
| `IDENTIDADE_AMBIGUA` | Entidade/unidade/profissional não confirmadas na camada do contrato | Defesa em profundidade; na prática a criação já bloqueia antes de o registro existir (invariante provado end-to-end) |
| `CLINICAL_CONTENT` | Marcador clínico detectado na fronteira | Nada persistido; só a categoria do erro é reportada (RN-1-007/CA-1-016) |
| `DUPLICADO` | Chave de idempotência já emitida | O MESMO envelope é devolvido; nenhuma saída nova (RN-1-005/CA-1-013) |

## 6. Idempotência — estado atual e limite

- **Fase 1:** registro de chaves em memória (`Map` por `idempotency_key`), demonstrado na tela com contador fixo em 1 na repetição.
- **Fase 2:** persistir o registro de emissão em store real antes de qualquer transporte; a chave e a regra não mudam.

## 7. Pendências herdadas para a Fase 2

| Pendência | Origem | Dono sugerido |
|---|---|---|
| API/endpoint, autenticação e credencial para transporte | SPEC-1-003 (fora de escopo na Fase 1) | Responsável financeiro / administrador |
| Persistência real do registro de idempotência | Limite declarado na F1-T011 | Ethos |
| Nomes das 3 unidades restantes | F1-T001 / STATUS.md | Fábio Schneider |
| Catálogo definitivo (planilha de referência com valores desatualizados) | F1-T005 | Responsável financeiro |
| Formato final do pacote de fechamento e indicadores | Preferências do champion | Fábio Schneider + consultor |
| Tecnologia da central e política de retenção no transporte | SPEC-1-003 ("Bloqueios") | Responsável financeiro / administrador |

## 8. Declaração: nenhuma API/PDF foi usada na Fase 1

Verificado por inspeção do módulo do contrato em 2026-10-01 (Skip v0.0.44): **nenhuma** chamada de rede, endpoint, token, credencial, leitura de pasta ou PDF em `src/lib/operational-record.ts` — os únicos imports são internos (`./service-catalog`, `./rls-fixture`). Nenhuma credencial foi provisionada, nenhuma escrita na central foi feita e nenhum dado real foi tocado (apenas fixtures sintéticas). Conforme CA-1-017 e o limite declarado na SPEC-1-003.

## 9. Checklist do critério binário (F1-T012)

- [x] **Contrato** — seção 2 (`production-record.v1` aprovado, F1-T009)
- [x] **Fixtures** — seção 4 (mapa completo com local das provas)
- [x] **Rejeições** — seção 5
- [x] **Idempotência** — seções 2, 5 e 6 (prova: contador fixo em 1)
- [x] **Aprovação** — `F1-CONTRACT-APPROVAL` (F1-T009)
- [x] **Pendências** — seção 7
- [x] **Nenhuma API/PDF** — seção 8 (inspeção do módulo)

## 10. Como a Fase 2 deve consumir

1. Ler somente envelopes `EMITIDO` com `contract_version: "production-record.v1"`.
2. Respeitar a chave de idempotência: repetição nunca é reprocessada como nova saída.
3. Qualquer campo novo exige emenda de contrato (nova versão + aprovação) — nunca adaptar o payload silenciosamente.
4. Rejeições continuam estruturadas e monitoradas por origem; ausência de erro nunca é considerada aceite.

## 11. Índice das evidências da Fase 1

| Task | Evidência | Skip |
|---|---|---|
| F1-T001 | `05_entregas/F1-T001-evidencia.md` | — |
| F1-T002 | registro no changelog (padrão da época) | v0.0.6 |
| F1-T003 | `05_entregas/F1-T003-evidencia.md` | v0.0.9 |
| F1-T004 | `05_entregas/F1-T004-evidencia.md` | v0.0.13 |
| F1-T005 | `05_entregas/F1-T005-evidencia.md` | v0.0.25 |
| F1-T006 | `05_entregas/F1-T006-evidencia.md` | v0.0.27/28 |
| F1-T007 | `05_entregas/F1-T007-evidencia.md` + debug em `06_notas/debug/` | v0.0.36 |
| F1-T008 | `05_entregas/F1-T008-evidencia.md` | v0.0.40 |
| F1-T009 | `05_entregas/F1-T009-evidencia.md` | — |
| F1-T010 | `05_entregas/F1-T010-evidencia.md` | v0.0.42 |
| F1-T011 | `05_entregas/F1-T011-evidencia.md` | v0.0.43/44 |
| F1-T012 | este documento | v0.0.44 |

**Fase 1: 12/12 tasks concluídas.** O fechamento da fase depende de validação do consultor na próxima sincronização.
