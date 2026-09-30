# Controle de aprendizado contínuo

- 2026-08-31T14:50:00-03:00 · task F1-T002 · sem sinal reutilizável · a implementação confirmou os critérios e o fluxo já definido na SPEC, sem novo padrão técnico independente.
- 2026-08-31T17:56:00-03:00 · task F1-T003 · sem sinal reutilizável · a implementação confirmou os critérios da SPEC (negações e privacidade) sem novo padrão técnico independente.
- 2026-09-01T17:40:00-03:00 · task F1-T004 · sem sinal reutilizável · versionamento com auditoria append-only e rollback seguiu o desenho já previsto na SPEC (RN-1-004), sem novo padrão técnico independente.
- 2026-09-01T21:10:00-03:00 · task F1-T005 · capturado:06_notas/aprendizado-continuo/AP-2026-09-01-2108-verificacao-caminho-real.md · QA verde + DOM íntegro não pegaram overflow-hidden (sem rolagem) e bloqueio indevido por falta de TUSS; validar caminho real do usuário em UI.
- 2026-09-30T00:23:00-03:00 · task F1-T006 · capturado:06_notas/aprendizado-continuo/AP-2026-09-29-2122-assinatura-funcao-reuso-modulo.md · chamada de função reutilizada escrita com argumentos invertidos; conferir assinatura real no código-fonte antes de reusar módulos de tasks anteriores.
- 2026-09-30T00:51:00-03:00 · task F1-T007 · capturado:06_notas/aprendizado-continuo/AP-2026-09-29-2150-gate-alem-do-aceite.md · gate de elegibilidade ignorava status de origem e recusa de correção não tinha base no aceite; gate deve considerar todas as condições da SPEC e restrição sem frase na SPEC é escopo a mais.
- 2026-09-30T01:02:00-03:00 · task F1-T008 · capturado:06_notas/aprendizado-continuo/AP-2026-09-29-2205-patch-nao-idempotente.md · patches SEARCH/REPLACE empilhados corromperam o módulo com marcadores de conflito; regravar o arquivo completo quando o patch falhar ou se repetir.
- 2026-09-30T01:12:00-03:00 · task F1-T009 · sem sinal reutilizável · task de decisão humana: contrato apresentado com pontos de atenção explícitos e aprovado sem objeção; nenhuma execução técnica para gerar padrão reutilizável.
