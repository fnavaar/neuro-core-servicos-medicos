# SPEC-1-001 — Governança de responsáveis e acesso por escopo

**Fase:** 1  
**Status:** bloqueada  
**Dono:** Administrador do sistema, com validação do responsável financeiro e do cliente  
**Origem no escopo:** D-004; RQ-012; Fase 1 de `03-Projeto/02-Escopo-Definitivo.md`  
**Degrau da solução:** construção mínima — a aplicação-alvo é nova e ainda não há plataforma técnica registrada; a entrega deve usar somente a superfície autorizada pelo cliente e parar se ela não estiver disponível.

## Contexto e decisões fechadas

- **Estado atual:** o processo distribui produção e conferência entre fontes e pessoas; não existe no workspace uma matriz operacional persistida de papéis, escopos e responsáveis da nova aplicação. Fonte: `03-Projeto/01-Escopo.md`, `03-Projeto/requisitos.md` e `04-Mapeamento-Processos/`.
- **Estado desejado:** usuários, entidades, unidades, profissionais e responsáveis têm escopo explícito; uma leitura ou alteração fora do escopo é negada e cada ação sensível gera auditoria.
- **Decisões já fechadas:** RLS é obrigatória; o agente não aprova pagamento, regra ou fechamento; conteúdo clínico não entra no MVP; MDMED não é fonte do sistema-alvo.
- **Bloqueios:** a relação oficial Neuro Core/Qualivida/Neurocor, a matriz final de papéis/RLS, os campos mínimos e a superfície técnica da nova aplicação não estão registrados no workspace. Esta SPEC pode ser demonstrada apenas com fixture sintética até esses itens serem confirmados; não liberar dados reais.

## Resultado observável

Um administrador cria uma organização, uma unidade, um profissional, usuários e seus escopos de acesso em uma fixture controlada. Um profissional vê somente registros vinculados ao seu escopo; um usuário sem escopo recebe negação; uma alteração de responsável fica registrada com ator, data, ação e versão.

## Limites e dependências

- **Inclui:** entidades técnicas, unidades, usuários, papéis, escopos, responsáveis hierárquicos, autorização por entidade/unidade/profissional e auditoria das ações sensíveis.
- **Fora de escopo:** prontuário clínico completo, diagnóstico, laudo, pagamento, aprovação de fechamento, integração externa e dados reais de pacientes.
- **Entradas e pré-condições:** fixture sintética; matriz de acesso confirmada pelo cliente antes de uso real; superfície técnica e ambiente de teste autorizados.
- **Saídas/artefatos:** registros de identidade e escopo; resultado de autorização; evento de auditoria; relatório de testes positivos e negativos.
- **Dependências e responsáveis:** relação cadastral e papéis — cliente/administrador; critério de aceite — responsável financeiro/consultor; execução — Ethos somente após desbloqueio.
- **Atores e permissões mínimas:** administrador gerencia configuração; responsável financeiro consulta e atribui decisões no escopo aprovado; analista opera registros pendentes; profissional consulta/valida o próprio escopo; recepção/administrativo cria ou corrige pendências do escopo atribuído.
- **Superfícies/arquivos/configurações afetadas:** nova aplicação e sua camada de autorização; o repositório e a tecnologia ainda não foram informados. Não criar integração presumida.
- **Risco e plano B:** se a matriz ou a superfície não forem confirmadas, manter a demonstração em fixture e produzir um relatório de bloqueio; não copiar dados para uma ferramenta intermediária.
- **Rollback ou reversão:** desativar o usuário/escopo criado em fixture, preservar o evento de auditoria e restaurar a versão anterior da política; nunca apagar o histórico para esconder uma concessão.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Cadastro de identidade → autorização | Nova aplicação, após confirmação do cliente | `entity_id`, `unit_id`, `professional_id`, `user_id`, `role`, `scope_status`, `version` | Sessão autenticada; alteração de RLS somente por administrador autorizado | Não há integração externa nesta SPEC; atualização deve ser versionada e idempotente por `user_id + scope_version` | Escopo inválido ou duplicado bloqueia a gravação e não altera a política vigente |
| Autorização → auditoria | Nova aplicação | `actor_id`, `action`, `target_type`, `target_id`, `before_version`, `after_version`, `occurred_at`, `result` | Evento somente de escrita pelo serviço de auditoria; leitura conforme papel | Repetição do mesmo evento deve manter `event_id` estável | Falha de auditoria bloqueia alteração sensível e mantém a política anterior |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-001 | Usuário consulta ou altera um registro | Permitir somente se o escopo do usuário cobrir a entidade, unidade e profissional do registro | Ausência de qualquer dimensão impede acesso | Fase 1; RQ-012 |
| RN-1-002 | Usuário tenta alterar política de acesso | Permitir somente ao administrador autorizado | Administrador não pode apagar auditoria nem conceder escopo fora da matriz confirmada | D-004; RQ-012 |
| RN-1-003 | Registro contém conteúdo clínico não necessário | Recusar o campo no contrato do MVP | Se a fonte exigir o campo, criar bloqueio e não persistir o conteúdo | D-004; RQ-012 |
| RN-1-004 | Alteração sensível é aceita | Registrar antes/depois, ator, resultado e versão | Falha de auditoria mantém a versão anterior | Fase 1; requisito de trilha |

