# Fase 2 — Tarefas

<!-- fase-format:2 -->

**Objetivo:** trazer um ciclo de teste à central financeira do próprio sistema por API preferencial ou fallback de PDF documentado, com proveniência, idempotência durável e recuperação.

**Abertura:** 2026-10-02. Uma task por vez. As tasks de preparação podem ser iniciadas; as demais aguardam suas dependências e acesso autorizado. Somente fixtures sintéticas até autorização de fonte real. UUIDs novos aguardam o portal; responsável e prazo dos cards não foram inventados.

## Cards

- [ ] Confirmar o ambiente e a fonte do primeiro ciclo #projeto
  > SPEC-2-001 — CA-2-001; prova F2-CONFIG-001. Pré-condições: Handoff F1-HANDOFF-001. Registrar ambiente do sistema existente, armazenamento durável autorizado, escopo de acesso e origem sintética do ciclo. Anexar configuração sem segredos, fonte oficial, responsáveis e relação origem → destino. Não escolher nova stack nem criar dados reais. A ausência de acesso fica registrada e impede a task de persistência.

- [ ] Guardar um ciclo de teste na central #projeto
  > SPEC-2-001 — CA-2-002; prova F2-CENTRAL-001. Pré-condições: Confirmar o ambiente e a fonte do primeiro ciclo. Reusar production-record.v1 e persistir um lote sintético validado na central do próprio sistema, com origem, versão, evidência e valores nulos preservados. Registrar configuração autorizada antes de alterar armazenamento. Após reiniciar, o ciclo continua disponível com as mesmas chaves.

- [ ] Impedir duplicação após reinício #projeto
  > SPEC-2-001 — CA-2-003; prova F2-IDEMP-001. Pré-condições: Guardar um ciclo de teste na central. Persistir a chave sistema:record_id:versão com unicidade no servidor. Reimportar o lote, reiniciar e repetir; duas requisições concorrentes resultam em uma única emissão. Mesma chave com conteúdo divergente gera conflito, sem sobrescrever o original.

- [ ] Recuperar o ciclo após uma falha #projeto
  > SPEC-2-001 — CA-2-004; prova F2-ROLLBACK-001. Pré-condições: Impedir duplicação após reinício. Simular falha no meio do lote, entrada sem permissão, identidade não mapeada e conteúdo clínico. Nenhum registro parcial é exposto. Repetir após correção e reverter a versão ativa sem apagar originais, histórico ou chaves emitidas.

- [ ] Registrar o caminho autorizado de entrada #projeto
  > SPEC-2-002 — CA-2-005; prova F2-API-CONFIG-001. Pré-condições: Handoff F1-HANDOFF-001. Documentar se a API da aplicação existente é viável: endpoint, método, esquema, escopos mínimos, paginação, timeout, limite e referência segura de autenticação. Sem segredo em arquivo. Central é o próprio sistema. Sem API autorizada, registrar inviabilidade e encaminhar para fallback; não inventar endpoint.

- [ ] Trazer um ciclo pela API #projeto
  > SPEC-2-002 — CA-2-006; prova F2-API-001. Pré-condições: Registrar o caminho autorizado de entrada; Guardar um ciclo de teste na central. Somente com API autorizada e configuração registrada: adaptar o payload ao contrato aprovado e importar um ciclo sintético até o fim da paginação, preservando proveniência e versão. A evidência deve listar quantos registros foram recebidos, aceitos, duplicados ou rejeitados e quais estão visíveis na central.

- [ ] Tratar indisponibilidade sem perder o ciclo #projeto
  > SPEC-2-002 — CA-2-007; prova F2-API-ERROR-001. Pré-condições: Trazer um ciclo pela API. Exercitar timeout, 401/403, 429, 5xx, página ausente, resposta inválida e fonte vazia. Não confirmar lote incompleto; reprocessamento usa a mesma chave. Indisponibilidade habilita revisão do fallback autorizado, sem executar leitura de pastas automaticamente.

- [ ] Demonstrar o resultado da entrada pela API #projeto
  > SPEC-2-002 — CA-2-008; prova F2-API-HANDOFF-001. Pré-condições: Tratar indisponibilidade sem perder o ciclo. Demonstrar origem, competência, versões, nulos, repetição com total estável e recuperação. Se API foi declarada inviável, esta task registra não aplicabilidade com motivo e evidência da decisão e referencia o teste equivalente da SPEC-2-003; não declarar API implementada.

- [ ] Autorizar a pasta e o layout do fallback #projeto
  > SPEC-2-003 — CA-2-009; prova F2-PDF-CONFIG-001. Pré-condições: Handoff F1-HANDOFF-001. Registrar fonte/pasta autorizada em leitura, responsável, amostra sintética ou anonimizada aprovada, hash, versão do layout, campos e mapeamentos. Restringir ao escopo autorizado. PDF clínico não entra. OCR só após regra de revisão aprovada; sem layout reconhecido, bloquear a importação.

