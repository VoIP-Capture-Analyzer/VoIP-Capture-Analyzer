## Fluxos operacionais e testes da v2.65.0

- `run.py` pode carregar `totp-secrets.env` somente por parser de whitelist; não introduza `source`, `eval` ou execução de shell para ler credenciais. Variáveis do ambiente devem continuar tendo precedência.
- Alterações em RBAC devem testar explicitamente `viewer`, `analyst` e `admin`; `/api/runtime/reset` faz parte do fluxo do Analyst, enquanto `/api/database/*` e manutenção permanecem Admin.
- Metadados de ator no Histórico não podem conter TOTP, token, `sub`, cookie ou segredo; TOTP compartilhado identifica apenas o papel.
- Mudanças em export/import SQLite devem cobrir consulta global ao registro de subprocessos e ambos os modos (`incremental`, `replace`).
- Conteúdo de governança (`Registro de requisitos VCA`) pertence a Projeto → Metodologia; relatórios exportados devem priorizar evidências, métricas, proveniência e reprodutibilidade do snapshot.

## Security & Reporting Hardening da v2.57.0

- O CSS utilitário do dashboard/glossário é gerado em `app/static/tailwind.css` e deve permanecer versionado. O runtime Python não requer Node.
- Ao adicionar/remover classes Tailwind em `index.html`, `glossary.html` ou `app.js`, regenere o bundle em ambiente de desenvolvimento com Tailwind CSS 4.1.10: `npm install` e `npm run build:css`.
- Não reintroduza `cdn.tailwindcss.com`, `unsafe-eval` ou compilação CSS no navegador. Alterações de CSP devem manter testes do Security Gate.
- Chamadas saudáveis não devem aparecer como “prioritárias” no Executivo; elas permanecem no inventário técnico completo.
- Nomes físicos de PDF devem permanecer derivados do `job_id`; nomes expostos ao navegador/download são responsabilidade das rotas HTTP e não devem alterar a rastreabilidade interna.

## Navigation & Context da v2.56.1

- Alterações de topologia informadas pelo operador devem permanecer em `data.operator_context.topology`; nunca promova alias/observação a evidência analítica.
- Testes de UI devem cobrir expansão inline da análise micro na guia Executivo, persistência runtime/histórico de contexto de topologia e títulos amigáveis dos visualizadores PDF.
- Helpers de codec devem distinguir payload RTP observado de negociação SDP; payload dinâmico sem evidência explícita deve permanecer indeterminado.
- Ao alterar contexto manual que afeta relatórios, valide a regeneração do PDF Técnico, PDF Executivo e DOCX.

## Reporting/UX da v2.55.0

Mudanças de apresentação devem preservar o snapshot analítico e nunca transformar contexto visual (aliases, ordenação, filtros) em evidência diagnóstica. PDF Executivo e PDF Técnico derivam do mesmo snapshot.

## Segurança da v2.54.1

Mudanças em autenticação, sessão, RBAC, CSRF, TLS/reverse proxy, headers, uploads ou WebSocket devem manter testes automatizados e o Security Gate. Não registre tokens, cookies, client secrets ou credenciais em logs/fixtures. O registro metodológico VCA permanece separado dos controles de segurança.

# Contribuindo para o VoIP Capture Analyzer

Obrigado por contribuir com o VoIP Capture Analyzer. O objetivo deste documento é manter as contribuições técnicas, a documentação e o licenciamento consistentes.

## Fluxos de relatório e paridade da v2.67.0

- Alterações em `report.html`, `report_executive.html`, `report.py` ou `report_csv.py` devem preservar a origem única no snapshot e a equivalência dos KPIs compartilhados.
- O DOCX Técnico deve manter paridade estrutural e visual com o PDF Técnico: mesma ordem de seções, mesmos gráficos/indicadores/tabelas e mesmo conteúdo técnico, tolerando apenas diferenças de paginação inerentes ao motor do Word/LibreOffice.
- Alterações no renderer DOCX exigem renderização visual do arquivo gerado (DOCX → PNG/PDF) antes da entrega, além dos testes automatizados.
- O PDF Executivo é uma projeção resumida; não deve introduzir métricas que não existam no Técnico/JSON nem recalcular valores.
- CSV permanece tabular por chamada, porém repete o contexto macro para ser auditável quando aberto isoladamente.
- O reset de runtime é uma limpeza de estado efêmero e pode ser executado por Viewer; qualquer exclusão de histórico, baseline, manutenção ou banco continua sujeita ao RBAC correspondente.
- Novas mudanças visuais devem ser cobertas por testes de contrato e, quando afetarem renderização, por um smoke test WeasyPrint/DOCX.


