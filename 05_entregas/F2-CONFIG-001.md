# F2-CONFIG-001 — Configuração do ambiente, armazenamento durável e fonte do primeiro ciclo

**Data:** 2026-10-06 · **Task:** F2-T001 — Confirmar o ambiente e a fonte do primeiro ciclo
**SPEC:** SPEC-2-001 (`04_fase-atual/specs/spec-2-001-central-proveniencia-idempotencia.md`) · **Critério:** CA-2-001 · **Fase:** 2
**Dono:** Fábio Schneider (champion/responsável único) · **Execução:** Ethos (Eduard), autorizada pelo champion em 2026-10-06 10:55 ("Pode implementar")
**Versão do sistema na data:** Skip v0.0.44 (build 17f402a, pipeline verde)

## 1. O que este registro é

Registro de configuração exigido pela SPEC-2-001 antes de qualquer persistência (fluxo item 5 e CA-2-001): ambiente do sistema existente, armazenamento durável autorizado, escopo de acesso, estratégia de unicidade/transação/reversão e origem sintética do primeiro ciclo. Nenhuma collection foi criada, nenhum lote foi persistido e nenhum código de produto foi alterado nesta task — a persistência é objeto da F2-T002.

## 2. Ambiente confirmado (sistema existente, sem stack nova)

| Item | Valor confirmado | Como foi verificado |
|---|---|---|
| Sistema | "Página em Branco" — Skip Cloud, projeto id 51227 (submódulo `07_sistemas` → `fabioschneider9/p-gina-em-branco-njnhgvesg`) | skip_project_get/status em 2026-10-06 |
| Versão | v0.0.44 (hash 17f402a), current | skip_project_status |
| Preview | https://pagina-em-branco-ee3ec--preview.goskip.app | skip_project_get |
| Produção | https://pagina-em-branco-ee3ec.goskip.app (não publicada) | skip_project_get |
| Backend PocketBase | ativo e gerenciado pelo Skip Cloud — URL de consumo do frontend exposta via `VITE_POCKETBASE_URL` (pública no bundle; não é credencial) | skip_cloud_get_backend_url |
| Collections atuais | somente `users` (auth) — nenhuma collection de produção/idempotência criada | skip_cloud_get_collections |
| Variável `VITE_POCKETBASE_URL` | provisionada (chave confirmada; valor nunca copiado para arquivo) | skip_env_list |
| Frontend | React + Vite + TypeScript + shadcn (entrypoint `src/main.tsx`) | `.skip.config.json` |
| Runner | ETHOS (decisão F1-T001) | evidência F1-T001 |
| Repositório oficial do projeto | fnavaar/neuro-core-servicos-medicos (este documento vive aqui) | estrutura do repositório |
| Observação | árvore do Skip com mudança não commitada em `.skip.config.json` (pendingChanges); não bloqueia; não foi alterada nesta task | skip_project_status |

Módulos preservados da Fase 1 (referência do handoff F1-HANDOFF-001): `src/lib/operational-record.ts` (contrato production-record.v1), `src/lib/service-catalog.ts`, `src/lib/rls-fixture.ts`, `src/pages/Index.tsx`. Scaffolding já existente: `src/lib/pocketbase/` (`client.ts` com `VITE_POCKETBASE_URL`, `errors.ts`, `schema.json` só com `users`).

## 3. Armazenamento durável — decisão registrada

- **Decisão:** PocketBase nativo do Skip Cloud como armazenamento durável da central, conforme plano autorizado pelo champion ("Pode implementar", 2026-10-06 10:55); confirmação final no teste humano desta task.
- **Justificativa:** recurso nativo do ambiente existente (SPEC: "reuso do sistema existente e recursos nativos autorizados"); scaffolding já presente no projeto; backend já provisionado com `VITE_POCKETBASE_URL`; a SPEC veda localStorage/Map e proíbe escolher nova stack.
- **O que não muda:** contrato `production-record.v1`, chave `sistema:record_id:versão`, RLS da Fase 1, nulls preservados, retenção permanente (F1-T005), rejeições estruturadas.
- **Collections:** serão criadas apenas na F2-T002, usando exclusivamente os campos da SPEC-2-001 (lot_id, source_kind, source_ref/source_version, source_hash, entity_id, unit_id, professional_id, competency, evidence_ref, contract_version, idempotency_key, estado do lote, auditoria) — nenhum campo inventado; design final registrado na evidência da F2-T002.

