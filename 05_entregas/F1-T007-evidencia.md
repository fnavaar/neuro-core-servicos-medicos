# F1-T007 — Evidência: bloqueios, bordas e versionamento

**Data:** 2026-09-29/30 · **SPEC:** SPEC-1-002 · **Skip:** v0.0.29–v0.0.36 · **Teste humano:** aprovado pelo champion ("para mim parece ok", 2026-09-29 21:47, após 2 rodadas de debug) + varredura completa solicitada e executada

## O que foi implementado

- `src/lib/operational-record.ts` (endurecido):
  - validação runtime de `status_source` desconhecido, `event_date` (AAAA-MM-DD) e `competency` (AAAA-MM) — nunca reinterpretados;
  - registro com ausência/ambiguidade é **persistido como `BLOQUEADO`** com `missing_fields` e fica fora da saída;
  - `detectClinicalContent` varre chaves e valores na fronteira; conteúdo clínico é rejeitado sem persistir nada;
  - `reviseOperationalRecord` aceita correção de registro `VALIDO` **ou** `BLOQUEADO` (CA-1-009 não restringe) — nova versão com revalidação completa;
  - `isEligibleForCentral`: `VALIDO` **e** `status_source = ATENDIDO` — pendência mantida para AGENDADO/CANCELADO/FALTOU;
  - escopo RLS compara apenas dimensões **presentes**: ausência vira `missing_fields`, divergência vira negação de RLS.
- `src/pages/Index.tsx` — seção "F1-T007 · Bloqueios, bordas e versionamento" com botões por cenário, "Corrigir registro (nova versão)", payload do registro e auditoria com motivo.

## Verificação automática (pipeline Skip v0.0.29–v0.0.36)

Todas as etapas (setup, análise estática, build, integrações, testes) aprovadas em cada versão.

## Varredura completa no caminho real (preview v0.0.36, 2026-09-30 ~00:49Z)

| # | Cenário | Resultado observado | Resultado |
|---|---|---|---|
| 1 | `F1-INVALID-001` (RED: sem unit_id, serviço ambíguo, STATUS_NOVO) | `BLOQUEADO` v1 — ausências: unit_id; identidade; service_ref não mapeado; status desconhecido | PASSOU |
| 2 | Correção do bloqueado | `VALIDO (v2, REC-EVT-0002)`, motivo na auditoria, v1 preservada | PASSOU |
| 3 | `F1-EDGE-001` (condicionais nulos) | `VALIDO` v1, `quantity/unitValue/grossValue: null`, elegível para a central | PASSOU |
| 4 | Correção de registro `VALIDO` | `VALIDO (v2)` criada sem recusa (correção vale para qualquer registro) | PASSOU |
| 5 | `F1-EDGE-002` (AGENDADO pós-data) | `VALIDO` com status `AGENDADO` preservado + "Pendência mantida — status AGENDADO não é produção enviável à central" | PASSOU |
| 6 | `F1-EDGE-003` (conteúdo clínico) | "Conteúdo clínico rejeitado (nome do paciente) — nada foi persistido"; nenhum registro criado | PASSOU |
| 7 | Regressão F1-T006 | `F1-VALID-001 registrado como VALIDO (v1, REC-EVT-0005)` | PASSOU |
| 8 | Privacidade na tela (F1-T003) | Payload clínico rejeitado, administrativo permitido — intacto | PASSOU |

## Debugs durante a task

- **Rodada 1 (v0.0.30/v0.0.32):** (a) unit_id vazio era negado por RLS antes da validação — corrigido: escopo compara só dimensões presentes; (b) `isEligibleForCentral` ignorava status de origem — corrigido: exige `ATENDIDO`.
- **Rodada 2 (v0.0.34):** recusa de correção sobre registro `VALIDO` era excesso sem base no aceite — removida; botão renomeado.
- **v0.0.36:** motivo da auditoria tornado neutro ("Correção do registro pelo champion").
- Detalhes: `06_notas/debug/debug-2026-09-29-f1t007-elegibilidade-central.md`.

## Veredito da verificação independente

| Critério | Evidência | Resultado |
|---|---|---|
| CA-1-007: ausência/ambiguidade/status desconhecido bloqueia e impede saída | Cenários 1 e 5 | PASSOU |
| CA-1-008: condicionais nulos nunca viram zero | Cenário 3 | PASSOU |
| CA-1-009: correção gera versão nova, relaciona a anterior, autor/motivo | Cenários 2 e 4 | PASSOU |
| CA-1-010: conteúdo clínico rejeitado | Cenário 6 | PASSOU |
| Regressão da F1-T006 | Cenário 7 | PASSOU |
| Pipeline de build/testes | v0.0.29–v0.0.36 verde | PASSOU |
| Teste humano | Champion aprovou ("parece ok") + varredura completa confirmou | PASSOU |

## Limitações

- Entrega de evidência para o contrato de saída é a **F1-T008** — aqui só o gate `isEligibleForCentral` existe.
- Nenhum dado real foi tocado; fixtures sintéticas.