## Licenciamento e direitos autorais

1. O projeto é distribuído sob a Licença MIT. Contribuições aceitas serão disponibilizadas sob a mesma licença do projeto.
2. O autor da contribuição mantém os direitos autorais que lhe pertencem, mas, ao enviar uma contribuição para incorporação ao projeto, concede ao projeto uma licença perpétua, mundial, não exclusiva, gratuita e irrevogável para usar, reproduzir, modificar, sublicenciar e distribuir a contribuição como parte do projeto.
3. Não envie código, documentação, imagens ou outros materiais de terceiros sem verificar os direitos de uso e a compatibilidade da licença.
4. Código GPL ou outro material com licença incompatível não deve ser incorporado ao código-fonte do projeto. Dependências externas podem possuir licenças próprias e devem continuar sendo tratadas como componentes independentes.

## Reportando bugs e sugerindo melhorias

Antes de abrir uma Issue:

- pesquise se o problema já foi reportado;
- informe a versão do VoIP Capture Analyzer;
- informe a versão do Python e do TShark;
- descreva o comportamento esperado e o observado;
- quando possível, forneça um exemplo mínimo reproduzível e logs relevantes, removendo dados sensíveis da captura.

## Pull Requests

1. Faça um fork do repositório.
2. Crie uma branch específica para a alteração:

```bash
git checkout -b feature/minha-nova-funcionalidade
# ou
 git checkout -b fix/nome-do-bug
```

3. Mantenha o escopo da alteração claro e pequeno sempre que possível.
4. Para novos arquivos de código-fonte, utilize o cabeçalho de licença padrão:

```text
# Copyright (c) 2026 Bruno Barreto
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.
```

5. Execute os testes antes de abrir o PR. Mudanças no engine, persistência ou ciclo de jobs devem incluir testes unitários e de integração para todos os novos ramos:

```bash
pytest
```

6. Atualize o `README.md` quando a alteração mudar instalação, configuração ou uso.
7. Descreva no PR o que foi alterado, como foi validado e eventuais limitações conhecidas.

## Padrões de código