- [ ] Importar um demonstrativo pelo fallback #projeto
  > SPEC-2-003 — CA-2-010; prova F2-PDF-001. Pré-condições: Autorizar a pasta e o layout do fallback; Guardar um ciclo de teste na central. Ler a amostra do layout autorizado preservando original e hash. Extrair campos administrativos e valores sem estimar ausências. Mostrar prévia com origem, linha/página e mapeamento; confirmar lote somente após validação completa. A origem é PDF_FALLBACK, distinta da API.

- [ ] Separar PDFs ilegíveis e entradas repetidas #projeto
  > SPEC-2-003 — CA-2-011; prova F2-PDF-ERROR-001. Pré-condições: Importar um demonstrativo pelo fallback. Exercitar pasta vazia, acesso negado, PDF ilegível, layout desconhecido, identidade ambígua, valor ausente e arquivo repetido. Erro vira exceção identificável e não lote parcial. Mesmo arquivo reimportado não altera total; revisão alterada mantém histórico e exige mapeamento aprovado.

- [ ] Fechar as evidências do ciclo importado #projeto
  > SPEC-2-003 — CA-2-012; prova F2-HANDOFF-001. Pré-condições: Recuperar o ciclo após uma falha; Demonstrar o resultado da entrada pela API; Separar PDFs ilegíveis e entradas repetidas. Consolidar ciclo importado pela rota viável, registro de decisão API/fallback, provas de proveniência, idempotência durável, permissão, falha e rollback, além das pendências para a fase seguinte. Rota inviável ou não usada exige motivo explícito e não conta como implementação. Solicitar demonstração e aceite do champion.

## Tasks — referência operacional

Esta tabela é uma projeção de critérios e dependências para compatibilidade. O status editável vive apenas nas caixas dos cards acima. Identificadores F2-Txxx são referências locais de evidência, não UUIDs do portal.

| Referência | Task | Papel de execução | SPEC | Critério | Prova | Evidência | Pré-condições | Ponto de parada |
|---|---|---|---|---|---|---|---|---|
| F2-T001 | Confirmar o ambiente e a fonte do primeiro ciclo | Champion + Ethos, conforme aceite | SPEC-2-001 | CA-2-001 | F2-CONFIG-001 | 05_entregas/F2-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T002 | Guardar um ciclo de teste na central | Champion + Ethos, conforme aceite | SPEC-2-001 | CA-2-002 | F2-CENTRAL-001 | 05_entregas/F2-CENTRAL-001.md | Confirmar o ambiente e a fonte do primeiro ciclo | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T003 | Impedir duplicação após reinício | Champion + Ethos, conforme aceite | SPEC-2-001 | CA-2-003 | F2-IDEMP-001 | 05_entregas/F2-IDEMP-001.md | Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T004 | Recuperar o ciclo após uma falha | Champion + Ethos, conforme aceite | SPEC-2-001 | CA-2-004 | F2-ROLLBACK-001 | 05_entregas/F2-ROLLBACK-001.md | Impedir duplicação após reinício | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T005 | Registrar o caminho autorizado de entrada | Champion + Ethos, conforme aceite | SPEC-2-002 | CA-2-005 | F2-API-CONFIG-001 | 05_entregas/F2-API-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T006 | Trazer um ciclo pela API | Champion + Ethos, conforme aceite | SPEC-2-002 | CA-2-006 | F2-API-001 | 05_entregas/F2-API-001.md | Registrar o caminho autorizado de entrada; Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T007 | Tratar indisponibilidade sem perder o ciclo | Champion + Ethos, conforme aceite | SPEC-2-002 | CA-2-007 | F2-API-ERROR-001 | 05_entregas/F2-API-ERROR-001.md | Trazer um ciclo pela API | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T008 | Demonstrar o resultado da entrada pela API | Champion + Ethos, conforme aceite | SPEC-2-002 | CA-2-008 | F2-API-HANDOFF-001 | 05_entregas/F2-API-HANDOFF-001.md | Tratar indisponibilidade sem perder o ciclo | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T009 | Autorizar a pasta e o layout do fallback | Champion + Ethos, conforme aceite | SPEC-2-003 | CA-2-009 | F2-PDF-CONFIG-001 | 05_entregas/F2-PDF-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T010 | Importar um demonstrativo pelo fallback | Champion + Ethos, conforme aceite | SPEC-2-003 | CA-2-010 | F2-PDF-001 | 05_entregas/F2-PDF-001.md | Autorizar a pasta e o layout do fallback; Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T011 | Separar PDFs ilegíveis e entradas repetidas | Champion + Ethos, conforme aceite | SPEC-2-003 | CA-2-011 | F2-PDF-ERROR-001 | 05_entregas/F2-PDF-ERROR-001.md | Importar um demonstrativo pelo fallback | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T012 | Fechar as evidências do ciclo importado | Champion + Ethos, conforme aceite | SPEC-2-003 | CA-2-012 | F2-HANDOFF-001 | 05_entregas/F2-HANDOFF-001.md | Recuperar o ciclo após uma falha; Demonstrar o resultado da entrada pela API; Separar PDFs ilegíveis e entradas repetidas | Acesso/regra ausente; nenhum dado real por inferência |
