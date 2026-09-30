# AP-2026-09-29-2122 — Verificar assinatura de função ao reutilizar módulo existente

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T006 / SPEC-1-002 (registro operacional de prontuário/produção)
- Sinal: ao reutilizar `availableServices` do catálogo da F1-T005, a chamada foi escrita com a ordem de argumentos invertida (unitId no lugar do payerId); a assinatura real é `availableServices(payerId, unitId)`. O erro foi detectado na leitura do módulo antes do build, não pelo QA.
- Evidência: diff da F1-T006 (`src/lib/operational-record.ts`) — primeira versão chamava `availableServices('UNIT-TEST-001')`; correção trocou para verificação por `isServiceOffered` em todos os pagadores da unidade; pipeline v0.0.27 verde após a correção.
- Regra reutilizável: antes de reutilizar uma função de módulo existente, conferir a assinatura real no código-fonte (nome, ordem e tipos dos parâmetros) em vez de deduzir pelo nome; ao reutilizar regras de negócio (RN-1-005), preferir a função que expressa a regra diretamente.
- Quando aplicar: qualquer task que importe funções de módulos criados em tasks anteriores do mesmo projeto.
- Quando não aplicar: funções recém-criadas na própria task, cuja assinatura é definida junto com o uso.
- Confiança: alta — erro real observado no diff e corrigido antes do build, com evidência no histórico de versões do Skip.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