- Siga a PEP 8 para Python.
- Prefira código simples, explícito e testável.
- Adicione testes `pytest` para novas regras de análise e correções de regressões.
- Funcionalidades que dependem do modo de análise devem possuir regressões para **Completa** e **Sinalização**; no modo Sinalização, não invente métricas/achados de mídia.
- Preserve a política de **um job ativo por instância**: mudanças em upload, cancelamento, reset ou WebSocket devem manter a exclusão mútua no backend. `client_id`/`tab_id` controlam quem acompanha o job; clientes diferentes só podem consumir o status global genérico, sem metadados do job alheio.
- Preserve o **engine analítico unificado**: tamanho/complexidade da captura pode alterar apenas `Resource Profile`, buffers, limites e outras políticas operacionais; nunca mantenha pipelines pequenos/grandes com parsing, correlação, métricas ou diagnóstico distintos.
- Preserve a **semântica calibrada de RTP**: bitrate/PPS da chamada lógica representam tráfego agregado dos frames RTP correlacionados no intervalo observado; bitrate/PPS por stream só são avaliáveis com pelo menos dois pacotes e duração positiva. Nunca transforme stream de pacote único em taxa usando duração artificial.
- Jitter RTP deve continuar baseado no interarrival da RFC 3550 apenas quando o clock do payload de áudio for conhecido/sustentado. Não use event payloads ou ausência de clock para fabricar jitter.
- Mudanças em inventário/Decode-As/correlação RTP devem preservar e testar a distinção entre `inventário nativo`, `candidatos`, `estruturalmente válidos`, `associados` e `não associados`. Ambiguidade é evidência de não correlação, não permissão para escolher um Call-ID arbitrariamente.
- A rastreabilidade de RTP não associado deve permanecer **limitada em memória**: totais por motivo podem ser completos, mas o detalhamento por SSRC/fluxo deve respeitar `MAX_UNASSOCIATED_RTP_STREAMS` e nunca reter payloads RTP.
- **Exportação forense incremental:** o inventário completo usado por CSV/JSON deve permanecer agregado por stream e persistido/iterado em lotes. Não materialize todos os pacotes nem todos os streams em listas de RAM; preserve o spool temporário em disco, a exclusão no `finally` e o cascade com a exclusão do histórico.
- Filtros forenses por intervalo usam segundos relativos ao início da captura e somente incluem streams com evidência temporal suficiente para provar sobreposição. Motivo de não correlação descreve a falha de associação conservadora, nunca a causa raiz da rede.
- A régua visual de etapas deve usar o `stage` canônico do backend; nunca derive a etapa atual por limiares percentuais. `Merge` deve existir somente em Multi-PCAP e etapas de mídia somente no modo Completa.
- O **preflight** deve separar requisitos obrigatórios, requisito condicional de Multi-PCAP e recomendações. Falha de `mergecap` não pode bloquear captura única; CPU/RAM recomendadas não devem virar bloqueio rígido sem decisão explícita de arquitetura. Testes devem mockar comandos/recursos do host, sem depender do ambiente da máquina de CI.
- A **matriz de compatibilidade TShark** deve usar os campos registrados pelo executável como evidência funcional. A versão é informação de auditoria, não atalho para presumir suporte. Ausência de RTP bloqueia somente Completa; ausência de RTCP/RTCP XR deve degradar métricas para `N/A` sem bloquear recursos independentes.
- Alterações em `available_fields()` devem testar o formato real de `tshark -G fields`: para registros `F`, a abreviação aceita por `-e` é distinta do nome descritivo. Preserve fallback defensivo apenas para formatos não padrão.
- O projeto não mantém ETA de conclusão. Mudanças de telemetria devem expor somente estado observado, percentual, início, tempo decorrido e tempo na etapa.
- Call Anomaly Score exige evidência suficiente. Novas regras não podem classificar ausência de dados como `EXCELENTE`; estados não avaliáveis devem permanecer `N/A`/`INDETERMINADA` e ser excluídos das estatísticas de score.
- Correlação RTP/RTCP por perna ou chamada deve ser conservadora e baseada em evidências determinísticas (SSRC/endpoints/portas/janela temporal); em empate/ambiguidade, preserve o dado como não associado em vez de forçar uma atribuição.
- Alterações em regeneração de relatórios devem preservar a data da análise original e registrar separadamente o momento da regeneração.
- Toda alteração em política de recursos deve possuir teste de paridade demonstrando que a mesma entrada produz a mesma semântica analítica independentemente do perfil operacional.
- Preserve a separação entre **runtime** e **histórico persistente**: abrir, filtrar, validar ou excluir um snapshot histórico não pode criar, restaurar, substituir ou cancelar um `Job` ativo no `JobManager`.
- Não trate uma métrica como fato quando ela for apenas uma inferência.
- Preserve a separação entre evidência observada, métrica calculada e diagnóstico.
- Mantenha o TShark como ferramenta externa; o projeto não incorpora código-fonte do TShark.


## Regressões de calibração

Mudanças que alterem parsing RTP, jitter, bitrate/PPS, Decode-As, correlação por SDP/SSRC ou agregação de mídia devem atualizar e executar as regressões de calibração em `tests/fixtures/reference_calibration_v243.json`. O fixture representa os valores de aceitação dos três cenários-base e **não** autoriza código especial por captura.

Regras obrigatórias:

- `2001_2002_001`: proteger inventário/validação RTP, os 6 frames ambíguos, 5 SSRCs e o bitrate agregado aproximado de 333,68 kbps; o SSRC de pacote único não pode contaminar a média de taxas por stream.
- `9392-to-9999_002`: proteger ausência real de RTP/RTCP, PDD T1/T2/T3 e classificação `NO_RTP`; não inventar perda/jitter.
- Multi-PCAP de referência: proteger os KPIs de aceitação registrados no fixture e a diferença entre inventário nativo, candidatos processados e RTP não associado.
- Se uma alteração intencional mudar um valor de aceitação, documente a justificativa técnica e a evidência de referência no PR/CHANGELOG antes de atualizar o fixture.