## 4. Estratégia de unicidade, transação e reversão (registrada antes da persistência)

| Mecanismo | Estratégia registrada | Onde se aplica |
|---|---|---|
| Unicidade | Índice único no servidor sobre `idempotency_key` (`sistema:record_id:versão`) na estrutura de idempotência; mesma chave + mesmo conteúdo devolve a emissão persistida (RN-2-002); mesma chave + conteúdo divergente → `CONFLITO_IDEMPOTENCIA` sem sobrescrita (RN-2-003) | F2-T002 / F2-T003 |
| Transação | Escrita atômica do lote (transação/batch nativo do PocketBase): itens + evidências + marcação do lote; se a transação não cobrir o caso, estado do lote (staging → confirmado) com barreira de leitura — só lote confirmado é visível, nunca parcial (fluxo item 5) | F2-T002 / F2-T004 |
| Reversão | Append-only: marca versão/lote como revertido e restaura a referência ativa anterior; nunca apaga trilha, originais ou chaves emitidas (RN-2-005) | F2-T004 |
| RLS/acesso | Regras de acesso mínimas por escopo (entidade/unidade/profissional) herdando RN-1-001; serviço sem usuário não herda acesso global; payload clínico rejeitado sem persistir | F2-T002 em diante |

## 5. Escopo de acesso e segredos

- **Fábio Schneider:** administrador/champion, responsável único (F1-T001 e STATUS.md).
- **Permissões:** mínima leitura/escrita necessária, herdando RLS por entidade/unidade/profissional da Fase 1.
- **Segredos:** nenhum valor de variável ou credencial neste documento; `VITE_POCKETBASE_URL` é gerenciada pelo Skip Cloud e não é copiada para arquivos do repositório.
- **Risco registrado (arquivo não lido por política):** `.env` commitado na raiz do repositório do sistema `fabioschneider9/p-gina-em-branco-njnhgvesg` (78 bytes). Recomendação: remover do repositório e rotacionar o valor antes da F2-T002. Dono: Fábio/administrador. Não bloqueia esta task (documentação); vira pré-condição de higiene para a persistência.

## 6. Fonte do primeiro ciclo (sintética, autorizada)

| Item | Valor |
|---|---|
| Origem | `fixture://fase-2` — somente fixtures sintéticas até autorização de fonte real |
| Identidades | ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001, USER-RESP-001, USER-CREATE-001 |
| Contrato | `production-record.v1` (aprovado na F1-T009); envelope de referência no handoff F1-HANDOFF-001 §2 |
| Chave de idempotência | `sistema:record_id:versão` (padrão do contrato; record_id/versão definidos pela fixture sintética da F2-T002) |
| Dados reais / clínicos | Nenhum |

## 7. Relação origem → destino (ciclo de produção)

Produção (`fixture://fase-2`, envelope `production-record.v1`) → staging do lote (validação reusada da Fase 1: VALIDO + ATENDIDO, RN-1-005/006/007) → persistência no PocketBase (lote + itens + registro de idempotência, unicidade no servidor) → visibilidade apenas após confirmação total do lote → consulta com origem/versão e nulls preservados → reversão lógica append-only quando aplicável.

Demonstrativo (PDF fallback) terá envelope de ingestão próprio (SPEC-2-003, F2-T009 em diante) e nunca é reinterpretado como produção. API externa (SPEC-2-002, F2-T005 em diante) exige endpoint/autenticação autorizados — pendência não bloqueante nesta task.

## 8. Checklist CA-2-001

- [x] Configuração sem segredos registrada (este documento; varredura de padrões de segredo executada antes do commit — resultado: limpa).
- [x] Acesso confirmado: `VITE_POCKETBASE_URL` provisionada; backend PocketBase ativo (collections consultáveis).
- [x] Armazenagem aprovada: PocketBase nativo do Skip — decisão registrada conforme plano autorizado pelo champion em 2026-10-06.
- [x] Bloqueios/pendências explícitos: `.env` commitado (risco registrado com dono); unidades sem nome e catálogo definitivo seguem não bloqueantes; API externa pendente para a F2-T005.
- [x] Nenhuma stack nova, nenhum dado real, nenhuma collection criada, nenhum código de produto alterado.

## 9. Próximo passo

F2-T002 — Guardar um ciclo de teste na central (CA-2-002, prova F2-CENTRAL-001), condicionada ao aceite humano desta configuração e à autorização task a task.