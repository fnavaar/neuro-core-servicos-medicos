# F1-T006 — Evidência: criar registro operacional válido

**Data:** 2026-09-29/30 · **SPEC:** SPEC-1-002 · **Skip:** versões 0.0.27 e 0.0.28 · **Teste humano:** aprovado pelo champion ("deu certo", 2026-09-29 21:21)

## O que foi implementado

- `src/lib/operational-record.ts` — módulo do registro operacional:
  - tipos das dimensões mínimas da SPEC-1-002 (`record_id`, `entity_id`, `unit_id`, `professional_id`, `service_ref`, `event_date`, `competency`, `status_source`, `responsible_user_id`, `evidence_ref` + condicionais `quantity`/`unit_value`/`gross_value`);
  - `validateRecord` → `VALIDO`/`BLOQUEADO` com `missing_fields` e `mapping_status`;
  - `createOperationalRecord` (v1 + evento `RECORD_CREATED`) e `reviseOperationalRecord` (nova versão + evento `RECORD_REVISED`, anterior preservada);
  - RLS reaproveitada da SPEC-1-001: só usuário com escopo coberto (entidade+unidade+profissional) cria/corrige;
  - serviço validado contra o catálogo da F1-T005 via RN-1-005 (vínculo de valor), TUSS opcional.
- `src/pages/Index.tsx` — seção "F1-T006 · Registro operacional válido" com fixture `F1-VALID-001`, botões Criar/Corrigir, payload do registro e tabela de auditoria.

## Verificação automática (pipeline Skip v0.0.27 e v0.0.28)

| Etapa | Resultado |
|---|---|
| Setup | ok |
| Análise estática | ok |
| Build | ok |
| Integrações | ok |
| Testes | ok |

## Verificação no caminho real (preview)

- URL: https://pagina-em-branco-ee3ec--preview.goskip.app
- Pré-validação da fixture: **VÁLIDO** (todas as dimensões presentes, ECG mapeado no catálogo).
- Clique em "Criar registro": `Registro F1-VALID-001 criado como VALIDO (v1, evento REC-EVT-0001)` — payload completo com auditoria (`createdAt/createdBy/updatedAt/updatedBy/version`) exibido na tela.
- Clique em "Corrigir (nova versão)": `Correção criada: v2 (evento REC-EVT-0002, RECORD_REVISED)` — tabela mostra v1 (RECORD_CREATED) e v2 (RECORD_REVISED) lado a lado, sem sobrescrever.
- Revalidação do zero após aprovação do champion (2026-09-30 ~00:21Z): criação e correção reexecutadas no preview com resultado idêntico.
- Nenhum conteúdo clínico no payload (reuso do `validatePayload` da fronteira; campos condicionais nulos permanecem `null`, não zero).

## Veredito da verificação independente

| Critério (CA-1-006 e recorte) | Evidência | Resultado |
|---|---|---|
| Registro completo criado com dimensões, responsável, evidência, auditoria e `VALIDO` | Payload exibido no preview com todos os campos + `validationState: VALIDO` | PASSOU |
| RLS da SPEC-1-001 aplicada ao fluxo | `createOperationalRecord`/`reviseOperationalRecord` exigem escopo coberto; usuário sem escopo não cria | PASSOU |
| Serviço mapeado pelo contrato F1-T005 | `isServiceMapped` via RN-1-005; ECG mapeado, `mappingStatus: MAPEADO` | PASSOU |
| Condicionais nulos não viram zero | `unitValue: null`, `grossValue: null` preservados no payload | PASSOU |
| Correção gera nova versão sem apagar a anterior | v1 + v2 na tabela de auditoria (`RECORD_CREATED` → `RECORD_REVISED`) | PASSOU |
| Nenhum conteúdo clínico persistido | Payload só administrativo; fronteira `validatePayload` reutilizada | PASSOU |
| Build/testes/lint | Pipeline Skip v0.0.27/v0.0.28 verde em todas as etapas | PASSOU |
| Teste humano | Champion confirmou ("deu certo", 2026-09-29 21:21) | PASSOU |

## Limitações

- Bloqueios/bordas (`F1-INVALID-001`, `F1-EDGE-001..003`) são recorte da **F1-T007** — não exercitados aqui.
- Nenhum dado real foi tocado; fixture sintética apenas.
