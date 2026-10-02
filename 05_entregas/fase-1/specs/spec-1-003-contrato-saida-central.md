# SPEC-1-003 — Contrato de saída para a central financeira

**Fase:** 1  
**Status:** bloqueada  
**Dono:** Responsável financeiro e administrador da nova aplicação  
**Origem no escopo:** D-001, D-002, D-003, D-004; RQ-002, RQ-003, RQ-004, RQ-012; Fase 1 de `03-Projeto/02-Escopo-Definitivo.md`  
**Degrau da solução:** construção mínima — nesta fase é contrato/fixture versionado, não integração externa; a API e o fallback de PDF pertencem à Fase 2.

## Contexto e decisões fechadas

- **Estado atual:** dados de produção, financeiro e demonstrativos são cruzados manualmente em fontes distintas; não há contrato versionado da nova aplicação para a central.
- **Estado desejado:** a nova aplicação produz um envelope estável, rastreável e minimizado que a central poderá consumir na Fase 2; o contrato aceita apenas registros `VALIDO` e mantém proveniência.
- **Decisões já fechadas:** hub multi-fonte sem MDMED como referência; API será preferencial em fase posterior; PDF/pastas são fallback; não há integração externa na demonstração desta fase.
- **Bloqueios:** endpoint, autenticação, identidade oficial das entidades, política de retenção e tecnologia da central não estão registrados. Não criar credencial, chamada de API ou escrita na central nesta SPEC.

## Resultado observável

Dado um registro `VALIDO` da SPEC-1-002, o sistema gera um envelope `production-record.v1` determinístico, com origem, versão, dimensões mínimas, status e responsável. Dado um registro `BLOQUEADO`, duplicado ou com identidade não confirmada, não gera saída elegível.

## Limites e dependências

- **Inclui:** contrato lógico versionado, validação, chave de idempotência, proveniência, minimização e fixture de consumo pela central.
- **Fora de escopo:** API real, leitura de pasta/PDF, autenticação externa, ingestão, normalização multi-fonte, reconciliação, pagamento, MDMED e alteração da central.
- **Entradas e pré-condições:** saída `VALIDO` da SPEC-1-002; catálogo de entidades/unidades/profissionais; ambiente de teste isolado; aprovação do formato antes de integração.
- **Saídas/artefatos:** envelope JSON/objeto equivalente, versão do contrato, resultado de validação e relatório de rejeições.
- **Dependências e responsáveis:** formato e identidade — cliente/financeiro; central consumidora — responsável financeiro/administrador; execução — Ethos após superfície autorizada.
- **Atores e permissões mínimas:** aplicação gera saída; analista/financeiro consulta fixture; administrador aprova versão do contrato; nenhum ator escreve na central nesta fase.
- **Superfícies/arquivos/configurações afetadas:** contrato no repositório/superfície autorizada e fixture de teste; caminho real depende do repositório informado pelo cliente.
- **Risco e plano B:** se a central não confirmar o contrato, manter a versão como `DRAFT` e anexar rejeições; não adaptar o payload durante a Fase 2 sem emenda/SPEC.
- **Rollback ou reversão:** versionar novo contrato sem apagar `production-record.v1`; rejeitar a versão não aprovada e restaurar a última versão aprovada da fixture.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Registro válido → fixture da central | Nova aplicação | Envelope `production-record.v1` abaixo | Não há chamada externa; leitura local autorizada | `idempotency_key = source_system + source_record_id + version`; repetir não duplica | Registro bloqueado, contrato inválido ou identidade ausente é rejeitado sem saída parcial |
| Fixture → futura central financeira | Contrato aprovado na Fase 1 | API e credencial somente na Fase 2 | Não provisionar token nesta fase | Timeout/retry pertencem à SPEC da Fase 2 | Falha futura deve deixar lote pendente e preservar a versão |

### Envelope mínimo `production-record.v1`

