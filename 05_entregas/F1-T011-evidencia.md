# F1-T011 — Evidência: rejeição, idempotência e privacidade

**Data:** 2026-10-01 · **SPEC:** SPEC-1-003 · **Skip:** v0.0.43/v0.0.44 · **Teste humano:** aprovado pelo champion ("deu certo", 2026-10-01 ~10:50)

## O que foi implementado

- `src/lib/operational-record.ts`:
  - `issueEnvelope(record)` — camada do contrato: valida privacidade (defesa em profundidade), identidade oficial e elegibilidade ANTES de gerar; rejeições estruturadas com código (`NAO_ELEGIVEL`, `IDENTIDADE_AMBIGUA`, `CLINICAL_CONTENT`), detalhe e recordId — nunca saída parcial (RN-1-006/RN-1-007);
  - registro de idempotência em memória: chave já emitida devolve o MESMO envelope como `DUPLICADO`, sem criar nova saída (RN-1-005/CA-1-013); `resetIssuedEnvelopes`/`issuedEnvelopeCount` para demonstração;
  - fixture `F1-NULL-VALUE-001`; `F1-BLOCKED-001` reutilizada da F1-T008.
- `src/pages/Index.tsx` — seção "F1-T011" com botões por cenário, contador de envelopes emitidos e tabela de resultados.

## Verificação automática (pipeline Skip v0.0.43/v0.0.44)

Setup, análise estática, build, integrações e testes — todos ok.

## Verificação no caminho real (v0.0.43, aprovação) + varredura da revisão consolidada (v0.0.44, 2026-10-01 11:19)

| Cenário | Resultado observado | Resultado |
|---|---|---|
| `F1-BLOCKED-001` (bloqueado) | `REJEITADO (NAO_ELEGIVEL)` — registro BLOQUEADO (service_ref não mapeado); contador 0 | PASSOU |
| `F1-DUPLICATE-001` 1ª emissão | `EMITIDO` — chave `nova-aplicacao-prontuario-producao:F1-VALID-001:1`; contador 1 | PASSOU |
| `F1-DUPLICATE-001` 2ª emissão | `DUPLICADO (sem nova saída)` — mesma chave devolvida; contador permaneceu **1** | PASSOU |
| `F1-NULL-VALUE-001` (nulos) | `EMITIDO` — envelope com valores `null` preservados (CA-1-015) | PASSOU |
| `F1-PRIVACY-001` (clínico) | `REJEITADO (CLINICAL_CONTENT)` — nada persistido; contador inalterado (CA-1-016) | PASSOU |

## Observação da revisão consolidada (identidade ambígua)

A rejeição `IDENTIDADE_AMBIGUA` existe na camada do contrato (defesa em profundidade), mas não é diretamente exercitável na demonstração porque a criação (F1-T006/T007) já bloqueia identidade ambígua antes de o registro existir — provado end-to-end: `F1-INVALID-001` (identidade não confirmada) → `BLOQUEADO` → `REJEITADO (NAO_ELEGIVEL)`, nunca um envelope. O invariante "identidade ambígua não gera saída" vale para o sistema como um todo.

## Veredito da verificação independente

| Critério (F1-T011) | Evidência | Resultado |
|---|---|---|
| Bloqueado não gera saída | Cenário 1 | PASSOU |
| Repetição não duplica (CA-1-013) | Cenários 2–3: contador fixo em 1 | PASSOU |
| Identidade ambígua não gera saída | Bloqueio na criação (end-to-end) + checagem na camada do contrato | PASSOU |
| Nulos preservados (CA-1-015) | Cenário 4 | PASSOU |
| Clínico rejeitado sem persistir (CA-1-016) | Cenário 5 | PASSOU |
| Pipeline de build/testes | v0.0.43/v0.0.44 verde | PASSOU |
| Teste humano | Champion confirmou ("deu certo") | PASSOU |

## Limitações

- Registro de idempotência em memória (Fase 1, sem persistência externa — CA-1-017); persistência real na Fase 2.
- Nenhum dado real foi tocado; fixtures sintéticas.