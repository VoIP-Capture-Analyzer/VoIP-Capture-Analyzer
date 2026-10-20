# Changelog

## v2.73.0 — SIP NAT / Routing / Connectivity Semantics

### [Added]
- Nova engine `app/analyzers/sip_connectivity.py` para correlacionar `source IP:porta`, `Via sent-by`, `received`, `rport`, `Contact`, `Route`, `Record-Route` e `Max-Forwards` como evidência topológica SIP.
- Confirmação conservadora de retorno `received/rport` por transação, distinguindo evidência compatível de NAT traversal de inconsistência observada.
- Detecção explícita de `483 Too Many Hops` e candidato de loop apenas quando a mesma tentativa reaparece no mesmo par origem/destino com `Max-Forwards` decrescente.
- Contexto ICE extraído do SDP, incluindo candidatos `host`, `srflx` e `relay`; `srflx` é tratado como evidência compatível com STUN e `relay` como evidência compatível com TURN, sem presumir sucesso ou causa raiz.
- Requisito rastreável `VCA-SIP-006`, novas colunas CSV e superfícies Macro/Micro/JSON/PDF/DOCX/Executivo para NAT/roteamento/conectividade.
- Documentação moderna do projeto: Datasheet e Guia de Administração em DOCX/PDF, versionados dentro de `docs/`.

### [Changed]
- Parser/fallback SIP passa a preservar múltiplos `Via`, `Route`, `Record-Route` e `Max-Forwards` quando disponíveis, mantendo o payload bruto como fonte de enriquecimento quando o TShark não expõe todos os campos nativamente.
- Inventário de protocolos reconhece também `stun` e `turnchannel` quando o TShark os identificar no `frame.protocols`; a ausência desses protocolos permanece N/A/0 e não impede evidência ICE via SDP.
- Backlog/roadmap canônico foi atualizado para refletir a conclusão de SIP Transaction Semantics, Dialog/Forking, Presentation Refinement, SDP Offer/Answer e SIP NAT/Routing, mantendo RTCP XR/MOS como trilha paralela do Core Gate.

### [Fixed]
- Corrigida a sobreposição visual entre as colunas **Estado** e **Status** em `5. Inventário consolidado de chamadas` no PDF/HTML Técnico, alinhando `colgroup` e CSS e ampliando a largura efetiva de Estado.
- O DOCX também passa a reservar mais largura para Estado e permite quebra segura nessa coluna, evitando `COMPLETED200`/`CANCELLED487` em renderizações compactas.

### [To do]
- **Transaction-aware Retransmissions**: distinguir retransmissão SIP compatível com timers/protocolo, resposta atrasada, timeout, duplicação de captura e perda provável, preservando evidência por transação e evitando classificar repetição esperada como falha.
- Em paralelo ao roadmap SIP, obter/homologar captura real com RTCP XR PT=207 / VoIP Metrics e MOS/R-Factor reportados e reexecutar o Gate Core VoIP.