## Fluxo e regras

1. Criar a fixture sintética com `ENTITY-TEST-001`, `UNIT-TEST-001`, `PROF-TEST-001` e usuários sem dados pessoais reais.
2. Registrar os papéis e escopos aprovados para o cenário de teste.
3. Criar um registro operacional vinculado à entidade, unidade e profissional da fixture.
4. Executar leitura e alteração com usuário autorizado e registrar o resultado.
5. Repetir com usuário sem entidade, sem unidade e com outro profissional; cada tentativa deve ser negada.
6. Alterar o responsável por usuário autorizado; conferir versão anterior, versão nova e auditoria.
7. Parar antes de qualquer dado real se a matriz final, a identidade oficial ou a superfície técnica não estiverem confirmadas.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | `PROF-USER-001` cobre `ENTITY-TEST-001/UNIT-TEST-001/PROF-TEST-001` | Leitura e alteração do registro permitidas; evento de auditoria criado | Se auditoria falhar, negar alteração e manter versão anterior |
| Limite | Usuário cobre entidade, mas não unidade ou profissional | Acesso negado sem revelar o conteúdo do registro | Registrar resultado `DENIED` sem copiar payload sensível |
| Falha | Tentativa de conceder escopo fora da matriz ou duplicar versão | Operação recusada; política ativa permanece intacta | Encaminhar ao administrador/cliente com evidência do conflito |
| Privacidade | Payload inclui nome, diagnóstico ou laudo | Validação falha; dado não é persistido | Remover do fixture e registrar bloqueio de minimização |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** `03-Projeto/02-Escopo-Definitivo.md` seção 4 e Fase 1; `03-Projeto/requisitos.md` RQ-012; esta SPEC inteira; `03-Projeto/03-Setup-Ethos/USER.md`.
2. **Alterar somente:** a camada de identidade/autorização e os artefatos de fixture da Fase 1 na superfície técnica autorizada.
3. **Não alterar:** MDMED, arquivos originais, dados clínicos, regras financeiras, pagamentos, fechamento, permissões fora da matriz ou integrações externas.
4. **Executar nesta ordem:** confirmar superfície e matriz → criar fixture → aplicar política → executar cenários positivos/negativos → conferir auditoria → anexar evidências.
5. **Parar e pedir validação quando:** faltar matriz RLS, relação oficial das entidades, campo de identificação, acesso ao ambiente, definição de retenção ou comando de teste do repositório.
6. **Estado válido ao parar:** fixture isolada, política anterior preservada, nenhum dado real copiado e bloqueio descrito com responsável.

## Checklist de execução

