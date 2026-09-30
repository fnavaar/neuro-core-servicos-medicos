# AP-2026-09-01-2108 — Verificar o caminho real do usuário, não só o DOM

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T005 / SPEC-1-002
- Sinal: a verificação automatizada (QA verde + DOM com todas as seções) passou, mas o teste humano falhou duas vezes por problemas invisíveis ao DOM: `overflow-hidden` no template do Skip impedia a rolagem da página única, e um bloqueio por falta de TUSS contrariava regra de negócio do champion.
- Evidência: `mainScrollHeight: 4135px` vs `clientHeight: 633px` com `overflow: hidden` (medição no preview); correção em `src/components/Layout.tsx` validada por screenshot e pelo champion.
- Regra reutilizável: em entrega de UI, exercitar o caminho real do usuário (rolar, clicar, recarregar com cache limpo) e medir overflow/scroll computado — não validar apenas estrutura do DOM ou QA do build.
- Quando aplicar: qualquer tela entregue em preview para teste humano.
- Quando não aplicar: entregas sem interface (contratos, documentos, dados).
- Confiança: alta — causa confirmada por medição e pelo teste humano após a correção.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