```json
{
  "contract_version": "production-record.v1",
  "record_id": "REC-TEST-001",
  "source": {
    "system": "nova-aplicacao-prontuario-producao",
    "source_record_id": "REC-TEST-001",
    "version": 1,
    "evidence_ref": "fixture://fase-1/REC-TEST-001"
  },
  "entity_id": "ENTITY-TEST-001",
  "unit_id": "UNIT-TEST-001",
  "professional_id": "PROF-TEST-001",
  "service_ref": {"id": "SERVICE-TEST-001", "label": "servico-controlado"},
  "event_date": "2026-08-01",
  "competency": "2026-08",
  "quantity": null,
  "unit_value": null,
  "gross_value": null,
  "status_source": "ATENDIDO",
  "validation_state": "VALIDO",
  "responsible_user_id": "USER-RESP-001",
  "audit": {"created_by": "USER-CREATE-001", "updated_by": "USER-CREATE-001", "version": 1},
  "idempotency_key": "nova-aplicacao-prontuario-producao:REC-TEST-001:1"
}
```

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-005 | `validation_state=VALIDO` e identidade confirmada | Gerar envelope exatamente uma vez por chave de idempotência | Versão repetida retorna o mesmo resultado, sem duplicar | RQ-002/RQ-003 |
| RN-1-006 | Registro `BLOQUEADO`, campo obrigatório ausente ou status desconhecido | Rejeitar e devolver motivo estruturado | Não gerar payload parcial | RQ-004/RQ-005 |
| RN-1-007 | Campo clínico ou identificador pessoal não necessário aparece na entrada | Remover/rejeitar antes do envelope | Não mascarar silenciosamente uma violação | RQ-012 |
| RN-1-008 | Contrato aprovado muda | Criar nova versão e manter a anterior | Nenhuma emenda silenciosa no mesmo `contract_version` | D-002/D-003 |

## Fluxo e regras

1. Ler somente registros `VALIDO` da SPEC-1-002 no escopo autorizado.
2. Verificar entidade, unidade, profissional, serviço, competência, status, responsável, evidência e versão.
3. Remover campos fora do contrato; se a remoção esconder dado proibido ou ambiguidade, rejeitar.
4. Montar o envelope `production-record.v1` e calcular a chave de idempotência.
5. Validar o envelope com fixture positiva e negativas.
6. Entregar a fixture ao responsável financeiro para aprovação do contrato; não chamar API nem escrever na central.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Registro `VALIDO`, identidade confirmada e versão 1 | Envelope completo e determinístico | Se contrato não validar, rejeitar e registrar campo |
| Limite | `quantity`, `unit_value` e `gross_value` nulos por indisponibilidade na Fase 1 | Envelope preserva nulos e estado `VALIDO` somente se regra de fase permitir; não inventa valor | Fase 2/3 trata impacto financeiro; se o campo for obrigatório no contrato aprovado, bloquear |
| Falha | Registro `BLOQUEADO`, duplicado ou entidade não confirmada | Nenhum envelope elegível; motivo e origem retornados | Corrigir na SPEC-1-002 ou obter decisão; não adaptar no transformador |
| Idempotência | Mesmo registro e mesma versão enviados duas vezes | Uma chave e um resultado; nenhum total duplicado | Se houver dois resultados, bloquear a liberação do contrato |
| Segurança | Entrada inclui nome de paciente ou conteúdo clínico | Rejeição sem persistir o campo no envelope | Registrar apenas categoria do erro e referência da fixture |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** Fase 1 e seção de sistemas/fontes de `03-Projeto/02-Escopo-Definitivo.md`; RQ-002, RQ-003, RQ-004 e RQ-012; SPEC-1-002; `03-Projeto/03-Setup-Ethos/sugestoes-conectores-automacoes.md`.
2. **Alterar somente:** contrato lógico, validação e fixture da Fase 1.
3. **Não alterar:** credenciais, endpoints, central financeira real, pastas/PDF, MDMED, arquivos originais ou qualquer fonte externa.
4. **Executar nesta ordem:** validar origem → transformar somente `VALIDO` → conferir schema lógico → testar idempotência/negações → submeter contrato ao aprovador.
5. **Parar e pedir validação quando:** surgir decisão sobre endpoint, token, autenticação, campo não previsto, identidade oficial, retenção ou escrita na central.
6. **Estado válido ao parar:** fixture versionada e rejeições reproduzíveis; nenhuma chamada externa ou dado real alterado.

## Checklist de execução

