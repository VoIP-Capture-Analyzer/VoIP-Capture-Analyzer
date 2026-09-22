# Changelog

## v2.62.0

[Added]
- Assets alternativos de branding **VCA** e **VC Analyzer** em `app/static/branding-ready/`, reutilizando o símbolo atual e permanecendo totalmente inativos no runtime.
- Testes de contrato para a arquitetura Ajuda/Projeto, faixa compacta de contexto, retorno ao topo real da página e isolamento dos assets de branding futuro.

[Changed]
- No menu `Projeto`, o rótulo `CONTRIBUTING` passa a ser apresentado como **Contributing**.
- O botão `↑ Topo` volta ao início real da página, já que a navegação operacional de resultados permanece sticky durante a rolagem.
- A faixa de contexto da análise foi simplificada para `Captura: <job_id> · Modo: <modo> · Saúde: <saúde>`; em visualização histórica de Sinalização, o modo identifica explicitamente que deriva de snapshot Completo.
- A estrutura conceitual fica consolidada: áreas operacionais somente em `Executivo / Visão Macro / Chamadas / Topologia / Diagnóstico`; `Metodologia` em `Projeto`; `Glossário` em `Ajuda`. O container técnico de Metodologia também deixa a área estrutural do workspace operacional.
- Cache-busting de `app.js` e `vca.css` atualizado para `20260921-2620`.

[Fixed]
- Eliminada a duplicação visual `Captura atual + Job + Modo original + Visualização` na faixa compacta de contexto.
- Corrigido o comportamento do botão `↑ Topo`, que ainda retornava apenas à região das abas após elas se tornarem sticky.

[To do]
- Obter/homologar captura real com RTCP XR PT=207 / VoIP Metrics e MOS/R-Factor reportados e reexecutar o Gate Core VoIP.
