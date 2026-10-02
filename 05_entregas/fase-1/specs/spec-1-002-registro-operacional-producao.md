# SPEC-1-002 — Registro operacional de prontuário/produção

**Fase:** 1  
**Status:** bloqueada  
**Dono:** Produto/operação do cliente, com execução do Ethos após desbloqueio da superfície  
**Origem no escopo:** D-001, D-004; RQ-004, RQ-005, RQ-012; Fase 1 de `03-Projeto/02-Escopo-Definitivo.md`  
**Degrau da solução:** construção mínima — registrar somente o conjunto operacional necessário ao ciclo financeiro, sem tentar construir prontuário clínico completo.

## Contexto e decisões fechadas

- **Estado atual:** a conferência cruza agenda/produção, planilhas e demonstrativos; há status operacionais e campos financeiros dispersos. Fonte: `03-Projeto/01-Escopo.md`, `03-Projeto/requisitos.md` RQ-004/RQ-005 e análises dos vídeos.
- **Estado desejado:** um usuário autorizado cria ou corrige um registro operacional com identificador técnico, dimensões mínimas, responsável, status de origem, evidência e histórico; registros incompletos ficam bloqueados, não viram produção válida.
- **Decisões já fechadas:** a aplicação é nova; MDMED não é referência do sistema-alvo; dados clínicos, diagnóstico e laudo ficam fora; o sistema não corrige presença, faturamento ou fonte legada por inferência.
- **Bloqueios:** a lista final de campos mínimos, a identidade oficial das entidades e a superfície técnica não estão persistidas. O contrato lógico abaixo permite fixture controlada; não executar contra dados reais até aprovação desses itens.

## Resultado observável

Uma pessoa autorizada cria um registro de atendimento/procedimento em uma fixture sintética, vê o estado `VALIDO` quando todas as dimensões mínimas estão presentes, recebe `BLOQUEADO` quando falta uma dimensão ou existe status não mapeado e consegue consultar a versão anterior após uma correção.

## Limites e dependências

- **Inclui:** registro operacional mínimo; profissional, unidade, entidade, serviço, data, competência, quantidade/valores quando disponíveis, status, responsável, evidência, validação e versionamento.
- **Fora de escopo:** prontuário clínico completo; nome de paciente como chave; diagnóstico; laudo; prescrição; marcação de presença; lançamento financeiro; integração MDMED; pagamento.
- **Entradas e pré-condições:** usuário autorizado; identidade da entidade/unidade/profissional; serviço conhecido ou pendência explícita; fixture sintética; política de acesso da SPEC-1-001.
- **Saídas/artefatos:** registro versionado; `validation_state`; lista de campos ausentes; evento de auditoria; payload elegível para SPEC-1-003 apenas quando `VALIDO`.
- **Dependências e responsáveis:** campos e identidade — cliente/financeiro; RLS — administrador; operação — recepção/profissional/analista conforme escopo; aceite — responsável financeiro/consultor.
- **Atores e permissões mínimas:** recepção/administrativo cria e corrige pendências atribuídas; profissional cria ou valida o próprio registro; analista consulta e corrige itens delegados; administrador configura campos e papéis, não altera conteúdo operacional sem trilha.
- **Superfícies/arquivos/configurações afetadas:** nova aplicação e seu modelo de registro; nenhuma tecnologia ou repositório foi definido, portanto não criar arquivos de produção fora da superfície autorizada.
- **Risco e plano B:** se a lista de campos ou a identidade oficial não for confirmada, executar somente o cenário sintético e registrar bloqueio; não preencher campos ausentes por aproximação textual.
- **Rollback ou reversão:** corrigir por nova versão, marcar a anterior como substituída e manter o histórico; não apagar o registro anterior nem alterar a fonte original.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Usuário → registro operacional | Nova aplicação | `record_id`, `entity_id`, `unit_id`, `professional_id`, `service_ref`, `event_date`, `competency`, `status_source`, `responsible_user_id`, `evidence_ref`, valores quando disponíveis | Sessão autenticada e RLS da SPEC-1-001 | Sem integração externa; repetição do mesmo `record_id + version` não cria duplicata | Campo obrigatório ausente ou identidade ambígua cria `BLOQUEADO` |
| Registro → validação | Regras desta SPEC | `validation_state`, `missing_fields`, `mapping_status`, `version` | Serviço interno dentro do escopo do usuário | Revalidação idempotente por `record_id + version` | Não transformar vazio em zero ou `ATENDIDO` |

