# F1-T009 — Evidência: aprovação do contrato de saída

**Data:** 2026-09-29 · **SPEC:** SPEC-1-003 · **Recorte:** `F1-CONTRACT-APPROVAL` · **Decisão:** APROVADO

## Decisão do responsável financeiro

- **Decisão:** APROVADO — "Aprovar o contrato production-record.v1" (Fábio Schneider, 2026-09-29 22:10).
- **Aprovador:** Fábio Schneider — CEO, líder do projeto e responsável financeiro único (identidade confirmada na F1-T001).
- **Central consumidora:** este próprio sistema (nova aplicação de prontuário/produção no Skip) — identificado pelo champion em 2026-09-29 22:05, resolvendo a pré-condição da task.

## O que foi aprovado

1. **Identidade e idempotência:** `contract_version: production-record.v1` + `record_id` + `idempotency_key = sistema:record_id:versão`; repetição do mesmo registro/versão não duplica saída; correção (v2) gera chave nova (v1 e v2 são eventos distintos para a central).
2. **Origem preservada:** `source` com sistema, id de origem, versão e referência de evidência — proveniência rastreável.
3. **Dimensões mínimas:** entidade, unidade, profissional, serviço (id + rótulo do catálogo próprio, TUSS opcional conforme F1-T005), data, competência, status de origem, responsável.
4. **Valores:** `quantity`, `unit_value`, `gross_value` nulos quando indisponíveis — nunca estimados (CA-1-015).
5. **Privacidade:** conteúdo clínico/identificador pessoal rejeitado antes do envelope, sem persistir (CA-1-016).
6. **Versionamento do contrato:** mudança aprovada cria nova versão; nenhuma emenda silenciosa no mesmo `contract_version` (RN-1-008).
7. **Fase 1 sem integração:** nenhuma credencial, endpoint ou escrita externa — apenas fixture validada (CA-1-017).

## Pontos destacados na apresentação (sem objeção do aprovador)

- Valores nulos aceitos nesta fase (alternativa seria bloquear registro sem valor — não solicitada).
- Idempotência por versão: v1 e v2 como eventos distintos.
- Serviço como id + rótulo do catálogo próprio, coerente com a F1-T005.

## Pendências herdeiras

- Nenhuma objeção registrada. A geração do envelope (F1-T010) e os testes de rejeição/idempotência/privacidade (F1-T011) seguem o contrato aprovado; divergência futura exige emenda/versionamento, não ajuste silencioso.
