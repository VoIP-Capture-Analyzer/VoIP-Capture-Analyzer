# Changelog

## v2.71.0 — Report & UI Presentation Refinement

### [Added]
- User-Agent observado por perna SIP na Análise Micro da interface, PDF/HTML Técnico e DOCX, com resumo por chamada e coluna adicional no CSV.
- Tempo relativo ao início da chamada no gráfico **Fluxo SIP + RTP**, mantendo a evidência temporal original no snapshot.
- Coluna **Sentido / direção** na timeline correlacionada, exibindo `IP:porta → IP:porta` para eventos SIP/RTP quando a evidência de endpoint está disponível.
- Requisito rastreável `VCA-REPORT-003` para consistência de apresentação da Análise Micro.

### [Changed]
- Tipografia de corpo/tabelas dos relatórios passa a usar tokens centralizados e consistentes; tabelas deixam de reduzir fonte individualmente para acomodar conteúdo.
- A interface Micro passa a reutilizar a mesma estrutura de pernas, User-Agent, fluxo temporal e direção da timeline utilizada pelos relatórios técnicos.
- `User-Agent` e `Server` SIP passam a ser preservados separadamente no parser para evitar misturar evidências de UAC/UAS.

### [Fixed]
- Timelines técnicas deixam de exigir interpretação manual de colunas separadas de origem/destino: a direção completa passa a ser exibida em uma única coluna sem remover os campos estruturados.
- Snapshots históricos sem `user_agents` por perna mantêm fallback de apresentação a partir dos User-Agent observados nas mensagens, sem fabricar associação por perna.

### [To do]
- **SDP Offer/Answer Deep Analysis**: reconstruir ofertas/respostas e reofertas, comparar codecs/atributos/endpoints e somente afirmar incompatibilidade quando a evidência demonstrar ausência de convergência.
- Em paralelo ao roadmap SIP, obter/homologar captura real com RTCP XR PT=207 / VoIP Metrics e MOS/R-Factor reportados e reexecutar o Gate Core VoIP.
