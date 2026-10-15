# AP-2026-10-15-1210 — Varredura de segredos sem GitHub Advanced Security e higiene do repo do sistema

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F2-T001 / SPEC-2-001 (prova F2-CONFIG-001)
- Sinal: o repositório do cliente (fnavaar/neuro-core-servicos-medicos) não tem GitHub Advanced Security — a chamada de secret scanning do MCP GitHub retorna erro "Repository does not have GitHub Advanced Security enabled". Na mesma rodada, a listagem do repo do sistema (fabioschneider9/p-gina-em-branco-njnhgvesg) revelou um `.env` commitado (78 bytes, não lido por política), registrado como risco com dono antes da task de persistência.
- Evidência: erro da ferramenta de scanning na F2-T001 (2026-10-06); listagem de arquivos do repo do sistema; `05_entregas/F2-CONFIG-001.md` §5/§8 (varredura local de 11 padrões executada antes do commit, resultado limpo; risco `.env` registrado).
- Regra reutilizável: antes de commitar documento ou código no repositório do cliente, rodar varredura local de padrões de segredo (script próprio) e registrar o resultado na prova; em qualquer task que toque o sistema, checar arquivos sensíveis do repo do sistema (ex.: `.env` commitado) e registrar pendência de remoção/rotação com dono antes de persistência.
- Quando aplicar: todo commit no projeto; obrigatório antes de tasks de persistência/integração (F2-T002 em diante).
- Quando não aplicar: não substitui revisão humana nem detecta segredos ofuscados; não autoriza ler o conteúdo do arquivo sensível.
- Confiança: alta — observação direta nas duas superfícies (erro da ferramenta + arquivo listado), com prova anexada.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto (o `.env` não foi lido).