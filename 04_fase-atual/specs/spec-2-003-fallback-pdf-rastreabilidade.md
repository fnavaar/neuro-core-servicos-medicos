# SPEC-2-003 — Fallback de PDF, exceções e handoff do ciclo

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono:** Fábio Schneider, champion/responsável único; execução assistida pelo Ethos  
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-006, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um demonstrativo de teste é importado com origem, página/linha e hash preservados; arquivo repetido não duplica e arquivo ilegível fica em exceção. O ciclo possui pacote de evidências para a próxima fase.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Permissão de pasta em leitura, amostra e layout aprovados, mapeamento e política de tratamento administrativo precisam estar registrados em F2-PDF-CONFIG-001. OCR/extração incerta exige revisão humana, sem aprovação automática. API viável não exige ativar fallback real; exige demonstrar a alternativa com fixture autorizada.

## Resultado observável

Um demonstrativo de teste é importado com origem, página/linha e hash preservados; arquivo repetido não duplica e arquivo ilegível fica em exceção. O ciclo possui pacote de evidências para a próxima fase.

## Limites e dependências

- Inclui: Leitura restrita a pasta/fonte autorizada; prévia do demonstrativo; layout versionado, normalização e relatório; fallback visível e recuperação; handoff final do ciclo.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

Demonstrativo: source_ref, source_hash, document_version, layout_version, mapping_version, page_or_line, entity_id, unit_id, professional_id, competency, service_ref, quantity/unit_value/gross_value, responsible_user_id, evidence_ref e estado. Chave durável do item de documento combina hash/layout/mapeamento/localização; relação com production-record.v1 exige mapeamento aprovado, sem adulterar sua chave.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-010 | Layout desconhecido ou PDF ilegível | Exceção; não extrair por suposição nem confirmar parcial | Escopo fase 2 |
| RN-2-011 | Arquivo repetido, mesmos hash e versão | Reutilizar importação registrada; total inalterado | RQ-003 |
| RN-2-012 | Identidade não mapeada ou catálogo não vigente | Bloquear uso real; somente fixture sintética isolada | F1-T001/F1-T005/F1-T012 |
| RN-2-013 | Fonte original | Leitura somente; hash antes/depois iguais | Escopo fase 2 |
| RN-2-014 | Caminho de fase encerrado | Pacote registra usada/não aplicável/bloqueada por rota, sem afirmar integração ausente | F1-HANDOFF-001/escopo fase 2 |

## Fluxo e regras

1. Registrar fonte autorizada em leitura, amostra sintética/anonimizada aprovada e versão do layout. Não buscar em outras pastas nem ler PDFs clínicos.
2. Reusar recurso de extração já disponível no ambiente autorizado. Se parser/OCR não existir ou precisar de serviço externo, parar e registrar dependência; não instalar ou enviar documento a serviço novo por inferência.
3. Preservar original e hash; selecionar arquivos do escopo e detectar repetição por hash + versão do layout/mapeamento. Mesmo nome com bytes diferentes é versão distinta que demanda validação.
4. Extrair apenas campos administrativos reconhecidos. Cada item mantém página/linha ou localização equivalente, fonte, hash e versões de layout/mapeamento. Desconhecido, ambíguo ou ilegível vira exceção.
5. Aplicar catálogo por clínica e identidades aprovadas. Três unidades sem nome e catálogo desatualizado não autorizam inferência de identidades ou preços reais. Valores ausentes permanecem null.
6. Mostrar prévia por lote com campos, origem e pendências. Confirmar somente lote completamente validado pela central da SPEC-2-001. Não promover resultado incerto de OCR automaticamente.
7. Reimportar o mesmo documento, interromper um lote e recuperar; contagem confirmada permanece correta. Reversão preserva original, evidência e histórico.
8. Consolidar pacote final com rota usada, decisão API/fallback, contagens, chaves, exceções, permissões, provas de falha e rollback. A central financeira é o mesmo sistema; reconciliação e cálculo pertencem à fase seguinte.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-PDF-001: PDF sintético layout aprovado | Prévia íntegra e lote confirmado com hash e página/linha |
| Vazio/acesso | Pasta vazia ou sem permissão | Vazio/negado explícito; nenhuma varredura fora do escopo |
| Qualidade | F2-PDF-ERROR-001: ilegível, layout novo, identidade ambígua | Exceção visível, sem dado inventado ou confirmação parcial |
| Repetição | Mesmo arquivo duas vezes e após reinício | Uma importação, total inalterado |
| Mudança | Mesmo nome, hash diferente | Versão nova em staging, revisão explícita e histórico preservado |
| Final | F2-HANDOFF-001: rota viável comprovada | Pacote com proveniência, versão, idempotência e recuperação |

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

- [ ] **CA-2-009:** Pasta/fonte, amostra, layout e mapeamentos autorizados registrados ou bloqueio formalizado. Prova: F2-PDF-CONFIG-001.
- [ ] **CA-2-010:** Amostra administrativa importada com original intacto, hash, origem e localização por item. Prova: F2-PDF-001.
- [ ] **CA-2-011:** Ilegíveis/ambíguos não confirmam lote; repetição/reinício não duplicam; versão nova preserva histórico. Prova: F2-PDF-ERROR-001.
- [ ] **CA-2-012:** Pacote comprova ciclo por API ou fallback documentado, proveniência, idempotência, falha recuperável e aceite. Prova: F2-HANDOFF-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Usar PDF sintético de layout aprovado e repetir importação; prova deve detectar perda de localização/proveniência e duplicação antes da solução. | Captura/registro da falha esperada antes da implementação |
| GREEN | Extrair amostra, mostrar prévia validada e confirmar ciclo; comparar hash do original e contagem na repetição. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Pasta vazia/negada, PDF ilegível, layout desconhecido, identidade ambígua, nulos, conteúdo clínico e arquivo repetido após reinício; provar rollback da central. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-PDF-CONFIG-001, F2-PDF-001, F2-PDF-ERROR-001, F2-HANDOFF-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Abrir prévia, localizar um valor no PDF de origem, confirmar ciclo e reimportar; conferir total estável e original intacto. Revisar pacote final e registrar aceite explícito. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Autorizar a pasta e o layout do fallback | CA-2-009 | F2-PDF-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Importar um demonstrativo pelo fallback | CA-2-010 | F2-PDF-001 | Autorizar a pasta e o layout do fallback; Guardar um ciclo de teste na central |
| Separar PDFs ilegíveis e entradas repetidas | CA-2-011 | F2-PDF-ERROR-001 | Importar um demonstrativo pelo fallback |
| Fechar as evidências do ciclo importado | CA-2-012 | F2-HANDOFF-001 | Recuperar o ciclo após uma falha; Demonstrar o resultado da entrada pela API; Separar PDFs ilegíveis e entradas repetidas |

## Emendas

Nenhuma. Qualquer mudança de contrato ou autorização material deve ser registrada antes de executar.