### Proveniência e pré-visualização forense

- Proveniência de ambiente deve ser **host-safe**: persista apenas versões, prontidão e capacidades técnicas necessárias para reprodutibilidade. Não inclua IP, hostname, usuário, caminhos, variáveis de ambiente arbitrárias, CPU/RAM ou identificadores do cliente.
- Hashes de captura usados em reprodutibilidade devem ser calculados incrementalmente durante o upload ou por leitura em chunks; nunca carregue o PCAP inteiro em memória. Não retenha a captura apenas para manter o manifesto.
- O manifesto de reprodutibilidade deve continuar exportável a partir do snapshot original e omitir dados sensíveis do host. Mudanças no manifesto exigem versionamento de schema e regressões para snapshots legados.
- Comparação de drift deve separar diferenças de ambiente/capacidades de diferenças de conteúdo das capturas. Mesmo quando houver drift, não atribua causalidade à mudança sem evidência adicional.
- Drift analítico só pode ser calculado quando os manifestos possuem SHA-256 completos/iguais e o modo de análise é o mesmo. Compare KPIs em precisão documentada e achados por código estável; não use mudança de texto editorial como identidade primária do achado.
- A resposta de drift analítico deve manter causalidade como não avaliada: diferença de versão/TShark e diferença de KPI podem coexistir sem que uma seja declarada causa da outra.
- A proveniência exibida em relatório regenerado deve ser a do momento da análise, nunca o preflight atual do host que está regenerando o documento.
- A pré-visualização do inventário forense deve usar paginação determinística por cursor/seek e consultar no máximo `limit + 1` linhas por página. Não implemente `OFFSET` crescente nem carregue todo o inventário para o frontend.
- Totais apresentados na pré-visualização devem ser recalculados com os mesmos filtros de motivo/SSRC/intervalo usados na página e na exportação.
- O drill-down de contexto forense pode ranquear chamadas próximas por janela temporal e endpoint/porta, mas nunca deve promover candidato a associação real. `association` permanece `N/A`/`null` até o engine de correlação possuir evidência suficiente.
- Atalhos de navegação forense podem abrir a análise micro/linha do tempo de um candidato, mas devem operar apenas no snapshot já carregado e nunca alterar `association`, `call_id` ou a correlação persistida do stream não associado.
- Alterações nesses contratos devem incluir testes de cursor inválido, limites de página, filtros, totais, migração de histórico e ausência de dados sensíveis na proveniência.

## Exportação forense incremental e compatibilidade

Mudanças em `rtp_forensics.py`, `forensic_export.py`, `persistence.py` ou nos endpoints `/forensics/rtp-unassociated` devem cobrir, no mínimo:

- agregação do mesmo SSRC/5-tuple através de múltiplos flushes do spool;
- persistência em lotes e exclusão em cascade com o histórico;
- filtros por motivo (inclusive motivo não primário), SSRC e sobreposição temporal;
- CSV e JSON válidos gerados como iteradores, sem `list()` do inventário completo;
- ausência de payload bruto nas tabelas forenses;
- autorização por `client_id`/`tab_id` para exportação a partir do runtime.

Mudanças em preflight/campos TShark devem testar uma matriz completa, a ausência de RTP sem bloquear Sinalização, a ausência de RTCP/RTCP XR sem bloquear Completa e a falha do probe de campos sem fabricar uma incompatibilidade que não foi observada.

## Segurança e dados de captura

Capturas SIP/RTP podem conter números, endereços IP, URIs, cabeçalhos, áudio e outras informações sensíveis. Não publique capturas reais em Issues ou Pull Requests sem autorização e sem aplicar a anonimização necessária.


