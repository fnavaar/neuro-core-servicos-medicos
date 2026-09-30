# AP-2026-09-29-2150 — Restrição além do aceite e elegibilidade que ignora status de origem

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T007 / SPEC-1-002 (registro operacional)
- Sinal: dois defeitos da mesma natureza na F1-T007. (1) A função de elegibilidade para a central checava apenas `validationState === 'VALIDO'` e ignorava o `status_source` — um registro AGENDADO pós-data aparecia como produção enviável, quando a SPEC manda manter pendência. (2) O botão de correção recusava registro VALIDO ("se aplica a registro BLOQUEADO") — restrição sem base no critério CA-1-009, que não limita a correção a registro bloqueado.
- Evidência: 06_notas/debug/debug-2026-09-29-f1t007-elegibilidade-central.md (duas rodadas, reprodução no preview, correções v0.0.32 e v0.0.34).
- Regra reutilizável: (a) gate de saída/elegibilidade deve considerar TODAS as condições da SPEC (estado de validação E status de origem), não apenas a mais óbvia; (b) restrição que não está no critério de aceite é escopo a mais — antes de recusar um caso, procurar a frase da SPEC que autoriza a recusa; sem frase, não recusar.
- Quando aplicar: qualquer implementação de gate, filtro ou regra de bloqueio derivada de SPEC.
- Quando não aplicar: restrições de segurança/LGPD, que valem mesmo sem menção explícita no aceite (linha vermelha).
- Confiança: alta — ambas as causas reproduzidas no preview e corrigidas com verificação.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
