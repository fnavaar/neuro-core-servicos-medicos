# SPEC-2-001 — Central financeira, proveniência e idempotência durável

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono:** Fábio Schneider, champion/responsável único; execução assistida pelo Ethos  
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um ciclo sintético permanece consultável na central após reinício; reimportação e concorrência não duplicam emissões; cada item aponta para fonte e versão.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Antes da persistência: ambiente e armazenamento durável autorizados, permissões mínimas e mecanismo de unicidade/transação registrados em F2-CONFIG-001. Se indisponíveis, executar apenas o registro de pré-condições; não substituir por localStorage ou Map.

## Resultado observável

Um ciclo sintético permanece consultável na central após reinício; reimportação e concorrência não duplicam emissões; cada item aponta para fonte e versão.

## Limites e dependências

- Inclui: Modelo mínimo de produção, referência financeira, demonstrativo, regra/mapeamento e evidência; persistência durável de lotes/chaves; normalização aprovada; histórico e reversão lógica.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

lot_id; source_kind (PRODUCTION_API, PDF_FALLBACK); source_ref e source_version; source_hash quando arquivo; entity_id, unit_id, professional_id, competency; evidence_ref; contract_version; idempotency_key; estado do lote; ator, data e evento de auditoria. Preservar quantity/unit_value/gross_value como null se ausentes. Referências de regras e demonstrativos armazenam versão e origem, sem calcular repasses nesta fase.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-001 | production-record.v1 recebido | Preservar envelope, chave e valores null; campo novo exige versão/aprovação | F1-T009/F1-T012 |
| RN-2-002 | Mesma chave e mesmo conteúdo | Uma única emissão persistida; devolver referência existente | RQ-003/F1-T011 |
| RN-2-003 | Mesma chave e conteúdo diferente | CONFLITO_IDEMPOTENCIA, zero sobrescrita | Integridade do contrato aprovado |
| RN-2-004 | Lote incompleto ou identidade ambígua | Lote não confirmado; registrar exceção sem conteúdo clínico | Escopo fase 2 |
| RN-2-005 | Reversão de lote confirmado | Histórico append-only; restaurar referência ativa anterior | Escopo fase 2/rollback |

## Fluxo e regras

1. Ler o contrato production-record.v1 e as decisões da F1-T005/F1-T009; confirmar o ambiente já utilizado, sem criar stack nova.
2. Criar área de staging do ciclo com lote, origem e versão; validar todos os itens antes da confirmação.
3. Reusar validação, RLS e elegibilidade VALIDO + ATENDIDO do sistema existente. Origem financeira/demonstrativo possui envelope de ingestão próprio e não é reinterpretada como produção.
4. Persistir lote, itens e registro de idempotência usando armazenamento autorizado e unicidade no servidor. Uma chave idêntica devolve o resultado persistido; conteúdo divergente sob mesma chave rejeita o lote.
5. Tornar lote visível apenas depois de todos os itens e suas evidências persistirem. Usar transação nativa; se indisponível, estado de lote e barreira de leitura impedem exposição parcial. Registrar a estratégia no F2-CONFIG-001 antes de implementar.
6. Reiniciar e consultar o mesmo ciclo; reimportar e enviar duas chamadas simultâneas. A versão corrigida do registro usa chave nova e mantém a anterior.
7. Reversão marca a versão/lote como revertido e restaura a referência ativa anterior. Mantém trilha, originais e chaves: nunca limpa a idempotência para repetir uma emissão.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-CENTRAL-001: envelope válido sintético | Lote confirmado e consultável após reinício |
| Repetição/concorrência | F2-IDEMP-001: mesma chave, incluindo duas chamadas simultâneas | Uma emissão, mesma referência; nenhum aumento de total |
| Conflito | Mesma chave, payload diferente | Rejeição estruturada; original preservado |
| Falha/rollback | F2-ROLLBACK-001: interrupção no segundo item | Nenhum parcial visível; retry seguro e reversão auditada |
| Permissão/privacidade | Usuário sem escopo ou payload clínico | Negação/rejeição sem persistência ou vazamento |

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

- [ ] **CA-2-001:** Configuração sem segredos registrada, acesso e armazenagem aprovados ou bloqueio formalizado. Prova: F2-CONFIG-001.
- [ ] **CA-2-002:** Ciclo persistido é recuperado após reinício com origem e versão completas e null preservado. Prova: F2-CENTRAL-001.
- [ ] **CA-2-003:** Repetição após reinício e concorrência deixam uma emissão; conflito não sobrescreve. Prova: F2-IDEMP-001.
- [ ] **CA-2-004:** Falha, permissão e rollback não expõem lote parcial nem apagam trilha. Prova: F2-ROLLBACK-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Reiniciar a demonstração volátil da fase 1 e evidenciar a falta de idempotência durável; configurar teste que detecta duplicação ou perda da referência persistida. | Captura/registro da falha esperada antes da implementação |
| GREEN | Persistir um ciclo sintético completo, reiniciar, recuperar e repetir a mesma chave; comparar contagem e conteúdo. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Executar concorrência, mesma chave divergente, falha intermediária, reversão, permissão e privacidade; repetir cenários da fase 1 sem regressão. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-CONFIG-001, F2-CENTRAL-001, F2-IDEMP-001, F2-ROLLBACK-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Abrir ciclo, consultar origem/versão, reiniciar, reimportar e conferir que o total é igual; observar reversão com histórico preservado. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Confirmar o ambiente e a fonte do primeiro ciclo | CA-2-001 | F2-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Guardar um ciclo de teste na central | CA-2-002 | F2-CENTRAL-001 | Confirmar o ambiente e a fonte do primeiro ciclo |
| Impedir duplicação após reinício | CA-2-003 | F2-IDEMP-001 | Guardar um ciclo de teste na central |
| Recuperar o ciclo após uma falha | CA-2-004 | F2-ROLLBACK-001 | Impedir duplicação após reinício |

## Emendas

Nenhuma. Qualquer mudança de contrato ou autorização material deve ser registrada antes de executar.