### Padrão de identidade de diálogo e SIP Forking (v2.70)
- Preserve a separação entre **chamada lógica/B2BUA**, **Call-ID físico**, **diálogo canônico** e **branch de forking**; nenhuma heurística B2BUA substitui `Call-ID + tags` quando a identidade normativa está disponível.
- Desafios `401/407` não materializam branches de forking mesmo quando carregam `To-tag`.
- Não force uma branch vencedora quando houver múltiplos `2xx`; represente a ambiguidade observada.
- `CANCEL` só pode ser atribuído a uma branch por evidência de tag ou por correlação conservadora da mesma tentativa (`CSeq`) acompanhada de `487`.
- Forking é estrutura SIP, não anomalia automática; mudanças nessa engine devem manter ASR, simultaneidade e Call Anomaly Score semanticamente invariantes e incluir testes de regressão.

### Preparação do ambiente


## Governança de release e gestão de estado

A continuidade entre versões é regida pela seção **“Diretriz de Governança e Gestão de Estado (ROADMAP)”** de `docs/VCA_PROJETO_CONCEITO_EVOLUCAO_ROADMAP.md`. Em contribuições de release:

- trate o último `[To do]` publicado como escopo ativo da próxima versão, salvo sobremarcação explícita do usuário para a entrega atual;
- em modo consulta, planeje e aguarde autorização antes de alterar código/artefatos; em modo execução autorizada, implemente o escopo ativo imediatamente;
- preserve o próximo `[To do]` no `CHANGELOG.md` e não promova itens do backlog para concluídos sem evidência/testes correspondentes;
- aplique documentação de forma condicional: README somente para mudanças de instalação/configuração/uso, CONTRIBUTING para mudanças de padrões de engenharia/testes, CHANGELOG em toda release e Glossário somente para novos termos técnicos;
- execute refatoração, revisão de performance, tratamento de erros/logs, tipagem e regressões pertinentes antes de empacotar a release.


Use `python activate.py` para criar `.venv`, instalar as dependências e, em Linux/macOS, abrir um shell já ativado. Para sair do ambiente, use `exit` nesse subshell ou `python deactivate.py` para consultar o comando apropriado.

### Padrão de tabelas de relatório
- Em PDF/HTML e DOCX, cabeçalhos devem permanecer em uma linha sempre que a largura semântica da coluna comportar o texto; quando a quebra for necessária, ela deve ocorrer em limites naturais de palavras, nunca caractere a caractere.
- Tabelas largas devem usar proporções determinísticas dentro da largura útil A4 (182 mm no DOCX técnico) e priorizar colunas de identificação/texto sem desperdiçar espaço em colunas numéricas curtas.
- Alterações de layout devem incluir regressão automatizada e validação visual renderizada em PDF/DOCX.

## Drift por chamada, baseline e UI contextual (v2.48)

- O drift por chamada só pode parear snapshots por **identidade canônica exata** (`Call-ID` + conjunto das pernas correlacionadas). Sobreposição parcial, identidade ausente/duplicada ou mudança de correlação deve produzir estado indeterminado/N/A; é proibido escolher o candidato "mais próximo" para fabricar pareamento.
- Mudança de estado, PDD T1/T2/T3, RTP (pacotes/streams/perda/jitter/one-way), Call Anomaly Score/classificação e códigos de achado precisa ser coberta por testes quando o comparador for alterado.
- Exportações de drift devem permanecer determinísticas e válidas; CSV/JSON não alteram os snapshots comparados e devem preservar a declaração `causality=NOT_ASSESSED` no payload JSON.
- **Baseline de aceitação** é metadado de governança em tabela separada. Testes devem provar que marcar/remover baseline não modifica `analyses.report_json`, exige manifesto completo e respeita `ON DELETE CASCADE`.
- Filtros do histórico devem usar aliases SQL explícitos; não alterar SQL por substituição textual de nomes de colunas.
- Detalhes de chamada abertos a partir da lista ou da investigação forense devem preservar o contexto visual. Reutilize o renderer de micro/timeline em vez de criar cópias divergentes.
- Zoom interativo de gráficos é responsabilidade exclusiva da interface web. Não adicionar scripts/macros ou comportamento dependente de um leitor específico ao PDF/DOCX.


## Gate de baseline e registro de requisitos (v2.49)