- [ ] Contrato `production-record.v1` aprovado pelo responsável da central.
- [ ] Fixture positiva e quatro fixtures negativas executadas.
- [ ] Chave de idempotência comprovada sem duplicação.
- [ ] Campos proibidos rejeitados sem persistência.
- [ ] Registros `BLOQUEADO` impedidos de gerar saída.
- [ ] Nenhuma credencial, endpoint ou escrita externa usada.
- [ ] Versão e evidências entregues para a SPEC da Fase 2.

## Critérios de aceite

- [ ] **CA-1-012:** registro `VALIDO` gera envelope `production-record.v1` com todos os campos obrigatórios, origem, versão, responsável e chave de idempotência.
- [ ] **CA-1-013:** repetir o mesmo registro/versão produz o mesmo resultado e não duplica saída.
- [ ] **CA-1-014:** registro `BLOQUEADO`, identidade não confirmada ou status desconhecido não gera envelope elegível.
- [ ] **CA-1-015:** quantidade/valores ausentes permanecem nulos ou geram bloqueio conforme contrato aprovado; nunca são estimados.
- [ ] **CA-1-016:** conteúdo clínico e identificador pessoal não necessário são rejeitados e não aparecem na saída.
- [ ] **CA-1-017:** a SPEC não usa credenciais, API, PDF/pastas ou escrita na central na Fase 1.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Transformar `F1-BLOCKED-001` e `F1-DUPLICATE-001` | Executar a validação do contrato sobre registro bloqueado e duplicado | Nenhum envelope elegível; duplicidade explicitada | Rejeições com código, origem e versão |
| GREEN | Transformar `F1-VALID-001` duas vezes | Executar o transformador lógico e comparar envelopes | Envelopes idênticos; uma chave de idempotência; todos os CA-1-012/013 passam | JSONs, comparação determinística e relatório |
| REFACTOR/REGRESSÃO | Adicionar campo clínico, alterar contrato e repetir entradas | Executar `F1-PRIVACY-001` e regressão das fixtures anteriores | Campo proibido rejeitado; versão anterior continua válida; nenhuma fixture anterior muda | Diff de contrato, rejeições e regressão |

**Dados/fixtures:** `F1-VALID-001`, `F1-BLOCKED-001`, `F1-DUPLICATE-001`, `F1-NULL-VALUE-001`, `F1-PRIVACY-001`; todos sintéticos e referenciados por `fixture://fase-1/`.  
**Caminhos de erro obrigatórios:** bloqueio de origem, identidade ambígua, duplicidade, nulo, campo clínico, contrato inválido e tentativa de integração externa.  
**Evidência exigida:** envelopes, rejeições, comparação de idempotência, versão do contrato e confirmação do responsável financeiro.

## Handoff e operação

- **Como demonstrar:** gerar o envelope positivo, repetir a entrada, rejeitar um bloqueado e mostrar que nenhuma chamada externa ocorreu.
- **Como operar depois:** administrador versiona o contrato; financeiro aprova; Fase 2 implementa transporte conforme contrato aprovado.
- **Como monitorar:** rejeições por origem, duplicidade, contrato, privacidade e identidade; nunca considerar ausência de erro como aceite.
- **Pendência conhecida:** API, PDF/pastas, autenticação, endpoint, retenção e central real pertencem a gates posteriores.

## Tasks vinculadas

| ID | Task | Critério | Recorte da prova | Status |
|---|---|---|---|---|
| F1-T009 | Registrar aprovação do contrato de saída | Versão, campos e idempotência aprovados ou rejeitados com motivo | `F1-CONTRACT-APPROVAL` | bloqueada |
| F1-T010 | Gerar envelope válido versionado | Envelope determinístico com origem, versão e chave | `F1-VALID-001` | bloqueada |
| F1-T011 | Exercitar rejeição, idempotência e privacidade | Bloqueio/duplicidade/campo clínico tratados sem saída indevida | `F1-BLOCKED-001`, `F1-DUPLICATE-001`, `F1-PRIVACY-001` | bloqueada |
| F1-T012 | Fechar evidências e handoff para a Fase 2 | Pacote completo sem API/PDF ou integração externa | `F1-HANDOFF-001` | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