- [ ] Matriz de papéis, escopos e responsáveis confirmada pelo cliente.
- [ ] Superfície técnica e ambiente de teste autorizados.
- [ ] Fixture sintética criada sem dados pessoais reais.
- [ ] Acesso permitido e três negações de acesso demonstrados.
- [ ] Alteração sensível auditada com versão anterior e posterior.
- [ ] Rollback da fixture demonstrado sem apagar o histórico.

## Critérios de aceite

- [ ] **CA-1-001:** usuário autorizado lê e altera somente um registro dentro de entidade, unidade e profissional atribuídos; a evidência contém o escopo aplicado.
- [ ] **CA-1-002:** usuários sem entidade, unidade ou profissional compatível recebem negação em todos os três cenários, sem vazamento do payload.
- [ ] **CA-1-003:** toda alteração de responsável ou escopo gera auditoria com ator, alvo, resultado, versão anterior e versão posterior.
- [ ] **CA-1-004:** payload com nome de paciente, diagnóstico ou laudo não é persistido no MVP.
- [ ] **CA-1-005:** sem matriz RLS, identidade oficial e superfície técnica confirmadas, a execução para e não toca dados reais.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Usuário sem unidade tenta consultar registro da fixture | Executar cenário `RLS-UNIT-DENY` com usuário sem `UNIT-TEST-001` | A operação falha com negação; se passar, a política está insegura | Log/captura do cenário e payload não retornado |
| GREEN | Usuário autorizado consulta, altera responsável e tenta negar três acessos | Executar cenários `RLS-ALLOW`, `RLS-ENTITY-DENY`, `RLS-UNIT-DENY`, `RLS-PROF-DENY` | Permissão correta, negações corretas e auditoria criada | Relatório de execução com `event_id` e versões |
| REFACTOR/REGRESSÃO | Repetir após alterar a política e restaurar a versão anterior | Executar os mesmos cinco cenários e o rollback da fixture | Nenhum escopo antigo reaparece; auditoria continua íntegra | Comparação de resultados antes/depois e registro de rollback |

**Dados/fixtures:** `ENTITY-TEST-001`, `UNIT-TEST-001`, `PROF-TEST-001`; três usuários sintéticos com escopos distintos; nenhum nome, CPF, diagnóstico, laudo ou conteúdo clínico.  
**Caminhos de erro obrigatórios:** escopo ausente, escopo parcial, duplicidade de versão, auditoria indisponível, payload proibido e tentativa de alteração administrativa não autorizada.  
**Evidência exigida:** matriz confirmada, relatório dos cenários, eventos de auditoria e decisão de desbloqueio para dados reais.

## Handoff e operação

- **Como demonstrar:** abrir o registro sintético como usuário autorizado e repetir como usuário sem cada dimensão do escopo.
- **Como operar depois:** administrador mantém papéis; responsável financeiro mantém responsáveis; usuários operacionais não alteram a política.
- **Como monitorar:** negações anormais, alterações de escopo, falhas de auditoria e acessos fora do padrão.
- **Pendência conhecida:** matriz RLS, identidade oficial, retenção e superfície técnica precisam ser registradas antes do uso real.

## Tasks vinculadas

| ID | Task | Critério | Recorte da prova | Status |
|---|---|---|---|---|
| F1-T001 | Registrar pré-condições da governança | Matriz RLS, identidade, superfície e runner registrados ou bloqueio formal | `F1-FIELDS-BASELINE`/bloqueios da SPEC | bloqueada |
| F1-T002 | Criar fixture e política mínima de escopo | Usuário autorizado lê o registro dentro do escopo | `RLS-ALLOW` | bloqueada |
| F1-T003 | Exercitar negações e privacidade | Três negações e payload clínico não persistido | `RLS-ENTITY-DENY`, `RLS-UNIT-DENY`, `RLS-PROF-DENY` | bloqueada |
| F1-T004 | Auditar alteração e demonstrar rollback de RLS | Versões e auditoria preservadas no rollback | `RLS-ROLLBACK` | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