- O gate de aceitação só pode produzir `PASS/WARN/FAIL` depois que `compare_analytical_results` confirmar SHA-256 completos/iguais e mesmo modo. Caso contrário, use `NOT_COMPARABLE`; nunca tente aproximar capturas por nome, tamanho ou horário.
- A política de tolerâncias é metadado do **baseline**, deve ser validada/normalizada e persistida separadamente de `analyses.report_json`. Alterar a política não pode reescrever o snapshot técnico.
- `PASS` não aprova nem promove automaticamente a execução a baseline. `WARN`/`FAIL` não provam causa; preserve `causality=NOT_ASSESSED`.
- Tolerâncias default são `VCA-OPS`: ao mudar um threshold, atualize testes e documentação e não atribua o valor a RFC/ITU-T se a norma não o definir como limite universal.
- O registro `requirements_registry.py` é a fonte machine-readable dos IDs estáveis. IDs publicados (`VCA-SIP-*`, `VCA-SDP-*`, `VCA-RTP-*`, `VCA-RTCP-*`, `VCA-DIAG-*`, `VCA-REPRO-*`, `VCA-OPS-*`) não devem ser renumerados por conveniência.
- Todo requisito deve possuir `specification_ref`, ao menos um arquivo de implementação, ao menos um teste e ao menos uma superfície de relatório/dashboard. Vínculos inexistentes devem falhar nos testes.
- Referências normativas devem apontar para fontes oficiais e conter somente metadados/uma interpretação VCA concisa; não copie textos integrais de RFC/ITU-T/IEEE/3GPP para o código.
- Ao alterar comportamento coberto por requisito registrado, inclua o ID correspondente no teste/CHANGELOG ou atualize o próprio registro quando a especificação realmente mudar.



## Fechamento dos gaps implementáveis do Core VoIP (v2.52)

- `Reason` deve permanecer disponível em forma bruta e estruturada. O parser pode expor protocolo/cause/text, mas Q.850/SIP Reason é evidência do protocolo e não autorização para declarar causa raiz.
- Direção SDP somente deve ser marcada quando `sendrecv`, `sendonly`, `recvonly` ou `inactive` estiver explicitamente disponível na evidência extraída. Diagnósticos que cruzem direção SDP e RTP devem ser contextuais e testados.
- Early media deve permanecer semanticamente separada de atraso pós-atendimento. RTP pré-200 OK não gera `RTP_START_DELAY`; se a chamada for atendida e houver somente mídia pré-200, o Analyzer deve apontar ausência de RTP pós-atendimento.
- O botão “abrir análise micro” da Linha do tempo de Diagnóstico deve alternar conteúdo inline pré-renderizado. Não reintroduza `showTab('calls')`, `navigateToCall()` ou dependência equivalente nesse listener.
- Alterações nesses três temas devem manter os requisitos `VCA-SIP-004`, `VCA-SDP-002` e `VCA-RTP-004`, seus testes e as superfícies de relatório sincronizados.
- RTCP XR e MOS/R-Factor só podem deixar `PARTIAL` após captura real compatível exercitar extração, associação e apresentação ponta a ponta.

## Fechamento conceitual e continuidade da matriz (v2.51)

- O `core_voip_closure_gate()` é um gate de **maturidade conceitual**, não um sinônimo de suíte verde. Um tema só deve permanecer `IMPLEMENTED` quando seus vínculos de requisito/teste existirem; lacunas conhecidas devem ser declaradas como `PARTIAL`/`PENDING`, nunca ocultadas para obter `READY_TO_CLOSE`.
- As evidências das três capturas-base são referências homologadas do projeto. Amostras externas, inclusive Wireshark SampleCaptures, são suplementares até decisão explícita de governança.
- Toda release deve manter um snapshot da matriz publicada imediatamente anterior em `requirements_history.py`. A comparação precisa diferenciar mudança apenas de engenharia de alteração semântica/metodológica/normativa.
- Qualquer `METHODOLOGY_CHANGED` ou `NORMATIVE_CHANGED` em requisito existente exige reconhecimento explícito no contexto da release. O texto deve explicar a decisão; campo vazio ou ID desconhecido é falha de gate.
- Mudanças de interface que prometem preservar contexto de investigação devem possuir teste de contrato evitando regressão para navegação que troque de aba/página.
- O título interno do gráfico de fluxo por chamada deve permanecer contextual (`Fluxo SIP + RTP`/`Fluxo SIP`); o Call-ID identifica a chamada sem repetir “chamada N” dentro do gráfico.

