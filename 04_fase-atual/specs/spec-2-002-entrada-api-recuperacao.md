# SPEC-2-002 — Entrada pela API e recuperação de falhas

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono:** Fábio Schneider, champion/responsável único; execução assistida pelo Ethos  
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um ciclo de teste chega ao módulo central do próprio sistema pela API autorizada, com relatório de entrada e recuperação sem duplicação. Se inviável, a decisão e o caminho alternativo ficam demonstráveis.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Endpoint, autenticação, escopos, esquema, paginação e orçamento de timeout/retry devem existir e ser autorizados em F2-API-CONFIG-001. API inviável é decisão registrada que ativa a rota alternativa; não inventar endpoint ou conceder acesso.

## Resultado observável

Um ciclo de teste chega ao módulo central do próprio sistema pela API autorizada, com relatório de entrada e recuperação sem duplicação. Se inviável, a decisão e o caminho alternativo ficam demonstráveis.

## Limites e dependências

- Inclui: Adaptador do contrato existente; configuração da API; paginação completa; validação, relatório de erro e retry seguro; indicação explícita de fallback disponível.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

Contrato production-record.v1 sem alteração: origem/versão, dimensões, service_ref, competency, quantidade e valores, estado, responsável, auditoria e idempotency_key. Metadados do transporte: lot_id, source_ref, request_ref sem segredo, paginação concluída, status e erros sanitizados.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-006 | Página ausente, erro ou resposta incompleta | Não confirmar lote; indicar estágio de falha | Escopo fase 2 |
| RN-2-007 | 401/403 | Parar e registrar acesso negado sem ampliar escopo | RQ-012 |
| RN-2-008 | Fonte vazia | Mostrar fonte vazia, zero importado; não confundir com sucesso de ciclo | Qualidade de origem |
| RN-2-009 | API indisponível | Oferecer fallback autorizado com origem distinta | D-003/escopo fase 2 |

## Fluxo e regras

1. Confirmar produtor e central lógica no próprio sistema conforme F1-T009; identificar API existente e registrar contrato técnico sem segredos.
2. Registrar endpoint/método, esquema dos itens, cursor/paginação, fonte de verdade, escopos, referência segura da autenticação e limites documentados pelo provedor. Timeout/retry fica explicitamente fechado nesta configuração, nunca arbitrado pelo executor.
3. Se API inviável, registrar motivo, evidência e decisão; não criar integração fictícia. Prosseguir somente com fallback autorizado da SPEC-2-003.
4. Solicitar um ciclo sintético e percorrer todas as páginas no staging. Guardar identificação da resposta e da origem sem armazenar credencial ou payload clínico.
5. Validar os itens contra production-record.v1. Mesmo quando origem e central compartilham sistema, a fronteira de validação e autorização permanece obrigatória.
6. Confirmar via SPEC-2-001 apenas depois de resposta/paginação completa. Relatório registra recebidos, aceitos, duplicados e rejeitados, origem e versão; null não vira zero.
7. No timeout/429/5xx, registrar falha e próximo passo. Repetir somente com idempotência durável e dentro da política aprovada. 401/403 bloqueiam e exigem correção de permissão, sem retry de autenticação infinito.
8. Indisponibilidade mostra alternativa; não varre pasta nem troca a fonte silenciosamente. Depois da recuperação, reprocessar o ciclo mantendo as chaves.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-API-001: um ciclo sintético com duas páginas | Todos os itens visíveis com origem e versão após confirmação |
| Vazio | Fonte sem itens | Relatório fonte vazia; nenhum dado fabricado |
| Acesso | 401/403 ou escopo não autorizado | Sem lote confirmado; orientar responsável |
| Falha | F2-API-ERROR-001: timeout, 429, 5xx, página faltante | Lote em falha; retry controlado; nenhuma duplicação |
| Contrato | Status inválido, payload clínico, identidade ambígua | Rejeitar e registrar categoria, nunca saída parcial |
| Alternativa | API declarada inviável | Decisão registrada, referência à rota PDF autorizada |

## Instruções de execução para o Ethos

1. Ler handoff F1-T012 §§2–10, decisões F1-T005/F1-T009 e esta SPEC.
2. Executar uma task por vez, observando as dependências. Preparação registra decisão/acesso; não conta como integração.
3. Alterar apenas o recorte da task. Preservar contrato, RLS, auditoria, null e originais.
4. Registrar configuração autorizada antes da implementação; se mudar regra de negócio/contrato, devolver ao consultor para emenda. Não inventar arquitetura ou campo.
5. Parar sem acesso, documento, regra, layout, permissão ou confirmação. Não inserir segredos no GitHub.
6. Deixar o sistema anterior funcionando e nenhum lote parcial visível; anexar prova em 05_entregas e solicitar teste humano antes de marcar task.

## Checklist de execução

- [ ] Pré-condições e ambiente conferidos; bloqueios explícitos.
- [ ] Caminho principal demonstrado com fixture sintética.
- [ ] Falhas, permissão, privacidade e repetição demonstradas.
- [ ] Original/versão anterior preservados; rollback demonstrável.
- [ ] Provas e aceite humano registrados, sem segredos ou dados clínicos.

## Critérios de aceite

- [ ] **CA-2-005:** Viabilidade e configuração autorizada da API registradas, ou inviabilidade com motivo. Prova: F2-API-CONFIG-001.
- [ ] **CA-2-006:** Ciclo sintético completo importado pela API com origem/versão por item. Prova: F2-API-001.
- [ ] **CA-2-007:** Falhas e paginação incompleta não confirmam lote; retry não duplica; acesso indevido é negado. Prova: F2-API-ERROR-001.
- [ ] **CA-2-008:** Demonstração e relatório da rota API, ou não aplicabilidade explícita vinculada ao fallback aprovado. Prova: F2-API-HANDOFF-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Usar transporte simulado ou sandbox autorizado com duas páginas e falha na segunda; evidenciar que lote parcial precisa permanecer invisível. | Captura/registro da falha esperada antes da implementação |
| GREEN | Executar ciclo de duas páginas até confirmação e repetir; o total não aumenta. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Fonte vazia, timeout, 401/403, 429, 5xx, payload inválido, clínico e repetição após reinício. Rota declarada inviável permanece como não aplicável, nunca como teste API aprovado. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-API-CONFIG-001, F2-API-001, F2-API-ERROR-001, F2-API-HANDOFF-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Acompanhar o ciclo da origem até a central, ver origem e competência; repetir, interromper a segunda página e recuperar com total estável. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Registrar o caminho autorizado de entrada | CA-2-005 | F2-API-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Trazer um ciclo pela API | CA-2-006 | F2-API-001 | Registrar o caminho autorizado de entrada; Guardar um ciclo de teste na central |
| Tratar indisponibilidade sem perder o ciclo | CA-2-007 | F2-API-ERROR-001 | Trazer um ciclo pela API |
| Demonstrar o resultado da entrada pela API | CA-2-008 | F2-API-HANDOFF-001 | Tratar indisponibilidade sem perder o ciclo |

## Emendas

Nenhuma. Qualquer mudança de contrato ou autorização material deve ser registrada antes de executar.
