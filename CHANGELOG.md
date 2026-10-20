# Changelog

## v2.72.0 — SDP Offer/Answer Deep Analysis

### [Added]
- Reconstrução conservadora de SDP Offer/Answer por `Call-ID` físico e transação (`INVITE`, re-INVITE, `UPDATE` e `PRACK`), incluindo late offer em `200 OK` com resposta SDP no `ACK` quando observável.
- Comparação estruturada de codecs, payloads, `rtpmap`, `fmtp` preservado como evidência, endpoints `c=/m=` e direção `sendrecv/sendonly/recvonly/inactive`.
- Estados explícitos de negociação (`CONVERGED`, `NO_COMMON_CODEC`, `ANSWER_CODEC_OUTSIDE_OFFER`, `DIRECTION_INCOMPATIBLE`, `INDETERMINATE_CODEC_MAPPING`, `REJECTED_WITHOUT_SDP_ANSWER`, `OFFER_WITHOUT_ANSWER`, `LATE_OFFER_UNANSWERED`).
- Requisito rastreável `VCA-SDP-003` e superfícies Macro/Micro/JSON/CSV/PDF/DOCX para Offer/Answer.

### [Changed]
- `488/606` passa a ser contextualizado com a reconstrução Offer/Answer: somente evidência comparável suficiente eleva a conclusão para incompatibilidade comprovada; rejeição isolada continua hipótese de negociação.
- Cards de **Chamada correlacionada** na timeline da interface passam a iniciar recolhidos; **abrir análise micro** expande o detalhe imediatamente abaixo do cabeçalho, independentemente da expansão dos eventos.
- Relatório Executivo recebe resumo compacto de Offer/Answer quando a análise possuir negociações materializadas.

### [Fixed]
- Tabelas DOCX removem flags residuais de `noWrap` antes de aplicar exceções explícitas, reduzindo risco de texto longo invadir a coluna adjacente em layout fixo.
- Tabelas HTML/PDF mantêm wrapping natural em células fixas e conteúdo longo sem truncamento lateral.
- A UI evita listas enormes de eventos abertas por padrão em capturas com muitos Call-IDs correlacionados.

### [To do]
- **SIP NAT / Routing / Connectivity Semantics**: correlacionar source IP/porta com `Via`, `received`, `rport`, `Contact`, `Route`, `Record-Route` e `Max-Forwards`, mantendo STUN/TURN/ICE como evidência contextual quando observável e sem atribuir automaticamente NAT/SIP ALG/SBC como causa raiz.
- Em paralelo ao roadmap SIP, obter/homologar captura real com RTCP XR PT=207 / VoIP Metrics e MOS/R-Factor reportados e reexecutar o Gate Core VoIP.