## Auditoria, rastreabilidade de release e validação suplementar (v2.50)

- Cada acionamento explícito do **Gate de baseline** é um evento de auditoria append-only. Não atualize nem reutilize uma linha anterior para refletir uma nova execução; política, resultado e fingerprints daquele momento precisam permanecer auditáveis.
- O gate de rastreabilidade de release deve permanecer `PASS`: todo requisito VCA precisa apontar para especificação, implementação, testes e arquivos que materializam sua superfície de relatório/dashboard. Caminhos removidos/renomeados exigem atualização coordenada do registro.
- Se uma mudança alterar metodologia ou referência normativa, declare explicitamente os IDs VCA impactados. Não esconda mudança conceitual apenas como alteração de código.
- Exportações JSON/CSV de requisitos e auditoria são derivadas e somente leitura; não devem modificar snapshots, baselines ou o registro.
- O gráfico **Fluxo SIP + RTP** da Análise Micro deve ser produzido para cada chamada incluída no relatório e não pode ser filtrado por Call Anomaly Score. Para controlar volume, represente RTP de forma agregada por direção em vez de desenhar cada pacote.
- As três capturas-base do projeto continuam sendo os gates semânticos obrigatórios. O repositório oficial de exemplos do Wireshark em `https://wiki.wireshark.org/SampleCaptures` pode ser usado **somente como validação suplementar** quando houver um protocolo/caso que as capturas-base não cubram. Não embuta amostras externas na release e não substitua os baselines aceitos por esses arquivos sem decisão explícita de governança.


## Engenharia operacional v2.53

- Telemetria de performance deve permanecer estritamente operacional: não pode alterar KPIs, `Call Anomaly Score`, diagnóstico, associação RTP ou resultados dos gates semânticos.
- Novas etapas devem usar o coletor de telemetria por transição; evite loops de sampling em alta frequência que aumentem custo em capturas grandes.
- Políticas de retenção automática devem preservar `analysis_acceptance_baselines` por padrão e depender de exclusão em cascade somente para análises efetivamente selecionadas pela política.
- `HISTORY_RETENTION_DAYS=0` e `HISTORY_MAX_ANALYSES=0` significam ausência daquele limite; não interprete zero como “apagar tudo”.
- Limpeza de runtime por idade nunca deve tocar `.gitkeep`, arquivos estáticos ou artefatos fora dos diretórios runtime.
- Filtros de histórico devem continuar parametrizados em SQL. Filtros de drift são apenas de visualização/exportação: não recalculam comparabilidade, severidade nem causalidade.
- Mudanças nesses contratos devem incluir regressões para preservação de baseline, filtros numéricos, cleanup stale e telemetria descritiva.



### Gerenciamento do SQLite / backups protegidos

Mudanças em `app/services/database_backup.py`, autenticação TOTP ou `/api/database/*` devem permanecer fail-closed. Exportação/importação exige sessão `admin`, CSRF e reconfirmação TOTP Admin; `TOTP_ADMIN_SECRET` fica somente no servidor e serve como raiz de derivação da proteção do `.vcadb`. Nunca registre segredo Base32, código TOTP, chave derivada, plaintext do backup ou conteúdo sensível do histórico em logs/fixtures. Testes devem cobrir RBAC, código inválido/expirado, rate limit, roundtrip criptográfico, schema malicioso e rollback.

O formato `.vcadb` é um contêiner criptografado, não um script SQL. Importações devem continuar validando integridade SQLite, chaves estrangeiras, schema, ausência de triggers/views/tabelas virtuais inesperadas e JSON persistido antes de escrever na base. O modo incremental não pode sobrescrever Job IDs existentes; o modo de substituição precisa preservar rollback transacional/recuperável. Toda alteração nesses contratos deve cobrir token incorreto, arquivo adulterado, schema incompatível, colisão incremental e substituição completa em testes.