| Campo lógico | Tipo lógico | Obrigatoriedade | Regra de qualidade | Fonte/observação |
|---|---|---|---|---|
| `record_id` | identificador técnico | Obrigatório | Estável e único no sistema | Contrato de fase 1 |
| `entity_id` | identificador | Obrigatório | Deve apontar para entidade oficial confirmada | D-001; bloqueio de identidade |
| `unit_id` | identificador | Obrigatório | Unidade deve pertencer à entidade | RQ-004 |
| `professional_id` | identificador | Obrigatório | Profissional deve pertencer ao escopo e à entidade | RQ-004/RQ-012 |
| `service_ref` | identificador + rótulo controlado | Obrigatório | Pelo menos um identificador ou rótulo aprovado; texto ambíguo bloqueia | RQ-004 |
| `event_date` | data | Obrigatório | Data válida; não inferir fuso ou competência ausente | Fase 1 |
| `competency` | período | Obrigatório | Período explícito, não apenas derivado silenciosamente | RQ-001/RQ-004 |
| `quantity` | número decimal | Condicional | Nulo quando não disponível; nunca substituir por zero sem regra | RQ-004 |
| `unit_value` / `gross_value` | moeda decimal | Condicional | Tipo e precisão validados; origem preservada | RQ-004 |
| `status_source` | enumeração preservada | Obrigatório | `AGENDADO`, `ATENDIDO`, `CANCELADO`, `FALTOU` quando presentes; desconhecido bloqueia | RQ-005 |
| `validation_state` | enumeração do sistema | Obrigatório | `RASCUNHO`, `VALIDO` ou `BLOQUEADO` | Regra desta SPEC |
| `responsible_user_id` | identificador | Obrigatório | Responsável dentro da hierarquia e do escopo | D-004 |
| `evidence_ref` | referência | Obrigatório | Aponta para evidência autorizada; não copia original | RQ-002/RQ-012 |
| `created_at`, `updated_at`, `created_by`, `updated_by`, `version` | auditoria | Automático | Imutável por versão e consistente | RQ-012 |
| conteúdo clínico | proibido no MVP | Nunca | Nome, diagnóstico, laudo e conteúdo clínico não são armazenados | D-004/RQ-012 |

## Fluxo e regras

1. Usuário autenticado inicia um registro dentro de seu escopo.
2. O sistema valida entidade, unidade, profissional, serviço, data, competência, responsável e evidência.
3. O sistema preserva o status de origem e não o reinterpreta.
4. Se as dimensões mínimas estiverem válidas, grava `VALIDO`; se houver ausência, ambiguidade ou dado proibido, grava `BLOQUEADO` sem liberar para fechamento.
5. Correção cria nova versão com autor, motivo e relação com a versão anterior.
6. Somente registros `VALIDO` sem bloqueio de minimização podem ser enviados ao contrato da SPEC-1-003.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Fixture com todas as dimensões mínimas e `status_source=ATENDIDO` | Registro criado como `VALIDO`, auditado e elegível para saída | Se auditoria falhar, não concluir a gravação |
| Limite | `quantity` ou valores ausentes, mas não necessários para o registro operacional | Registro permanece `VALIDO` se demais campos estiverem válidos; ausência fica explícita | Não preencher com zero; fase posterior trata impacto financeiro |
| Falha | Unidade desconhecida, profissional fora da entidade ou serviço ambíguo | Registro `BLOQUEADO`, sem saída financeira | Responsável corrige origem ou mapeamento e cria nova versão |
| Status antigo | Registro `AGENDADO` depois da data do atendimento | Preservar status e manter pendência; não converter para `ATENDIDO` | Encaminhar à recepção/administrativo responsável |
| Privacidade | Payload contém nome, diagnóstico ou laudo | Rejeitar e não persistir conteúdo proibido | Registrar somente tipo de bloqueio e remover o dado da fixture |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** Fase 1 de `03-Projeto/02-Escopo-Definitivo.md`; RQ-004, RQ-005 e RQ-012 de `03-Projeto/requisitos.md`; SPEC-1-001; `03-Projeto/03-Setup-Ethos/USER.md`.
2. **Alterar somente:** modelo, validação, estado e tela/fluxo mínimo do registro operacional na superfície autorizada.
3. **Não alterar:** campos clínicos, fonte legada, regras de repasse, central financeira, integração externa, pagamentos ou dados reais sem desbloqueio.
4. **Executar nesta ordem:** confirmar campos/identidade/superfície → criar fixture → validar principal/limite/falha → demonstrar versionamento → exportar somente registro `VALIDO` para SPEC-1-003.
5. **Parar e pedir validação quando:** campo mínimo, status, relação de entidade, RLS, identificador de serviço, ambiente ou regra de retenção estiverem ausentes.
6. **Estado válido ao parar:** nenhum registro real alterado; fixture e versões preservadas; bloqueios atribuídos.

