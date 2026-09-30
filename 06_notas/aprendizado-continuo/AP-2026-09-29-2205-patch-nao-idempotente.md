# AP-2026-09-29-2205 — Patch SEARCH/REPLACE não é idempotente: regravar o módulo quando o arquivo corrompe

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T008 / SPEC-1-002
- Sinal: três patches SEARCH/REPLACE consecutivos no mesmo arquivo deixaram marcadores de conflito (`=======`) residuais e duplicaram uma função — o pipeline falhou em duas versões (v0.0.37/v0.0.38) antes de o módulo ser regravado completo (v0.0.39, verde de primeira).
- Evidência: falhas de build/lint em v0.0.37 e v0.0.38 citando diff markers em `src/lib/operational-record.ts`; correção por regravação completa em v0.0.39.
- Regra reutilizável: quando um patch falhar ou o arquivo já tiver sido alterado por patch na mesma sessão, reler o arquivo e preferir regravação completa (write) a empilhar novos patches; após qualquer patch, conferir ausência de marcadores de conflito antes de rodar o pipeline.
- Quando aplicar: edição de arquivos via SEARCH/REPLACE em superfícies remotas (Skip/GitHub), especialmente mais de um patch no mesmo arquivo na mesma sessão.
- Quando não aplicar: primeira edição simples e isolada de arquivo, onde patch único e verificado é mais barato.
- Confiança: alta — falha observada duas vezes no pipeline e correção verificada.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