### Gate de segurança

Alterações de autenticação/autorização devem exercitar no mínimo: login OIDC, PKCE, state/nonce, validação de ID Token, sessão, RBAC, CSRF, headers, WebSocket, exportações protegidas e fail-closed de configuração. Segredos reais não entram em testes. Para revisão pré-release execute `pytest`, `python -m compileall -q app tests` e `node --check app/static/app.js`. O Security Gate é complementar ao pentest independente.


### Clean Installation Gate
Toda mudança em dependências/runtime deve executar `python scripts/check_runtime_dependencies.py` e, quando houver acesso a índice/wheelhouse, `bash scripts/clean_install_gate.sh`. O gate cria um venv vazio, reinstala `requirements.txt`, importa `app.main` e executa a suíte.



## Regressões de exportação/responsividade da v2.59.0

Ao alterar ações de relatório ou layout responsivo, preserve os contratos: PDF aberto inline deve anunciar nome com `job_id`; o menu **Outros formatos** deve oferecer DOCX/JSON/CSV; CSV deve ser gerado em streaming; `↑ Topo` deve apontar para a barra de abas; e controles essenciais devem permanecer utilizáveis em viewport estreito. Testes de contrato ficam em `tests/test_v259_responsive_exports_navigation.py`.

## Regressões de UX da v2.58.0

Mudanças de navegação/visibilidade devem preservar três contratos: controles recolhidos não podem perder os valores persistidos; nomes de download devem ser testados pelo `Content-Disposition` além do nome físico; filtros client-side (Glossário/forense) devem ser validados também por presença dos seletores/handlers no bundle JavaScript. Ao adicionar classes Tailwind em templates ou `app.js`, regenere `app/static/tailwind.css` com `npm run build:css` antes da release.
## Regressões de histórico da v2.60.0

Mudanças no Histórico devem preservar estes contratos:

- snapshot original Completo pode ser visualizado como Completa ou Sinalização sem mutação do `report_json`;
- snapshot original Sinalização nunca oferece Completa;
- visualização Sinalização oculta componentes exclusivos de RTP/RTCP e filtra eventos de mídia da timeline;
- `Gate baseline`, `Marcar/Remover baseline` e `Política` ficam em **Outras ações**;
- **Excluir** permanece visível fora do menu;
- o selo de baseline continua visível;
- nenhuma dessas mudanças altera a matriz metodológica/normativa nem o Gate Core VoIP.



## Regressões de visualização histórica da v2.60.2

Ao alterar o fluxo de reabertura do Histórico, preserve estes contratos: normalize os modos persistidos e selecionados antes de renderizar; a visão Sinalização deve ser derivada somente de snapshot Completo e nunca mutar o snapshot; microanálise e achados devem respeitar o filtro de mídia; e toda mudança em `app.js`/`vca.css` deve atualizar a chave de cache em `app/templates/index.html`. Execute também `tests/test_v2602_history_view_selection.py` e `node --check app/static/app.js`.

### Organização de informação da interface (v2.61.0)

- A navegação principal de resultados deve permanecer focada em **Executivo, Visão Macro, Chamadas, Topologia e Diagnóstico**.
- **Metodologia** é conteúdo de projeto/auditoria e deve ser acessada pelo menu **Projeto**, sem voltar a ocupar uma aba operacional da análise.
- **Glossário** é conteúdo de apoio: deve permanecer em **Ajuda** (e página dedicada), sem duplicação na tela principal ou nos relatórios PDF/DOCX.
- Filtros técnicos de Histórico de uso menos frequente devem ficar sob **Filtros avançados**, preservando valores e contratos de API.
- Navegação sticky, badges e cards mobile são apresentação; não podem alterar snapshot, KPIs, score, diagnóstico ou persistência.

### Preparação de branding

Os assets em `app/static/branding-ready/` são alternativas **inativas** para eventual migração futura de nome. Não os referencie em templates, CSS, JavaScript, relatórios, login ou favicon sem uma decisão explícita de mudança de branding. A marca ativa permanece `VoIP Capture Analyzer`; testes de release devem garantir que os assets alternativos existam sem alterar o runtime atual.