## Checklist de execução

- [ ] Campos mínimos e identidade oficial confirmados.
- [ ] RLS da SPEC-1-001 aplicada ao fluxo.
- [ ] Registro válido criado com fixture sintética.
- [ ] Registro incompleto, ambíguo e com status antigo bloqueado corretamente.
- [ ] Correção gerou nova versão e manteve a anterior.
- [ ] Nenhum conteúdo clínico foi persistido.
- [ ] Somente registro `VALIDO` foi aceito pelo contrato de saída.

## Critérios de aceite

- [ ] **CA-1-006:** registro completo da fixture é criado com `record_id`, entidade, unidade, profissional, serviço, data, competência, status, responsável, evidência, auditoria e estado `VALIDO`.
- [ ] **CA-1-007:** ausência de dimensão obrigatória, identidade ambígua ou status desconhecido produz `BLOQUEADO` e impede saída para a central.
- [ ] **CA-1-008:** campos condicionais ausentes permanecem nulos/pendentes e nunca são convertidos silenciosamente em zero.
- [ ] **CA-1-009:** correção gera versão nova, relaciona a anterior, registra autor/motivo e não sobrescreve a fonte original.
- [ ] **CA-1-010:** conteúdo clínico e nome de paciente não entram no payload do MVP.
- [ ] **CA-1-011:** o executor para antes de dados reais se campos, identidade, RLS ou superfície não estiverem confirmados.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Criar registro sem `unit_id`, com serviço ambíguo e `status_source=STATUS_NOVO` | Submeter fixture `F1-INVALID-001` ao fluxo de cadastro | Registro não pode ficar `VALIDO` nem ser enviado à central | Resposta de validação, `missing_fields` e `mapping_status` |
| GREEN | Criar fixture completa e depois alterar `responsible_user_id` | Executar `F1-VALID-001` e `F1-REVISION-001` | Primeiro registro `VALIDO`; segunda operação cria nova versão e mantém histórico | Payloads, versões, auditoria e tela/relatório de demonstração |
| REFACTOR/REGRESSÃO | Repetir com quantidade nula, status `AGENDADO` pós-data e conteúdo clínico | Executar `F1-EDGE-001` a `F1-EDGE-003` | Nulo explícito, status preservado como pendência e conteúdo proibido rejeitado | Relatório de cenários e comparação com critérios CA-1-008/009/010 |

**Dados/fixtures:** uma entidade, uma unidade, um profissional, um serviço aprovado, um responsável sintético e três registros sem dados pessoais reais; valores podem ser nulos quando o cenário testa ausência.  
**Caminhos de erro obrigatórios:** campo ausente, identidade ambígua, status desconhecido, profissional fora do escopo, auditoria indisponível, conteúdo clínico e reprocessamento da mesma versão.  
**Evidência exigida:** payloads versionados, estados, eventos de auditoria e demonstração de que somente `VALIDO` segue para SPEC-1-003.

## Handoff e operação

- **Como demonstrar:** criar um registro completo, consultar o estado `VALIDO`, remover uma dimensão, corrigir e mostrar a nova versão sem apagar a anterior.
- **Como operar depois:** recepção/profissional mantêm origem; analista trata bloqueios; administrador mantém catálogo e RLS.
- **Como monitorar:** quantidade de registros `BLOQUEADO`, campos ausentes, status não mapeado, conteúdo rejeitado e versões reabertas.
- **Pendência conhecida:** campos mínimos, identidade oficial, retenção e superfície técnica ainda precisam ser persistidos como decisão executável.

## Tasks vinculadas

| ID | Task | Critério | Recorte da prova | Status |
|---|---|---|---|---|
| F1-T005 | Registrar contrato de campos mínimos | Campos, identidade, fronteira e retenção registrados | `F1-FIELDS-BASELINE` | bloqueada |
| F1-T006 | Criar registro operacional válido | Fixture completa produz `VALIDO` com auditoria e versão | `F1-VALID-001` | bloqueada |
| F1-T007 | Exercitar bloqueios, bordas e versionamento | Incompleto/ambíguo/status antigo bloqueiam; correção cria versão | `F1-INVALID-001`, `F1-EDGE-001` a `F1-EDGE-003` | bloqueada |
| F1-T008 | Entregar evidência do registro para o contrato | Só `VALIDO` segue; `BLOQUEADO` permanece fora | `F1-VALID-001` e `F1-BLOCKED-001` | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
