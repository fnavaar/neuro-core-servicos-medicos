# F1-T008 — Evidência: entrega do registro para o contrato

**Data:** 2026-09-29/30 · **SPEC:** SPEC-1-002 · **Skip:** v0.0.37–v0.0.40 · **Teste humano:** aprovado pelo champion ("funcionou", 2026-09-29 22:00)

## O que foi implementado

- `src/lib/operational-record.ts`:
  - `separateForContract(records)` — separação determinística (leitura, não altera registros): elegíveis = `VALIDO` **e** `ATENDIDO` (gate da F1-T007); excluídos com motivo `BLOQUEADO`, `STATUS_SEM_PRODUCAO` (AGENDADO/CANCELADO/FALTOU) ou `RASCUNHO`;
  - fixture `F1-BLOCKED-001` (registro bloqueado sintético exigido pelo TDD da SPEC para esta task).
- `src/pages/Index.tsx` — seção "F1-T008 · Evidência do registro para o contrato": botão cria os 3 registros de prova (F1-VALID-001, F1-BLOCKED-001, F1-EDGE-002) e executa a separação, exibindo as duas listas com payload versionado e motivo.

## Verificação automática (pipeline Skip v0.0.39/v0.0.40)

Setup, análise estática, build, integrações e testes — todos ok. (v0.0.37/v0.0.38 falharam por marcador de conflito residual de patch; corrigido regravando o módulo completo — aprendizado capturado.)

## Verificação no caminho real (preview v0.0.39, revalidada do zero após aprovação)

- "Separação executada: 1 elegível(is) para o contrato, 2 excluído(s) com motivo."
- **Segue para o contrato (1):** `F1-VALID-001` — payload versionado completo (`VALIDO`, `ATENDIDO`, `MAPEADO`, v1, auditoria).
- **Fora da saída (2):**
  - `F1-BLOCKED-001 — motivo: BLOQUEADO` (service_ref não mapeado no catálogo da clínica);
  - `F1-EDGE-002 — motivo: STATUS_SEM_PRODUCAO` (status AGENDADO mantém pendência — não é produção enviável).

## Veredito da verificação independente

| Critério (F1-T008) | Evidência | Resultado |
|---|---|---|
| Somente registro `VALIDO` é separado para SPEC-1-003 | F1-VALID-001 único na lista de elegíveis, com payload versionado | PASSOU |
| Registros `BLOQUEADO` permanecem fora da saída | F1-BLOCKED-001 excluído com motivo BLOQUEADO | PASSOU |
| Pendências permanecem fora (sem conversão de status) | F1-EDGE-002 excluído com motivo STATUS_SEM_PRODUCAO | PASSOU |
| Separação não altera registros (leitura) | Payloads idênticos antes/depois; nenhuma auditoria nova criada | PASSOU |
| Pipeline de build/testes | v0.0.39/v0.0.40 verde | PASSOU |
| Teste humano | Champion confirmou ("funcionou") | PASSOU |

## Limitações

- A geração do envelope `production-record.v1` (com `idempotency_key`) é a **F1-T010** (SPEC-1-003) — aqui só a separação/elegibilidade.
- Nenhum dado real foi tocado; fixtures sintéticas.
