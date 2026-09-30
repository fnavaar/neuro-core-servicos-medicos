# Changelog — Projeto Qualivida Serviços Médicos LTDA

> Registro de tudo que acontece no projeto, em ordem cronológica inversa (mais recente no topo).
> Formato: `- AAAA-MM-DD · [quem] · o que aconteceu`
> **Dúvidas para o consultor** entram como: `- AAAA-MM-DD · [quem] · DÚVIDA: …` — ele responde na próxima sincronização.

## Registro

- 2026-09-01 · [Fábio Schneider] · Task F1-T005 concluída: contrato F1-FIELDS-BASELINE registrado com 8 decisões do champion — catálogo próprio por clínica (27 serviços, 21 pagadores, fonte Valores_Qualivida.xlsx), identidade `guide_number` + `record_id`, retenção permanente, RN-1-005 (sem vínculo = não atendido/não agendável), RN-1-006 (catálogo por clínica), RN-1-007 (vigência imediata/futura/retroativa sem alterar histórico) e código TUSS opcional (pacote contratual aceito); catálogo estruturado e tela implementados no Skip v0.0.25; dois debugs no caminho (rolagem da página única e bloqueio indevido por falta de TUSS), ambos corrigidos e provados no preview; evidência em `05_entregas/F1-T005-evidencia.md`.
- 2026-09-01 · [Fábio Schneider] · Task F1-T004 concluída: alteração sensível de escopo versionada (before/after) com auditoria append-only e rollback restaurando a política anterior sem apagar eventos; não-admin e rollback inválido negados; demonstrado no Skip versão 0.0.13, com evidência em `05_entregas/F1-T004-evidencia.md`.
- 2026-08-31 · [Fábio Schneider] · Task F1-T003 concluída: três negações de escopo (entidade, unidade, profissional) demonstradas sem vazamento de payload e payload clínico não persistido; cenários `RLS-ENTITY-DENY`, `RLS-UNIT-DENY`, `RLS-PROF-DENY` e privacidade no Skip versão 0.0.9, com evidência em `05_entregas/F1-T003-evidencia.md`.
- 2026-08-31 · [Fábio Schneider] · Task F1-T002 concluída: fixture sintética com entidade, unidade, profissional, usuários e escopos; política RN-1-001 e cenário RLS-ALLOW demonstrados no Skip versão 0.0.6, sem dados reais ou conteúdo clínico.
- 2026-08-31 · [Fábio Schneider] · Task F1-T001 concluída: pré-condições de governança confirmadas e formalizadas em `05_entregas/F1-T001-evidencia.md`; Qualivida é a entidade principal, Fábio é CEO/champion/único responsável, a superfície e o runner são ETHOS, e há 5 unidades (2 Qualivida e 3 nomes pendentes).
- 2026-08-31 · [Eduard / Fábio] · F1-T001 atualizada com a identidade operacional confirmada: Qualivida como entidade principal; Fábio Schneider como CEO, champion e único responsável; 5 unidades (2 Qualivida e 3 com nomes pendentes); superfície e runner ETHOS.
- 2026-08-19 · [Adapta Labs] · Pasta operacional do cliente criada com o template público e os artefatos liberados da Fase 1; exportação validada sem publicação externa.