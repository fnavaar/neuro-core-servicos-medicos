# STATUS — Projeto Qualivida / Neuro Core

**Atualizado em:** 2026-10-02 · **Por:** Adapta Labs

## Onde estamos

- **Fase atual:** 2 — Central financeira e ingestão multi-fonte, aberta em 2026-10-02.
- **Objetivo:** importar um ciclo de teste pela API preferencial ou pelo fallback de PDF documentado, preservando origem, versão, idempotência durável e recuperação.
- **Tasks:** 0/12. Cards e critérios em 04_fase-atual/fase.md; três SPECs em 04_fase-atual/specs/.
- **Próxima task:** Confirmar o ambiente e a fonte do primeiro ciclo.
- **Execução:** uma task por vez; dependências e permissões conferidas antes de implementar.

## Contexto confirmado

- Fábio Schneider é champion e responsável único. Qualivida é entidade operacional principal.
- A central consumidora é o próprio sistema, conforme decisão da F1-T009.
- Preservar production-record.v1 e chave sistema:record_id:versão; valores ausentes continuam null.
- Execução assistida pelo Ethos no ambiente existente; submódulo 07_sistemas preservado.

## Pré-condições da Fase 2

| Item | Quem confirma | Quando |
|---|---|---|
| Ambiente existente, cópia de implementação, armazenamento durável e escopo de acesso | Champion/Ethos | Antes da primeira implementação |
| Endpoint, autenticação segura, esquema, paginação e limites da API; ou inviabilidade documentada | Champion/administrador | Antes do transporte pela API |
| Pasta/fonte em leitura, amostra e layout reconhecido, mapeamento e revisão de OCR | Champion/financeiro | Antes da importação PDF |
| Catálogo vigente e nomes das três unidades restantes | Champion/financeiro | Antes de dados reais |

Nenhum segredo é gravado no repositório. Fixtures sintéticas isoladas até autorização de fonte real.

## Entregas preservadas

- **Fase 1:** 12/12 concluídas no registro do cliente; fase encerrada administrativamente por autorização expressa do consultor em 2026-10-02. Unidade arquivada em 05_entregas/fase-1/.
- Evidências F1-T001 a F1-T012 continuam em 05_entregas; pacote F1-HANDOFF-001 em 05_entregas/F1-T012-handoff-fase-2.md.
- A liberação não registra nova reexecução técnica ou novo teste humano da Fase 1; conserva os aceites e limitações existentes.

## Próxima demonstração

Após o primeiro ciclo sintético importado: mostrar origem/versão, reiniciar, reimportar sem duplicação e recuperar falha. Aceite do champion registrado por task.
