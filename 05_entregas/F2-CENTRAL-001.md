# F2-CENTRAL-001 — Ciclo persistido na central (armazenamento durável)

**Data:** 2026-10-15 · **Task:** F2-T002 — Guardar um ciclo de teste na central
**SPEC:** SPEC-2-001 (`04_fase-atual/specs/spec-2-001-central-proveniencia-idempotencia.md`) · **Critério:** CA-2-002 · **Fase:** 2
**Dono:** Fábio Schneider (champion) · **Execução:** Ethos (Eduard), autorizada pelo champion em 2026-10-15 ("Pode implementar a F2-T002")
**Versões do sistema:** Skip v0.0.45 → v0.0.49 (pipeline verde em todas; v0.0.49 = hash 4288def)

## 1. O que foi implementado

| Componente | Arquivo | Papel |
|---|---|---|
| Migração 0001 | `pocketbase/migrations/0001_create_f2_central_collections.js` | Collections `f2_lots` (estado STAGING→CONFIRMADO→REVERTIDO, campos da SPEC-2-001) e `f2_idempotency` (chave única + envelope JSON), índice único no servidor sobre `idempotency_key` |
| Migração 0002 | `pocketbase/migrations/0002_fix_f2_central_rules.js` | Regras de acesso: update do lote só na transição única STAGING→CONFIRMADO com todos os demais campos travados (`:changed = false`); emissões imutáveis (update/delete só superuser); delete de lote só superuser (reversão é lógica, RN-2-005) |
| Migração 0003 | `pocketbase/migrations/0003_seed_demo_user.js` | Usuário sintético de demonstração (fixture, sem dado real) para autenticação exigida pelas regras |
| Módulo de persistência | `src/lib/central-store.ts` | `importLot` (staging → emissões → confirmação; DUPLICADO/CONFLITO_IDEMPOTENCIA/LOTE_INCOMPLETO), `listConfirmedLots` (barreira de leitura), `listEmissionsByLot`, autenticação idempotente da sessão |
| Tela | `src/pages/Index.tsx` (seção F2-T002) | Importar ciclo, reimportar mesmo lote, atualizar consulta; tabelas de lotes confirmados e emissões com nulos preservados |

## 2. Prova GREEN (executada no preview v0.0.49, 2026-10-15)

| Cenário | Ação executada | Resultado observado |
|---|---|---|
| Positivo | Botão "Importar ciclo (fixture://fase-2)" | Lote `F2-LOT-2026-10-001` CONFIRMADO com 1 emissão; origem `PRODUCTION_API · fixture://fase-2 · v1`; competência 2026-10 |
| Consulta após reinício | Página recarregada + "Atualizar consulta" | Lote e emissão reaparecem do servidor: chave `nova-aplicacao-prontuario-producao:F2-VALID-001:1`, `qty=1 · unit=null · gross=null` (nulos preservados, CA-1-015) |
| Repetição | "Reimportar mesmo lote" (mesma chave, mesmo conteúdo reconstruído da fixture determinística) | `DUPLICADO — mesma chave, mesmo conteúdo; emissão existente devolvida sem nova saída (RN-2-002)` |
| Contagem no servidor | Consulta direta à API do PocketBase (autenticada) | `f2_lots`: totalItems 1 · `f2_idempotency`: totalItems 1 — nenhuma duplicata |
| Envelope íntegro | Consulta direta à API | Envelope `production-record.v1` completo persistido no campo json: source (sistema/id/versão/evidência), dimensões, `service_ref {id: SRV-002, label: ECG}`, audit, `unit_value: null`, `gross_value: null` |

## 3. Verificação estrutural (via skip_cloud_get_collection_details)

- `f2_lots`: listRule/viewRule exigem autenticação e só expõem `CONFIRMADO`/`REVERTIDO` (barreira de leitura — staging nunca visível, RN-2-004); createRule autenticada; updateRule = transição única STAGING→CONFIRMADO com 14 campos travados; deleteRule = null (superuser); índices `idx_f2_lots_lot_id` (único) e `idx_f2_lots_competency`.
- `f2_idempotency`: list/view/create autenticados; update/delete = null (imutável — nunca limpar a idempotência); índice único `idx_f2_idem_key` sobre `idempotency_key` (unicidade no servidor, RN-2-002/RN-2-003).
- Migrações: 0001/0002/0003 `applied` (status consultado no Skip Cloud).

## 4. Correções durante a execução (2 rodadas de debug, documentadas)

1. **v0.0.46:** as regras do 0001 usavam `updateRule: ''`/`deleteRule: ''`, que em PocketBase significa PÚBLICO (não negado) — lote e emissão poderiam ser alterados/apagados por anônimo. Corrigido na migração 0002 (transição única + imutabilidade + delete superuser) e autenticação da sessão adicionada ao store.
2. **v0.0.47–v0.0.49:** a consulta de emissões e a comparação de idempotência usavam o `id` do registro PocketBase onde o filtro espera o campo de negócio `lot_id` (string), e a comparação de conteúdo usava `JSON.stringify` sensível à ordem de chaves após o round-trip do json. Corrigido: filtro por `lot_id` e comparação canônica (chaves ordenadas recursivamente). O efeito visível era `CONFLITO_IDEMPOTENCIA` falso na reimportação; após a correção, `DUPLICADO` correto.

## 5. Checklist CA-2-002

- [x] Ciclo persistido com origem, versão, evidência e contrato (`production-record.v1`) — envelope íntegro no campo json (prova via API).
- [x] Consultável após reinício com as mesmas chaves — recarregado + atualizado: lote e emissão reaparecem.
- [x] Valores nulos preservados (nunca zero) — `qty=1 · unit=null · gross=null` na tela e no envelope da API.
- [x] Repetição não duplica — reimportação devolve DUPLICADO; contagem no servidor: 1 emissão.
- [x] Nenhum parcial visível — barreira de leitura nas regras (só CONFIRMADO/REVERTIDO); lote incompleto rejeitado antes de confirmar (LOTE_INCOMPLETO).
- [x] Sem segredo no repositório: credencial de demonstração é fixture sintética criada por migração seed (usuário de teste, sem acesso a dado real); `VITE_POCKETBASE_URL` permanece gerenciada pelo Skip Cloud.
- [x] Nenhuma fonte real, nenhum dado clínico, nenhum campo fora da SPEC-2-001.

## 6. Teste humano do champion

1. Abrir o preview (v0.0.49): https://pagina-em-branco-ee3ec--preview.goskip.app
2. Na seção "F2-T002 · Ciclo persistido na central", clicar **Atualizar consulta** → o lote `F2-LOT-2026-10-001` (CONFIRMADO) e a emissão com nulos preservados devem aparecer.
3. Recarregar a página (F5) e clicar **Atualizar consulta** de novo → o ciclo continua consultável (persistência durável).
4. Clicar **Reimportar mesmo lote** → resultado esperado: `DUPLICADO — mesma chave, mesmo conteúdo... (RN-2-002)`.
5. Falha reconhecível: erro vermelho na seção, lote ausente após recarregar, ou contagem de emissões > 1.

## 7. Próximo passo

F2-T003 — Impedir duplicação após reinício (CA-2-003, prova F2-IDEMP-001): concorrência simultânea e conflito de chave divergente, condicionada ao aceite humano desta task e à autorização task a task.