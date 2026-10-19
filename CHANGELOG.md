# Changelog

## v2.68.2

### [Added]
- Classificação direcional de early media (`BIDIRECTIONAL`, `RETURN_ONLY`, `OUTBOUND_ONLY`, `UNRESOLVED`, `NOT_OBSERVED`) baseada nos endpoints RTP anunciados pelo SDP das respostas provisórias `180/183`.
- Evidências estruturadas `early_media_return_path_observed`, `provisional_sdp_endpoints` e direção da mídia provisória nas análises Micro/relatórios.
- Testes de regressão para o cenário real `183` sem `180` com RTP apenas originador → destino antes do `200 OK`.

### [Changed]
- `SIP_PROGRESS_WITHOUT_RINGING` deixa de ser suprimido pela mera existência de RTP pré-atendimento: agora exige mídia de retorno observável para suprimir o achado.
- Análise Micro em UI, HTML/PDF e DOCX passa a expor a classificação direcional da early media e se houve fluxo de retorno.

### [Fixed]
- Corrigido falso negativo da v2.68.1 em cenários com `183 + SDP` e early media unidirecional somente no sentido originador → destino, nos quais o chamador pode permanecer sem ringback remoto.

### [To do]
- Obter/homologar captura real com RTCP XR PT=207 / VoIP Metrics e MOS/R-Factor reportados e reexecutar o Gate Core VoIP.
