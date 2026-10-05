# AP-2026-10-01-1735 — Botão openui com link externo exige @OpenUrl

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T012 (SPEC-1-003) — cartões de status no canal web
- Sinal: botão de cartão openui com ação `@ToURL` ficou morto (clique sem efeito); `@ToURL` não existe na biblioteca — o passo correto para URL externa é `@OpenUrl`
- Evidência: relato do champion em 2026-10-01 ("o botão não funciona") + `skills/openui/SKILL.md`, seção Action (passos disponíveis: `@ToAssistant`, `@OpenUrl`, com exemplo `Button("View", Action([@OpenUrl("https://example.com")]))`); cartão corrigido com `@OpenUrl` sem nova reclamação
- Regra reutilizável: em qualquer botão/item openui que deva abrir link externo, usar `Action([@OpenUrl("https://...")])`; conferir a lista de passos da Action na skill antes de usar passo não documentado
- Quando aplicar: cartões de status/entrega com link para GitHub, preview ou documento externo
- Quando não aplicar: botões que enviam mensagem ao assistente (usar `@ToAssistant` ou rótulo puro)
- Confiança: alta — documentação da biblioteca + falha observada + correção aplicada sem recorrência
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto
