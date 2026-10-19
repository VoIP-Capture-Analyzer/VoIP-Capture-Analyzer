# VoIP Capture Analyzer

> **v2.68.2 — Directional Early Media Diagnostics:** evolui o diagnóstico de progresso SIP para distinguir early media por direção usando os endpoints anunciados no SDP provisório. O VCA diferencia RTP de saída de RTP de retorno antes do `200 OK`, preserva o diagnóstico conservador de `183` sem `180` e mantém o Gate Core VoIP aguardando captura real com RTCP XR/MOS-R-Factor.


Framework web para análise técnica de capturas VoIP, com foco em **SIP, SDP, RTP e RTCP**, reconstrução de chamadas, correlação de pernas B2BUA, métricas de mídia e diagnóstico baseado em evidências.

## Principais recursos

- Upload de uma ou várias capturas `.cap`, `.pcap` ou `.pcapng`.
- Consolidação automática de múltiplas capturas com `mergecap` antes da análise.
- TShark como decodificador e leitor primário da captura consolidada.
- Recuperação de SIP/SDP/RTP/RTCP por caminhos alternativos baseados em dados extraídos pelo TShark.
- Correlação de chamadas com múltiplos `Call-ID`s, incluindo cenários B2BUA.
- Métricas de ASR, PDD (incluindo T1/T2/T3), duração, RTP, perda, jitter, bitrate e PPS quando os dados permitem cálculo confiável.
- Identificação de RTP unilateral, interrupções assimétricas e gaps de mídia.
- Linha do tempo correlacionada SIP + RTP.
- Fluxo gráfico SIP + RTP compacto para **cada chamada** da Análise Micro em PDF/DOCX, independente do Call Anomaly Score; RTP é resumido por direção para manter relatórios grandes controláveis.
- Gráficos técnicos e relatório em HTML, PDF e DOCX.
- Glossário técnico centralizado em **Ajuda**, sem duplicação na tela operacional nem nos relatórios exportados.
- Nome original das capturas preservado nos relatórios para facilitar auditoria e rastreabilidade.
- Limpeza automática dos arquivos de captura após o processamento.
- Limpeza dos relatórios e artefatos anteriores ao iniciar uma nova análise.
- Interface web com atualização de progresso em tempo real via WebSocket, incluindo estágio canônico, início, tempo decorrido e tempo na etapa. O ETA foi removido para evitar previsão instável.
- Menu de ajuda com instruções de uso, download da suíte atual, glossário, licença, contribuição e contato.
- Checklist de validação de infraestrutura orientado pelos achados da captura, sem afirmar causa raiz sem evidência externa.
- Registro de validações externas no checklist, com status, evidência e resultado, refletidos nos relatórios PDF e DOCX.
- Histórico/auditoria das alterações de validações externas durante a análise, preservando o estado anterior e o novo estado.
- Persistência local do histórico em SQLite, incluindo análises concluídas e estados terminais de erro/cancelamento; análises concluídas podem ser reabertas após reinicializações e regenerar PDF/DOCX sob demanda.
- **Autenticação TOTP por papel e SQLite protegido (v2.64):** `AUTH_MODE=totp` oferece acesso local `viewer`/`analyst`/`admin` sem usuário/senha, Google ou Microsoft. Projeto → Banco de dados exige sessão Admin e reconfirmação do TOTP Admin para exportar/importar `.vcadb`; o segredo Base32 permanece somente no servidor.
- Histórico avançado com filtros por período, status, modo e saúde, busca por nome/ID e resumo comparativo de chamadas, ASR, PDD, jitter, perda e score. O histórico é independente do `JobManager` e nunca recria um job ativo ao abrir um snapshot concluído.
- **Filtros avançados de histórico (v2.53):** baseline aceito, número mínimo de chamadas, PDD máximo, ASR mínimo, perda máxima e quantidade mínima de chamadas com RTP unilateral; o tempo total de processamento também é persistido para comparação operacional.
- **Telemetria operacional por etapa (v2.53):** registra wall time, CPU do processo e pico RSS quando suportado, além de tempos agregados de metadata/SIP/SDP/correlação/RTP/RTCP/métricas/diagnóstico/relatório. Essa telemetria é descritiva e nunca altera score, métricas ou diagnóstico VoIP.
- **Retenção segura (v2.53):** `HISTORY_RETENTION_DAYS` e `HISTORY_MAX_ANALYSES` podem limitar o histórico persistente; baselines aceitos são preservados da limpeza automática. `RUNTIME_RETENTION_HOURS` remove artefatos temporários antigos deixados por interrupções anteriores.
- **Engine analítico unificado:** qualquer captura usa o mesmo pipeline incremental de parsing, correlação, métricas e diagnóstico. `small`, `medium` e `large` são apenas perfis de recursos e não selecionam analisadores diferentes.
- **Preflight de ambiente:** a interface consulta `/api/preflight` e bloqueia a análise quando faltam Python compatível, TShark, WeasyPrint, escrita no diretório de dados ou espaço livre mínimo; `mergecap` é exigido somente para Multi-PCAP.
- **Matriz de compatibilidade TShark:** o preflight registra a versão informada pelo executável e consulta `tshark -G fields`; campos de leitura/SIP/SDP bloqueiam ambos os modos quando ausentes, campos RTP bloqueiam somente o modo Completa e campos RTCP/RTCP XR são opcionais, gerando limitação explícita sem bloquear funcionalidades independentes.
- **Exportação forense incremental:** o inventário completo de streams RTP estruturalmente válidos e não associados é agregado em spool SQLite temporário, persistido no histórico e exportado em CSV/JSON sem materializar todos os pacotes ou todos os streams em memória. A tabela exibida no relatório continua resumida e limitada.
- **Pré-visualização forense paginada:** antes do download, o inventário persistido pode ser consultado por motivo, SSRC e intervalo temporal, com totais correspondentes aos filtros e paginação por cursor/seek. Somente a página solicitada é materializada no frontend.
- **Proveniência reprodutível do ambiente:** o snapshot de cada análise registra versão do Analyzer, Python, TShark, mergecap/WeasyPrint quando disponíveis e a matriz de capacidades TShark observada no momento da admissão. Não são persistidos identidade, IP, hostname, caminhos, CPU/RAM ou outros dados sensíveis do host. O resumo é exibido no relatório e no histórico.
- **Manifesto de reprodutibilidade:** SHA-256 das capturas de entrada é calculado durante o upload em streaming e combinado com a proveniência técnica em um manifesto JSON exportável, sem retenção do PCAP após a limpeza do job.
- **Drift de ambiente:** duas análises históricas podem ter versões/capacidades comparadas sem confundir mudança de ambiente com causalidade sobre os resultados.
- **Drift analítico controlado:** KPIs, códigos de achado e candidatos diagnósticos só são comparados como drift quando as duas análises possuem os mesmos SHA-256 de entrada e o mesmo modo. Diferença observada e drift de ambiente são exibidos separadamente; coexistência não prova causalidade.
- **Filtros de drift (v2.53):** a visão/exportação pode restringir o drill-down por status da chamada e código de achado sem recalcular o resultado do drift nem alterar a declaração de causalidade.
- **Gate de baseline PASS/WARN/FAIL:** uma execução histórica pode ser comparada diretamente com o baseline aceito mais recente que possua o mesmo conjunto completo de SHA-256 e o mesmo modo. A política de tolerâncias por KPI/achado/identidade é configurável e permanece metadado de governança separado do snapshot técnico. Cada execução explícita é registrada em auditoria append-only e pode ser exportada em JSON/CSV.
- **Registro de requisitos VCA:** requisitos metodológicos/normativos recebem IDs estáveis (`VCA-SIP-*`, `VCA-SDP-*`, `VCA-RTP-*`, `VCA-RTCP-*`, `VCA-DIAG-*`, `VCA-REPRO-*`, `VCA-OPS-*`) e vinculam a Especificação Conceitual, Metodológica e Normativa à implementação, testes e superfícies de relatório/dashboard. A v2.51 acrescentou comparação entre a matriz publicada e a release anterior. A v2.52 amplia o registro para Reason/Q.850 estruturado, direção SDP explícita e early media separada da mídia pós-atendimento, mantendo reconhecimento explícito obrigatório para mudanças metodológicas/normativas de requisitos existentes.
- **Contexto forense RTP:** cada stream não associado pode ser investigado por janela temporal e chamadas candidatas próximas; candidatos permanecem hipótese/N/A e não alteram a correlação original. A partir do candidato é possível abrir diretamente a análise micro ou a linha do tempo da chamada no snapshot atual. Na Linha do tempo correlacionada, a ação “abrir análise micro” expande um detalhe já pré-renderizado no próprio card, sem depender de troca para a aba Chamadas e preservando a posição de investigação.
- **Rastreabilidade RTP não associada:** o engine registra motivos de não correlação de forma agregada e, com limite de memória, por SSRC/fluxo, sem reter payloads e sem transformar ambiguidade em associação forçada.
- **Semântica RTP calibrada:** bitrate no nível da chamada lógica representa tráfego agregado dos frames RTP correlacionados no intervalo observado; bitrate/PPS por stream só é calculado com ao menos dois pacotes e intervalo temporal mensurável. Jitter continua baseado no interarrival RFC 3550 quando o clock de áudio é conhecido.
- **Checklist explicável:** cada validação de infraestrutura indica o candidato diagnóstico que a priorizou, abrangência/quantidade de chamadas afetadas e permanece separada da evidência externa registrada pelo usuário.
- A régua visual de processamento é derivada do `stage` canônico enviado pelo backend; percentual não escolhe etapa. `Merge` só aparece quando duas ou mais capturas são processadas.
- Canal global `/ws/system-status` (com fallback de polling em `/api/system/status`) informa somente disponibilidade, etapa e progresso do slot, sem expor `job_id`, nome de captura, Call-ID ou outros detalhes de outro cliente.
- Call Anomaly Score só é calculado quando há evidência suficiente; chamadas `INCOMPLETE`, `UNKNOWN` ou `CANCELLED` sem desfecho avaliável ficam com score `N/A`/`INDETERMINADA` e não contaminam estatísticas de score nem correlação simultaneidade × qualidade.
- Relatórios distinguem **Análise executada em** de **Relatório regenerado em** quando PDF/DOCX é reconstruído a partir de snapshot persistente.
- No modo Completa, o relatório expõe PDD T1/T2/T3, streams RTP por SSRC, atribuição RTP por perna quando determinística e RTCP associado/utilizável. No modo Sinalização, métricas de mídia — inclusive interrupções RTP — permanecem `N/A`.
- Processamento de RTP e volumetria de captura em streaming para todos os tamanhos, evitando materializar saídas extensas do TShark em memória.
- Correlação RTP otimizada por índice de endpoints/portas e agregação incremental de métricas, com amostragem limitada para percentis e detalhes.
- Limite operacional independente do limite de upload, configurável por `MAX_ANALYSIS_PACKETS`, com classificação de complexidade para capturas grandes.
- Cancelamento seguro disponível pela interface e pelo endpoint do job, com encerramento propagado aos subprocessos TShark/mergecap e limpeza idempotente.
- Modos de análise selecionáveis antes do upload: **Completa** processa SIP/SDP/RTP/RTCP; **Sinalização** processa SIP/SDP e não calcula métricas de mídia.
- Política de **um job ativo por instância**: o backend mantém admissão atômica e HTTP 409. Um `client_id` persistente identifica o navegador e um `tab_id` identifica a guia; a mesma guia reconecta automaticamente, outra guia do mesmo cliente pode assumir explicitamente o acompanhamento e clientes diferentes recebem somente status/progresso genéricos até a liberação automática do slot.

### Modos de análise

- **Completa**: SIP + SDP + RTP + RTCP, incluindo métricas, correlação e diagnóstico de mídia.
- **Sinalização**: SIP + SDP; RTP/RTCP não são processados e perda, jitter, RTP unilateral e demais métricas de mídia permanecem `N/A`. REGISTER, OPTIONS e demais controles SIP continuam auditáveis.
- A ação **Nova análise** só é liberada após o job chegar a um estado terminal. Ela limpa o estado e os artefatos temporários do runtime sem apagar o histórico persistente.

### Governança de execução entre guias/clientes

- `client_id` é gerado no navegador e persistido em `localStorage`; `tab_id` é exclusivo da guia e fica em `sessionStorage`. IP não é usado como identidade principal.
- `GET /api/jobs/active?client_id=...&tab_id=...` informa a relação `same_tab`, `same_client` ou `other_client`. Dados do job só são retornados para `same_tab`/`same_client`.
- `POST /api/jobs/{job_id}/monitor/claim` transfere o acompanhamento entre guias do mesmo `client_id` sem reiniciar o processamento. A guia anterior entra em modo passivo.
- `GET /api/system/status` e `/ws/system-status` são deliberadamente genéricos. Clientes alheios não recebem nome de captura, `job_id`, Call-ID, IPs ou resultados parciais.
- Quando o slot volta a ficar disponível, a interface em espera é liberada automaticamente; polling é usado apenas como contingência se o WebSocket global falhar.

### Preflight do ambiente e compatibilidade TShark

- `GET /api/preflight` retorna `required_ok`, `signaling_ok`, `complete_ok`, `multi_pcap_ok`, checks obrigatórios/condicionais, recomendações de CPU/RAM e a matriz `compatibility`.
- A matriz registra a versão bruta/parseada do TShark para auditoria, mas **não presume compatibilidade apenas pelo número da versão**: a prontidão funcional é determinada pelos campos realmente registrados em `tshark -G fields`.
- Leitura base, SIP e SDP são obrigatórios para os dois modos. RTP é obrigatório somente para **Completa**. RTCP básico e RTCP XR são capacidades opcionais: quando ausentes, as métricas dependentes permanecem `N/A` e os demais recursos continuam disponíveis.
- Se Completa estiver indisponível mas Sinalização estiver compatível, a interface mantém Sinalização utilizável em vez de bloquear toda a ferramenta.
- Se `mergecap` falhar, captura única continua permitida e apenas Multi-PCAP é bloqueado.
- O backend repete a validação de prontidão do modo no `POST /api/analyze`, evitando bypass da proteção apenas por cliente manual.
- Se a consulta fina de campos falhar, o preflight informa que a matriz não pôde ser determinada; não inventa incompatibilidade sem evidência.
- Na interface, os detalhes começam recolhidos. Ao expandir, os checks são agrupados em **Requisitos obrigatórios**, **Dependências condicionais**, **Capacidade recomendada** e **Compatibilidade TShark por recurso**. CPU/RAM abaixo do recomendado aparecem como recomendação em âmbar, não como falha bloqueante em vermelho.

### Proveniência, manifesto e drift de ambiente

A v2.46 preserva `data.environment_provenance` e acrescenta `data.reproducibility_manifest` dentro do snapshot concluído. O objetivo é permitir auditoria e comparação reprodutível sem coletar detalhes sensíveis do host. O snapshot inclui:

- versão do Analyzer e modo/capturas da execução;
- versões de Python, TShark, mergecap e WeasyPrint quando disponíveis;
- quantidade de campos registrados pelo TShark e estado da matriz por recurso;
- prontidão de Completa, Sinalização e Multi-PCAP observada antes do processamento;
- timestamp do preflight usado para autorizar a análise.

Durante o upload, o SHA-256 de cada captura é calculado no mesmo fluxo de escrita em disco, sem carregar o arquivo inteiro em memória e sem exigir uma segunda leitura da entrada. O manifesto guarda nome original, tamanho e SHA-256 das entradas, além da proveniência host-safe. Em Multi-PCAP, o artefato produzido pelo `mergecap` continua efêmero; a identidade reprodutível é sustentada pelos hashes das capturas originais.

O histórico mantém apenas um resumo desnormalizado da proveniência para listagem eficiente. O snapshot completo permanece no `report_json` e é reutilizado quando PDF/DOCX são regenerados, sem recalcular o ambiente atual.

Endpoints de reprodutibilidade:

- `GET /api/jobs/{job_id}/reproducibility/manifest` exporta o manifesto JSON do job concluído ainda em runtime e exige a guia responsável;
- `GET /api/history/{job_id}/reproducibility/manifest` exporta o manifesto persistido; snapshots v2.45 podem produzir manifesto parcial/legado sem hash da captura;
- `GET /api/history/{job_id}/reproducibility/drift/{other_job_id}` compara versões, quantidade de campos e matriz de capacidades TShark entre duas análises históricas. Drift é evidência de mudança de ambiente e **não prova** que o resultado analítico tenha sido alterado.
- `GET /api/history/{job_id}/reproducibility/analytical-drift/{other_job_id}` compara resultados somente quando os hashes das entradas são completos/iguais e o modo de análise coincide. O retorno separa mudanças de KPI, achados por código, candidatos diagnósticos e drift de ambiente, sempre com causalidade não avaliada.

### Exportação forense de RTP não associado

A v2.46 preserva as camadas da v2.44/v2.45 e acrescenta contexto investigativo por stream:

- o relatório/dashboard mostra um **resumo limitado** por `MAX_UNASSOCIATED_RTP_STREAMS`, adequado para leitura humana;
- o inventário completo de streams não associados é agregado durante a análise em um spool temporário em disco e, após a conclusão, persistido no SQLite do histórico sem armazenar payload RTP.

Endpoints de exportação:

- `GET /api/jobs/{job_id}/forensics/rtp-unassociated/csv` e `/json` para o job concluído ainda em runtime; exigem `client_id`/`tab_id` da guia responsável;
- `GET /api/history/{job_id}/forensics/rtp-unassociated/csv` e `/json` para snapshots concluídos do histórico.

Filtros opcionais: `reason`, `ssrc`, `offset_from` e `offset_to`. Os offsets são segundos relativos ao início da captura e usam sobreposição do intervalo do stream. Streams sem referência temporal suficiente não são incluídos quando um filtro temporal é solicitado. Os motivos (`OUTSIDE_CALL_WINDOW`, `NO_MEDIA_ENDPOINT_MATCH`, `AMBIGUOUS_CALL_MATCH` etc.) explicam **por que a correlação conservadora falhou**; não são causa raiz da rede.

CSV e JSON são produzidos por iteradores/cursores em lotes. O JSON permanece um documento único válido, mas é transmitido progressivamente; o CSV envia cabeçalho e linhas incrementalmente.

A pré-visualização também oferece **Investigar** por stream. O endpoint `/forensics/rtp-unassociated/context` apresenta a janela temporal do SSRC/5-tuple e ranqueia chamadas próximas usando somente proximidade temporal e evidências de endpoint/porta já persistidas. Esses candidatos são contexto investigativo: a associação continua `N/A` e nenhum Call-ID é promovido automaticamente.
Na v2.47, cada candidato contextual oferece atalhos para **Análise micro** e, quando existente, **Linha do tempo**. A navegação apenas focaliza dados já pertencentes à chamada candidata no snapshot aberto; não modifica a associação do RTP não associado.

### Histórico avançado

- `GET /api/history` aceita `q`, `date_from`, `date_to`, `status`, `mode` e `health` e devolve `items` + `summary` comparativo.
- `GET /api/history/{job_id}` abre apenas snapshots concluídos persistidos; não cria nem restaura estado no `JobManager`.
- `GET /api/history/{job_id}/download/{kind}` regenera `pdf`/`docx` sob demanda sem depender de job de runtime.
- `PATCH /api/history/{job_id}/infrastructure-validation/{check_code}` registra validação externa diretamente no snapshot histórico.
- Estados `error` e `cancelled` são persistidos apenas como metadados terminais para filtragem/auditoria e não possuem relatório técnico para abertura.

## Requisitos mínimos do ambiente

Na interface, este painel inicia **recolhido** para reduzir ocupação visual. O status geral permanece visível e os detalhes da matriz de compatibilidade podem ser expandidos sob demanda; o botão **Revalidar** continua disponível sem iniciar uma análise.

A v2.45 executa um **preflight** antes de liberar uma nova análise. O painel da interface e `GET /api/preflight` separam requisitos obrigatórios, requisito condicional de Multi-PCAP e recomendações de capacidade. O preflight não usa CPU/RAM como bloqueio rígido porque a necessidade real varia conforme a captura.

### Obrigatórios para uma análise

- **Python 3.11 ou superior**.
- **TShark** instalado e disponível no `PATH`, ou informado por `TSHARK_BIN`.
- Dependências Python de `requirements.txt`, incluindo **WeasyPrint**, e suas bibliotecas de sistema quando exigidas pela plataforma.
- Permissão de escrita nos diretórios de upload/relatório e no caminho de `HISTORY_DB`.
- Espaço livre no volume de dados de pelo menos `MIN_FREE_DISK_MB` (padrão: **1024 MB**). Esse é um limiar operacional configurável, não uma estimativa do tamanho final de uma análise.

### Condicional para Multi-PCAP

- **mergecap** disponível no `PATH` ou em `MERGECAP_BIN`. A ausência de mergecap **não bloqueia uma captura única**, mas impede selecionar duas ou mais capturas até a correção do ambiente.

### Capacidade recomendada

- **2 núcleos de CPU ou mais** (`MIN_RECOMMENDED_CPU_CORES`).
- **4 GB de RAM ou mais** (`MIN_RECOMMENDED_RAM_GB`).

CPU e RAM são recomendações visíveis no preflight, não requisitos absolutos. Capturas longas, grande volume RTP, Decode-As extenso e relatórios muito grandes podem exigir recursos superiores.

## Execução local

### Instalação

A forma recomendada é usar o script de preparação do ambiente:

```bash
python activate.py
```

No Linux/macOS, o script cria `.venv`, instala as dependências e abre um novo shell com o ambiente já ativado. Use `exit` para sair desse shell e retornar ao ambiente anterior. Em PowerShell, o script informa o comando de ativação porque um processo Python não consegue alterar o ambiente do shell pai. Também é possível executar manualmente:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
tshark --version
mergecap --version
```

### Inicialização simplificada

```bash
python run.py
```

Por padrão, a aplicação escuta em `0.0.0.0:8000`.

A porta e o endereço podem ser escolhidos na inicialização:

```bash
python run.py --port 8080
```

```bash
python run.py --host 127.0.0.1 --port 8080
```

Também é possível utilizar as variáveis de ambiente:

```bash
HOST=0.0.0.0 PORT=8080 python run.py
```

A inicialização direta pelo Uvicorn continua disponível:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Acesse `http://IP:PORTA` pelo navegador.

## Upload de capturas

A interface permite selecionar ou arrastar uma ou mais capturas.

- **Uma captura:** o arquivo é analisado diretamente pelo engine unificado incremental.
- **Duas ou mais capturas:** os arquivos são enviados para um diretório temporário do job e consolidados pelo `mergecap`. A captura consolidada é então analisada pelo mesmo pipeline TShark/SIP/SDP/RTP/RTCP.
- Os nomes originais das capturas são preservados como metadados do job e apresentados no relatório.
- Os arquivos de captura temporários, inclusive o arquivo consolidado, são removidos após a conclusão, falha ou cancelamento do job.
- O dashboard disponibiliza **Cancelar análise** somente para a guia que detém o acompanhamento do job. O backend usa `POST /api/jobs/{job_id}/cancel` com `client_id`/`tab_id`; a tarefa propaga a interrupção aos subprocessos, preserva o percentual alcançado, finaliza com estado `cancelled` e executa limpeza idempotente.
- Ao iniciar **Nova análise**, os relatórios e gráficos da análise anterior também são removidos.
- Após a análise, é possível registrar validações externas no checklist de infraestrutura; essas informações são separadas das evidências observadas na captura, os relatórios são regenerados ao salvar e o histórico das alterações é persistido em SQLite.
- O botão **Histórico** permite filtrar por período, estado terminal, modo e saúde, comparar KPIs e excluir registros persistidos. Apenas registros `completed` oferecem abertura do relatório; `error` e `cancelled` permanecem auditáveis sem serem materializados como jobs. Os arquivos temporários e relatórios continuam sendo removidos no shutdown; PDF/DOCX são regenerados sob demanda a partir do snapshot persistente. A retenção automática é opcional por `HISTORY_RETENTION_DAYS`.

O limite padrão é de **500 MB por arquivo** e **10 arquivos por análise**. Esses valores podem ser alterados por variáveis de ambiente. Além desses limites de entrada, existe um limite operacional independente de **20 milhões de pacotes por análise** (`MAX_ANALYSIS_PACKETS`). O objetivo é evitar aceitar uma captura que o servidor não consiga processar de forma previsível. Todas as capturas usam o mesmo engine incremental. O limiar de 1 milhão de pacotes apenas classifica o **Resource Profile** como `large`, permitindo aplicar políticas de recursos mais conservadoras sem alterar parsing, correlação, métricas ou diagnóstico.

## Semântica e calibração do engine unificado

A calibração introduzida na v2.43 e protegida pela v2.44 usa os três cenários-base acordados: `2001_2002_001`, `9392-to-9999_002` e o Multi-PCAP `captura_sip_00153_20260821123524 + captura_sip_00152_20260821122806`. O objetivo é proteger **semântica**, não criar um caminho especial para essas capturas.

- **Bitrate RTP agregado da chamada:** soma os bytes dos frames RTP correlacionados a todas as pernas da chamada lógica e divide pelo intervalo RTP observado. É tráfego capturado, não bitrate nominal do codec. Em B2BUA/multi-leg, somar as pernas é intencional no nível lógico.
- **Bitrate/PPS por stream:** exige pelo menos dois pacotes e duração observável. Um SSRC com um único pacote fica `N/A`, evitando outlier artificial como `1 pacote / 1 ms`.
- **Jitter:** segue o algoritmo de interarrival da RFC 3550 somente para payload de áudio cujo clock pode ser sustentado pelo SDP/payload conhecido. Event payloads e ausência de clock não geram jitter inventado.
- **Inventário nativo:** pacotes já dissecados como RTP pelo TShark. **Candidatos adicionais** podem surgir por Decode-As/fallback em portas negociadas. Só pacotes estruturalmente válidos seguem para correlação.
- **RTP associado/não associado:** um RTP válido só entra nas métricas de uma chamada quando a correlação é sustentada. Falhas de correlação são registradas com motivos como janela temporal incompatível, ausência de endpoint SDP compatível ou empate entre chamadas; ambiguidade permanece não associada.
- A tabela forense por SSRC é limitada por `MAX_UNASSOCIATED_RTP_STREAMS`; os totais por motivo continuam sendo contabilizados mesmo quando o limite de detalhes é atingido.

### Checklist de validação de infraestrutura

O checklist continua sendo **orientação de validação externa**, não causa raiz automática. Na v2.43 cada item (`Firewall`, `NAT`, `SIP ALG`, `SBC/PBX`, `QoS/DSCP`, `Interfaces/enlaces`, `Codecs/transcodificação/media proxy` e `Roteamento/ponto de captura`) registra:

- se foi **priorizado** por um candidato diagnóstico da captura ou aparece apenas como baseline informativo;
- o **motivo da prioridade**;
- a quantidade de chamadas afetadas e os Call-IDs disponíveis para rastreio;
- `Validação externa`, `Evidência externa` e `Resultado`, preenchidos pelo usuário sem alterar os fatos observados no PCAP.

## Configuração

| Variável | Padrão | Finalidade |
|---|---|---|
| `HOST` | `0.0.0.0` | Endereço de escuta HTTP |
| `PORT` | `8000` | Porta HTTP |
| `TSHARK_BIN` | `tshark` | Executável do TShark |
| `MERGECAP_BIN` | `mergecap` | Executável do mergecap |
| `MAX_UPLOAD_MB` | `500` | Limite por arquivo de captura |
| `MAX_CAPTURE_FILES` | `10` | Quantidade máxima de capturas por análise |
| `MAX_ANALYSIS_PACKETS` | `20000000` | Limite operacional de pacotes para uma análise |
| `LARGE_CAPTURE_PACKET_THRESHOLD` | `1000000` | Limiar para classificar o perfil operacional como `large`; não troca o engine nem a metodologia analítica |
| `MAX_SIGNALING_ROWS` | `1000000` | Limite de registros SIP/SDP mantidos em memória pelo engine unificado |
| `MAX_RTCP_ROWS` | `500000` | Limite de registros RTCP mantidos em memória pelo engine unificado |
| `MAX_DECODE_AS_PORTS` | `512` | Máximo de portas negociadas aplicadas como regras Decode-As em uma única execução; acima disso usa fallback de payload em streaming |
| `MAX_UNASSOCIATED_RTP_STREAMS` | `5000` | Máximo de SSRC/fluxos RTP não associados mantidos em detalhe forense; totais por motivo não são limitados |
| `FORENSIC_SPOOL_BUFFER_STREAMS` | `512` | Quantidade máxima de agregados de stream mantidos no buffer do spool forense antes do flush em SQLite temporário |
| `FORENSIC_EXPORT_BATCH_SIZE` | `500` | Tamanho dos lotes usados para persistência/leitura incremental das exportações forenses |
| `FORENSIC_PREVIEW_PAGE_SIZE` | `25` | Quantidade padrão de streams exibidos por página na pré-visualização forense |
| `FORENSIC_PREVIEW_MAX_PAGE_SIZE` | `100` | Limite máximo permitido por página na pré-visualização forense |
| `FORENSIC_CONTEXT_MAX_CANDIDATES` | `5` | Máximo de chamadas candidatas exibidas no drill-down de um stream RTP não associado |
| `FORENSIC_CONTEXT_NEARBY_SECONDS` | `30` | Janela temporal, em segundos, para incluir candidatos sem evidência de endpoint/porta |
| `ANALYSIS_TIMEOUT_SECONDS` | `1800` | Tempo limite de operações de análise/merge |
| `MIN_FREE_DISK_MB` | `1024` | Espaço livre mínimo exigido pelo preflight no volume de dados |
| `MIN_RECOMMENDED_RAM_GB` | `4` | RAM recomendada exibida pelo preflight; não bloqueia análise |
| `MIN_RECOMMENDED_CPU_CORES` | `2` | Núcleos de CPU recomendados exibidos pelo preflight; não bloqueia análise |
| `HISTORY_DB` | `data/analysis_history.db` | Caminho do banco SQLite do histórico persistente |
| `HISTORY_RETENTION_DAYS` | `0` | Retenção automática do histórico em dias; `0` mantém o histórico até exclusão controlada |
| `TOTP_VIEWER_SECRET` | vazio | Segredo Base32 RFC 6238 do papel Viewer; obrigatório quando `AUTH_MODE=totp` |
| `TOTP_ANALYST_SECRET` | vazio | Segredo Base32 RFC 6238 do papel Analyst; obrigatório quando `AUTH_MODE=totp` |
| `TOTP_ADMIN_SECRET` | vazio | Segredo Base32 RFC 6238 do papel Admin; obrigatório quando `AUTH_MODE=totp` e usado server-side como raiz de proteção dos backups `.vcadb` |
| `TOTP_ISSUER` | `VoIP Capture Analyzer` | Nome exibido pelo aplicativo Authenticator |
| `TOTP_PERIOD_SECONDS` | `30` | Período TOTP em segundos |
| `TOTP_DIGITS` | `6` | Quantidade de dígitos do código TOTP (`6` ou `8`) |
| `TOTP_VALID_WINDOW` | `1` | Janela de tolerância de relógio em passos TOTP para trás/frente |
| `TOTP_MAX_FAILURES` | `5` | Falhas consecutivas por origem/papel antes do bloqueio temporário |
| `TOTP_LOCKOUT_SECONDS` | `300` | Duração do bloqueio temporário após excesso de falhas |
| `DB_BACKUP_MAX_MB` | `512` | Tamanho máximo aceito para um arquivo `.vcadb` importado |
| `DB_BACKUP_KDF_ITERATIONS` | `600000` | Iterações PBKDF2-HMAC-SHA256 usadas para derivar a chave AES-256 do backup protegido; mínimo aceito: 200000 |

Exemplo:

```bash
export PORT=8080
export MAX_UPLOAD_MB=1024
export MAX_CAPTURE_FILES=20
python run.py
```

## Docker

O `docker-compose.yml` permite parametrizar a porta publicada:

```bash
PORT=8080 docker compose up -d --build
```

A aplicação ficará disponível na porta escolhida no host.

Em diagnósticos de mídia/sinalização, valide também firewall, portas/protocolos/serviços, NAT e eventual SIP ALG quando os achados forem compatíveis.

## Estrutura

```text
app/
  analyzers/       Motor de análise e modelos
  services/        TShark, mergecap, gráficos, relatórios, glossário e jobs
  static/          JavaScript e identidade visual da interface
  templates/       Dashboard e template do relatório
data/              Arquivos temporários e relatórios gerados em runtime
tests/             Testes automatizados
run.py             Inicialização simplificada com escolha de host/porta
LICENSE            Licença MIT
CONTRIBUTING.md    Diretrizes para contribuições
CHANGELOG.md       Histórico de alterações
```

## Licenciamento

O código-fonte deste projeto é distribuído sob a **Licença MIT**. Consulte o arquivo [LICENSE](LICENSE) para os termos completos.

As ferramentas e bibliotecas externas utilizadas pelo ambiente de execução permanecem sujeitas às suas próprias licenças. O Analyzer executa TShark e mergecap como utilitários externos por linha de comando e não incorpora o código-fonte dessas ferramentas.

## Download

A suíte atual está disponível no release mais recente do GitHub:

[Download da suíte atual](https://github.com/VoIP-Capture-Analyzer/VoIP-Capture-Analyzer/releases/latest/download/VoIP_Capture_Analyzer.zip)

## Desenvolvimento

Execute os testes com:

```bash
pytest
```

Para regras de contribuição, padrões de código e processo de Pull Request, consulte [CONTRIBUTING.md](CONTRIBUTING.md).

## Gate de baseline, auditoria e política de tolerâncias (v2.50)

O baseline continua sendo aceito **manualmente** pelo usuário e armazenado fora de `analyses.report_json`. A avaliação direta da execução atual/histórica usa um baseline aceito compatível e, na v2.50, cada execução explícita do gate é anexada a um ledger de auditoria imutável. A seleção automática usa o baseline aceito mais recente com o **mesmo conjunto completo de SHA-256** e o **mesmo modo de análise**; também é possível indicar um `baseline_job_id` específico na API.

Endpoints:

- `GET /api/history/{job_id}/acceptance-gate` — compara com o baseline compatível mais recente;
- `GET /api/history/{job_id}/acceptance-gate?baseline_job_id={id}` — força um baseline aceito específico, ainda sujeito à regra de comparabilidade;
- `GET /api/acceptance-policy/default` — retorna a política VCA default;
- `PUT /api/history/{job_id}/acceptance-baseline/policy` com `{"policy":{...}}` — atualiza somente a política do baseline;
- `PUT /api/history/{job_id}/acceptance-baseline` também aceita `policy` opcional ao registrar/atualizar o baseline.
- `GET /api/history/{job_id}/acceptance-gate/audit?limit={n}` — consulta a auditoria append-only e a tendência cronológica do resultado;
- `GET /api/history/{job_id}/acceptance-gate/audit/export/json` e `/csv` — exportam até 1.000 execuções do gate sem alterar o histórico.

A política default controla KPIs como quantidade de chamadas, ASR, PDD T3, jitter p95, perda média, RTP unilateral e contagens de achados HIGH/CRÍTICO, além de aumentos de achados por severidade e divergências de identidade de chamada. `warn_abs` e `fail_abs` são desvios absolutos em relação ao baseline; `direction` define se apenas aumento, apenas redução ou qualquer mudança é prejudicial. A política pode ser editada no histórico pelo botão **Política**.

`PASS`, `WARN` e `FAIL` são **gates operacionais VCA**, não limites universais definidos por RFC/ITU-T e não constituem aprovação automática. O resultado nunca altera o snapshot nem registra causalidade. Uma análise sem hashes completos, com hashes diferentes ou modo diferente retorna `NOT_COMPARABLE`.

## Registro de requisitos e gate de rastreabilidade (v2.50)

O módulo `app/services/requirements_registry.py` mantém o registro versionado de requisitos da metodologia. Cada requisito possui:

- ID estável;
- título e declaração VCA concisa;
- classificação (`NORM`, `VCA-MET`, `VCA-OPS` etc.);
- referência à **VoIP Capture Analyzer - Especificação Conceitual, Metodológica e Normativa**;
- fontes normativas oficiais quando aplicáveis, por referência/link e sem reprodução integral;
- arquivos de implementação;
- arquivos de teste;
- superfícies do dashboard/relatório impactadas.

A API expõe `GET /api/requirements` e `GET /api/requirements/{requirement_id}`, exportações em `GET /api/requirements/export/{json|csv}`, o gate em `GET /api/requirements/traceability-gate` e suas exportações em `GET /api/requirements/traceability-gate/export/{json|csv}`. Novas análises também guardam um snapshot do registro dentro de `data.requirements_registry`; PDF/DOCX e a Metodologia acessível pelo menu Projeto exibem um resumo curto da versão e quantidade de requisitos, sem inflar o relatório com a matriz completa.


O gate de rastreabilidade confirma que cada requisito possui referência de especificação, arquivos de implementação existentes, testes existentes e arquivos de superfície de relatório/dashboard. Requisitos ligados a caminhos alterados são marcados como impactados. Mudanças metodológicas ou normativas devem ser declaradas explicitamente pelos respectivos IDs; IDs desconhecidos tornam o gate `FAIL`. Um `PASS` comprova a integridade desses vínculos de engenharia, não substitui testes funcionais nem validação das capturas-base.

O registro atual referencia principalmente RFCs do IETF/RFC Editor; a estrutura admite ITU-T, IEEE e 3GPP quando um requisito concreto do VCA depender desses organismos. A regra de projeto permanece: **norma externa, metodologia VCA, limiar operacional e decisão de implementação são categorias diferentes**.


## Fechamento do VCA Core VoIP e comparação entre releases (v2.51)

O endpoint `GET /api/requirements/core-closure-gate` confronta o escopo original SIP/SDP/RTP/RTCP contra a Especificação Conceitual, Metodológica e Normativa. Cada tema recebe um dos estados `IMPLEMENTED`, `PARTIAL`, `N/A` ou `PENDING`, com vínculos para requisitos, testes e evidências das três capturas-base. `OPEN` significa que o Analyzer permanece funcional, porém ainda há lacunas bloqueantes para declarar formalmente concluído o conceito inicial; `READY_TO_CLOSE` só pode ocorrer quando não houver lacunas bloqueantes e o Gate de rastreabilidade da release também estiver em `PASS`.

Exportações: `GET /api/requirements/core-closure-gate/export/json` e `/csv`. A fonte oficial `https://wiki.wireshark.org/SampleCaptures` pode ser utilizada futuramente como evidência suplementar para comportamentos que as três capturas-base não exercitam, sem substituir silenciosamente os baselines VCA.

A continuidade da matriz é verificada por `GET /api/requirements/release-comparison`. O comparador classifica requisitos como `UNCHANGED`, `ENGINEERING_CHANGED`, `METHODOLOGY_CHANGED`, `NORMATIVE_CHANGED`, `ADDED` ou `REMOVED`. Alteração metodológica ou normativa de um requisito já publicado exige reconhecimento textual versionado na própria release; sem esse reconhecimento, o comparador e o Gate de rastreabilidade retornam `FAIL`. Exportações: `GET /api/requirements/release-comparison/export/json` e `/csv`.

Na v2.51 o Gate de fechamento permanece `OPEN` de forma deliberada: ainda existem lacunas identificadas em Reason/Q.850 estruturado, atributos de direção SDP, classificação explícita de early media e validação ponta a ponta de RTCP XR/MOS/R-Factor com captura real apropriada. O NER continua `N/A`/não bloqueante enquanto a evidência disponível não sustentar classificação confiável.

## Contato

[Bruno Barreto](https://linktr.ee/brunombarreto)



## Drift analítico por chamada e baseline de aceitação (v2.48)

Quando duas análises possuem **o mesmo conjunto completo de SHA-256 das entradas** e o mesmo modo, o comparativo analítico inclui drill-down por chamada. O pareamento é feito somente pelo conjunto canônico exato de `Call-ID` + `leg_call_ids`. Se a correlação de pernas mudou, existe sobreposição parcial, identidade ausente ou duplicada, o VCA registra a divergência como indeterminada/N/A e **não força pareamento**.

O comparativo pode ser consultado e exportado em JSON/CSV:

- `GET /api/history/{job_id}/reproducibility/analytical-drift/{other_job_id}`
- `GET /api/history/{job_id}/reproducibility/analytical-drift/{other_job_id}/export/json`
- `GET /api/history/{job_id}/reproducibility/analytical-drift/{other_job_id}/export/csv`

Uma análise concluída com manifesto completo pode ser registrada como **baseline de aceitação**. O registro é persistido em tabela própria e não reescreve `report_json` nem as evidências originais:

- `GET /api/history/{job_id}/acceptance-baseline`
- `PUT /api/history/{job_id}/acceptance-baseline` com `{"label":"...","note":"..."}`
- `DELETE /api/history/{job_id}/acceptance-baseline`

Na interface, o histórico identifica baselines aceitos e o seletor de comparação os destaca com `★`. Marcar um baseline significa registrar uma referência homologada pelo usuário; não significa que futuras execuções serão automaticamente aprovadas.

## Navegação contextual e zoom de gráficos (v2.48)

Na aba **Chamadas**, a análise micro abre imediatamente abaixo da linha selecionada. Na **Rastreabilidade forense**, os botões de análise micro e linha do tempo também expandem o conteúdo abaixo do candidato, preservando o ponto de investigação e mantendo a associação do RTP como `N/A · hipótese`.

Os gráficos do **dashboard web** podem ser clicados para abrir uma visualização ampliada com controles `-`, `100%` e `+`. PDF e DOCX permanecem documentos estáticos: não existe um mecanismo portátil e confiável de zoom interativo por imagem nesses formatos, portanto a ampliação é feita pelo visualizador de PDF ou pelo Word/compatível.

## Como interpretar proveniência e reprodutibilidade

`Drift de ambiente` e `Drift analítico` respondem a perguntas diferentes. É perfeitamente válido obter **“sem drift de ambiente”** ao comparar duas capturas diferentes: isso apenas diz que Analyzer/Python/TShark/capacidades são equivalentes. O drift analítico, por outro lado, só pode comparar resultados da **mesma entrada binária**, comprovada pelo conjunto de SHA-256, no mesmo modo. `RTCP XR: compatível` na matriz significa que o TShark expõe os campos necessários; não significa que a captura contém RTCP XR.

## Security Foundation / SSO / TLS (v2.54.1)

A aplicação suporta `AUTH_MODE=oidc` para SSO OpenID Connect. O fluxo implementado é **Authorization Code + PKCE S256**, com `state` e `nonce` por transação, validação de `issuer`, `audience`, expiração, algoritmo e assinatura via JWKS. O VCA armazena na sessão apenas identidade mínima e papel RBAC; access token, ID token e client secret não são gravados no histórico nem em logs.

Perfis de autorização: `viewer` (leitura), `auditor` (leitura sem mutação), `analyst` (análise/validação/exportação operacional) e `admin` (retenção, baseline e ações administrativas). O controle é feito no backend; ocultar um botão na UI nunca é usado como autorização.

Para produção, publique o VCA atrás de HTTPS e restrinja o Uvicorn à interface interna/localhost. O arquivo `deploy/nginx-vca.conf.example` contém um exemplo de terminação TLS e proxy de WebSocket. Configuração mínima típica:

```env
HOST=127.0.0.1
AUTH_MODE=oidc
SESSION_SECRET=<segredo-aleatorio-com-pelo-menos-32-caracteres>
SESSION_COOKIE_SECURE=1
FORCE_HTTPS=1
TRUSTED_HOSTS=vca.exemplo.com
OIDC_ISSUER=https://idp.exemplo.com
OIDC_CLIENT_ID=voip-capture-analyzer
OIDC_CLIENT_SECRET=<secret-do-cliente-confidencial>
OIDC_REDIRECT_URI=https://vca.exemplo.com/auth/callback
```

`GET /api/security/status` apresenta o **Security Gate** sem expor segredos. Um gate verde/sem warnings depende da configuração de produção; ele não substitui pentest independente. Desde a v2.57 o frontend usa CSS estático empacotado localmente; o runtime não depende do Tailwind browser compiler/CDN nem de `unsafe-eval`.

A documentação interativa `/docs`, `/redoc` e `/openapi.json` fica desabilitada por padrão (`API_DOCS_ENABLED=0`). O Uvicorn também é iniciado sem o header `Server`.



### Estrutura de sinalização e controle (v2.68.2)

O VCA diferencia fato observado de hipótese diagnóstica. `180 Ringing` e `183 Session Progress` são respostas provisórias SIP com papéis distintos; a ausência de `180` não é tratada isoladamente como falha. Desde a v2.68.2, a presença de RTP antes do `200 OK` não é suficiente para suprimir o diagnóstico: o Analyzer também avalia a **direção** da early media.

A direção é resolvida de forma conservadora usando os endpoints RTP anunciados no SDP das respostas provisórias `180/183`. RTP **originado** nesses endpoints é tratado como mídia de retorno para o originador naquela perna; RTP apenas **destinado** a eles representa o sentido oposto. O resultado é exposto como `BIDIRECTIONAL`, `RETURN_ONLY`, `OUTBOUND_ONLY`, `UNRESOLVED` ou `NOT_OBSERVED`.

Quando houver `183` sem `180`, o achado `SIP_PROGRESS_WITHOUT_RINGING` é suprimido apenas quando há mídia de retorno observável antes do atendimento. Early media somente no sentido de saída permanece como evidência compatível com ausência de ringback remoto. No modo **Sinalização**, RTP continua explicitamente não avaliável.

As análises **Macro** e **Micro** exibem possíveis causas como hipóteses a confirmar, incluindo SBC/PBX, encaminhamento de respostas provisórias, early media unidirecional, firewall/NAT, roteamento e media proxy. SDP descreve parâmetros de mídia e RTP representa a mídia efetivamente observada; STUN/TURN não são inferidos como causa sem evidência compatível.

No **Checklist de validação de infraestrutura**, `Expandir todos / Recolher todos` afeta todos os itens simultaneamente. Validações externas continuam sem alterar as evidências extraídas da captura.

### Paridade visual PDF Técnico ↔ DOCX (v2.67.0)

- **DOCX editável com identidade do PDF Técnico:** A4, margens equivalentes, logotipo, barra de destaque, meta-card, badge de saúde, cards de KPI, mapa do documento, blocos de proveniência, hierarquia de títulos, tabelas, gráficos e rodapé seguem o mesmo contrato visual do PDF.
- **Mesma organização e conteúdo técnico:** o DOCX segue as mesmas oito seções e reproduz os mesmos indicadores, tabelas, gráficos, RTCP, codec/mídia, timeline, achados e anexo de limitações. Pequenas diferenças de paginação entre Word/LibreOffice e WeasyPrint são esperadas.
- **Fonte única:** PDF Técnico e DOCX continuam derivados do mesmo snapshot e dos mesmos gráficos em memória; o DOCX não é uma imagem do PDF e permanece editável.
- **Repositório oficial:** referências de download/licença passam para `https://github.com/VoIP-Capture-Analyzer/VoIP-Capture-Analyzer/`.
- **SIP Signaling Coverage:** permanece diferida até existir conjunto de capturas de homologação suficiente; esta release não altera semântica SIP/SDP/RTP/RTCP.

### Experiência de relatórios e paridade de exportação (v2.66.0)

- **Viewer / Limpar tela:** `DELETE /api/runtime/reset` é permitido a Viewer, Analyst e Admin porque limpa somente estado efêmero concluído/cancelado; não apaga histórico persistido nem executa manutenção administrativa.
- **PDF Técnico e Executivo:** capa/meta-card, status visual de saúde, KPIs padronizados, hierarquia tipográfica mais clara, tabelas com melhor quebra e chamadas micro que podem fluir entre páginas sem criar vazios excessivos.
- **PDF Técnico:** inclui um mapa do documento e mantém metodologia/reprodutibilidade separadas da cadeia de evidências.
- **PDF Executivo:** explicita que é uma projeção resumida do mesmo snapshot e mantém os mesmos KPIs compartilhados do relatório técnico.
- **CSV:** cada linha de chamada passa a repetir o contexto macro do snapshot (`analysis_mode`, saúde, chamadas, ASR, PDD, jitter p95, perda média, RTP unilateral e pico simultâneo), facilitando auditoria isolada e comparação com PDF/DOCX/JSON.
- **Paridade:** todos os formatos são derivados do mesmo snapshot persistido. O conteúdo não precisa ser idêntico em extensão: PDF Executivo é resumido; PDF Técnico/DOCX são humanos e detalhados; JSON é a fonte estruturada completa; CSV é o inventário tabular de chamadas com contexto macro repetido. Os valores compartilhados devem ser numericamente consistentes.

### Autenticação TOTP por papel e gerenciamento protegido do SQLite (v2.64+, atualizado na v2.65.0)

O modo `AUTH_MODE=totp` implementa TOTP RFC 6238 compartilhado por **papel**, sem cadastro de usuário, senha ou dependência de Google/Microsoft. Cada papel possui um segredo Base32 próprio: `viewer`, `analyst` e `admin`. Na tela de login o operador escolhe o nível e informa o código atual do Authenticator; o backend cria uma sessão RBAC correspondente. Como o segredo é compartilhado por papel, a auditoria identifica o **nível de acesso**, não a pessoa individual.

Gere os segredos e QR Codes com:

```bash
python scripts/generate_totp_setup.py --role all --output-dir totp-setup --write-env totp-secrets.env
```

O script cria três PNGs QR Code, imprime as chaves Base32/URIs `otpauth://` e, com `--write-env`, grava `totp-secrets.env` com permissão restrita. A partir da v2.65, **`python run.py` carrega esse arquivo automaticamente quando ele estiver ao lado do launcher**, sem executar conteúdo como shell: apenas chaves TOTP conhecidas são aceitas e variáveis já definidas por systemd, Docker, shell ou secret manager têm precedência. Se o arquivo fornecer ao menos um segredo TOTP e `AUTH_MODE` não estiver definido, o launcher assume `AUTH_MODE=totp`; se `SESSION_SECRET` estiver ausente, cria uma chave efêmera forte para aquele processo.

Para laboratório local, portanto, depois de gerar `totp-secrets.env` basta:

```bash
python run.py
```

Em produção, continue configurando `AUTH_MODE=totp`, os segredos TOTP e uma `SESSION_SECRET` **estável** no serviço/secret manager. A chave efêmera criada pelo launcher é apenas conveniência de laboratório e invalida sessões a cada reinício. Proteja ou apague os PNGs após cadastrar os autenticadores, pois eles equivalem às credenciais daquele papel.

```env
AUTH_MODE=totp
SESSION_SECRET=<segredo-de-sessao-aleatorio-com-pelo-menos-32-caracteres>
TOTP_VIEWER_SECRET=<base32-viewer>
TOTP_ANALYST_SECRET=<base32-analyst>
TOTP_ADMIN_SECRET=<base32-admin>
TOTP_ISSUER=VoIP Capture Analyzer
TOTP_PERIOD_SECONDS=30
TOTP_DIGITS=6
TOTP_VALID_WINDOW=1
TOTP_MAX_FAILURES=5
TOTP_LOCKOUT_SECONDS=300
```

O menu **Projeto → Banco de dados** só aparece para sessão `admin`. Exportação e importação exigem ainda **reconfirmação do código TOTP Admin atual**, solicitada em modal somente quando a operação é acionada; no modo completo, `SUBSTITUIR` é confirmado no mesmo modal. O navegador envia somente o código temporário no header `X-TOTP-Code`. O segredo `TOTP_ADMIN_SECRET` nunca é enviado ao cliente e também é usado, com separação de domínio e PBKDF2-HMAC-SHA256, como raiz server-side para derivar a chave AES-256-GCM do contêiner `.vcadb`.

Consequência operacional: se `TOTP_ADMIN_SECRET` for rotacionado, backups antigos protegidos pela chave anterior não podem ser descriptografados com o segredo novo. Preserve o segredo anterior em cofre seguro enquanto existirem backups que ainda precisem ser restaurados, ou reexporte a base depois da rotação.

**Compatibilidade de backup:** a v2.64 usa o formato protegido `VCADB2`, cuja raiz criptográfica é o `TOTP_ADMIN_SECRET`. Backups `.vcadb` criados pela v2.63 com `DB_ADMIN_TOKEN` não são importados diretamente pela v2.64. Para preservar um backup legado, restaure-o com a v2.63 usando o segredo antigo e gere um novo backup depois da migração para v2.64.

A importação aceita **somente** `.vcadb` produzido pelo Analyzer. Antes de qualquer escrita, o backend descriptografa para arquivo temporário com permissão restrita, executa `PRAGMA integrity_check`, `PRAGMA foreign_key_check`, compara o schema com a versão atual, rejeita views/triggers/tabelas virtuais inesperadas e valida os campos JSON persistidos. Nenhum SQL textual proveniente do arquivo é executado.

Modos disponíveis:

- **Incremental:** importa apenas Job IDs inexistentes e preserva integralmente qualquer Job ID já presente na base atual. Registros dependentes acompanham somente os novos jobs.
- **Completo / substituição:** exige a confirmação textual `SUBSTITUIR`, cria snapshot de rollback e restaura a base anterior se a substituição falhar.

As importações são recusadas enquanto houver análise/processo ativo. `/api/database/*` exige sessão `admin`, CSRF nas mutações e TOTP Admin fresco em export/import. Com `AUTH_MODE=disabled`, o gerenciamento do banco não fica disponível.

O Histórico registra também a origem que iniciou cada análise. Em TOTP compartilhado, o registro é do papel (`TOTP Analyst`/`TOTP Admin`), não da pessoa; em SSO/OIDC é usado o nome/e-mail já presente na sessão. Snapshots anteriores à v2.65 aparecem como `Origem não registrada`.

Endpoints administrativos:

- `GET /api/database/status` — estado do recurso para sessão Admin;
- `POST /api/database/export` — exporta `.vcadb`; requer sessão Admin, CSRF e header `X-TOTP-Code`;
- `POST /api/database/import` — importa `.vcadb` com `mode=incremental|replace`; requer os mesmos controles e, em `replace`, `confirmation=SUBSTITUIR`.

### Exportações

- **PDF:** formato humano principal, aberto diretamente em nova guia.
- **JSON:** formato estruturado para integração, automação, auditoria e troubleshooting; não inclui caminhos locais de gráficos/arquivos.
- **DOCX:** mantido por compatibilidade e necessidade de edição manual, porém deixou de ser a exportação principal. A remoção futura deve ser tratada como mudança de compatibilidade e não silenciosamente.


### Cadastro em Google/Microsoft Authenticator e similares

O QR Code gerado contém um URI padrão `otpauth://totp/...` com algoritmo SHA-1, período de 30 segundos e 6 dígitos por padrão. No aplicativo Authenticator, use **Adicionar conta → Ler QR Code**. Se a câmera não estiver disponível, escolha a opção de **chave de configuração/chave manual**, informe a chave Base32 exibida pelo script, tipo **baseado em tempo (TOTP)** e o mesmo nome do papel. Sincronização correta do relógio do servidor e do celular é necessária; `TOTP_VALID_WINDOW=1` tolera um passo anterior/posterior para pequenos desvios.

### Login federado Google / Microsoft (v2.54.1)

O VCA suporta `AUTH_MODE=google`, `microsoft` ou `multi`. Em `multi`, a tela de login exibe **Entrar com Google** e **Entrar com Microsoft**. O fluxo continua Authorization Code + PKCE S256; a senha permanece no provedor e não é recebida pelo VCA.

Para Google, registre o callback `https://SEU-VCA/auth/callback/google`. Para Microsoft Entra ID, use preferencialmente um `MICROSOFT_TENANT_ID` específico e registre `https://SEU-VCA/auth/callback/microsoft`. A autorização é separada da autenticação: mantenha `AUTH_REQUIRE_ALLOWLIST=1` e configure `AUTH_ALLOWED_EMAILS` e/ou `AUTH_ALLOWED_DOMAINS`; papéis locais podem ser atribuídos por `AUTH_ADMIN_EMAILS`, `AUTH_ANALYST_EMAILS` e `AUTH_VIEWER_EMAILS`.

Em laboratório sem Internet, use `AUTH_MODE=disabled`. Não existe bypass oculto quando SSO está habilitado. Google/Microsoft externos exigem conectividade HTTPS do navegador e do servidor VCA aos endpoints do provedor.

Consulte `docs/SECURITY_DEPLOYMENT.md` para exemplos Apache/Nginx, HTTPS, Google e Microsoft. `scripts/clean_install_gate.sh` cria um venv vazio e reinstala `requirements.txt`; em rede isolada, forneça um wheelhouse local com `--wheelhouse /caminho`.





## History Snapshot Views & Actions (v2.60.3)

A v2.60.3 corrige a seleção efetiva da visualização histórica Completa/Sinalização e atualiza o cache-busting `20260921-2603` dos assets estáticos para impedir que o navegador reutilize bundle JavaScript anterior após upgrade.

No **Histórico**, uma análise concluída mantém o modo em que foi originalmente executada. Quando o snapshot original é **Completo**, o botão **Abrir análise concluída** permite escolher entre:

- **Completa** — exibe SIP, SDP, RTP, RTCP e métricas de mídia já persistidas;
- **Sinalização** — reutiliza o mesmo snapshot, mas oculta componentes exclusivos de RTP/RTCP e eventos de mídia.

Essa escolha é somente de visualização: não reprocessa a captura, não converte o snapshot, não altera hashes, baseline, score persistido ou evidências originais. Snapshots originalmente em **Sinalização** não podem ser promovidos para Completa.

Cada registro concluído mantém **Abrir análise concluída**, **Outras ações** e **Excluir** visíveis. **Outras ações** concentra Gate baseline, Marcar/Remover baseline e Política do baseline; o selo de baseline aceito continua visível no registro.

## Responsive Exports & Navigation (v2.59.0)

- **PDF Executivo/Técnico:** cada botão abre o PDF inline com `Content-Disposition` usando `VCA-Relatorio-Executivo_<job_id>.pdf` ou `VCA-Relatorio-Tecnico_<job_id>.pdf`; ao salvar pelo visualizador, o navegador recebe o mesmo nome com `job_id`.
- **Outros formatos:** menu único para DOCX, JSON e CSV. O CSV contém uma linha por chamada lógica com identificação, estado/status, duração/PDD, RTP, perda, jitter, score e códigos de achado.
- **Navegação:** `↑ Topo` retorna à barra `Executivo / Visão Macro / Chamadas / Topologia / Diagnóstico`, não ao início absoluto da página.
- **Responsividade:** viewport mobile preservado e layout mobile-first para KPIs, régua de etapas, ações do relatório, abas e busca de chamadas; tabelas extensas continuam com rolagem horizontal controlada.

## UX & Navigation Refinement (v2.58.0)

- **Histórico dedicado:** o acesso ao histórico saiu do cabeçalho e fica entre **Analisar captura** e **Nova captura**, mantendo o contador de registros.
- **Ajuda × Projeto:** Ajuda concentra orientação de uso e Glossário; Metodologia, Download, Licença, Contributing e Contato ficam em **Projeto**.
- **Área inicial compacta:** título, instrução, modos e dropzone ocupam menos altura para manter a régua/evolução da análise mais próxima da primeira dobra da tela.
- **Checklist recolhido:** cada item de validação de infraestrutura inicia fechado, com expansão individual e comando **Expandir todos/Recolher todos**.
- **Glossário pesquisável:** filtro textual por termo, palavra ou explicação em Ajuda e na página dedicada. O Glossário não é repetido na tela principal nem nos relatórios exportados.
- **Forense responsivo:** os filtros de RTP não associado usam distribuição responsiva, reservando mais espaço para motivo/SSRC e reduzindo os campos numéricos.
- **Voltar ao topo:** botão flutuante aparece após rolagem significativa.
- **PDF:** arquivos físicos continuam `<job_id>_tecnico.pdf` e `<job_id>_executivo.pdf`; a URL/guia usa `VCA-Relatorio-Tecnico.pdf` / `VCA-Relatorio-Executivo.pdf`; os botões **Baixar PDF** usam `VCA-Relatorio-Tecnico_<job_id>.pdf` / `VCA-Relatorio-Executivo_<job_id>.pdf`.

## Security & Reporting Hardening (v2.57.0)

- O dashboard e o glossário usam `app/static/tailwind.css` empacotado localmente; o runtime não depende do Tailwind CDN nem de `unsafe-eval`.
- A CSP usa `script-src 'self'`; estilos de elemento também ficam restritos a `'self'`. `style-src-attr 'unsafe-inline'` permanece somente para ajustes geométricos controlados do frontend (barra de progresso/zoom).
- PDFs são armazenados internamente como `<job_id>_tecnico.pdf` e `<job_id>_executivo.pdf`. Ao abrir no navegador, os nomes são estáveis: `VCA-Relatorio-Tecnico.pdf` e `VCA-Relatorio-Executivo.pdf`. Ao baixar, recebem o `job_id` para evitar colisões.
- O Executivo lista como prioritárias apenas chamadas com achados diagnósticos. Quando não há achados, informa explicitamente que nenhuma chamada requer priorização.
- O resumo de codecs passa a preferir rótulos SDP descritivos e apresenta payload RTP observado com nome do codec/PT quando conhecido; SDP e RTP continuam sendo evidências distintas.

Para desenvolvimento do CSS estático, consulte `CONTRIBUTING.md`. O arquivo gerado é versionado; Node/Tailwind não são dependências de runtime.

## Navigation & Context Evolution (v2.56.1)

- A guia **Executivo** permanece focada na situação geral e chamadas prioritárias. O botão **Ver análise micro** expande o detalhe logo abaixo do Call-ID selecionado, sem trocar de guia.
- A aba **Topologia** permite preencher **Papel / Alias** e **Observação** por endpoint. Esse conteúdo é contexto manual do operador, fica persistido no snapshot/histórico e não altera evidências, métricas, score ou diagnóstico.
- `TOPOLOGY_ALIASES_JSON` continua aceito apenas como fallback de compatibilidade para ambientes antigos; o fluxo recomendado é a edição pela interface.
- A análise micro apresenta **SDP anunciado**, **codec selecionado no SDP** somente quando houver evidência explícita e **payload/codec observado no RTP por direção**. Payloads dinâmicos sem mapeamento seguro permanecem identificados como dinâmicos, sem inferência de codec.
- **Abrir PDF Executivo** e **Abrir PDF Técnico** abrem o PDF diretamente por rotas com nomes amigáveis, sem `iframe`.
- Endpoints de contexto de topologia: `PATCH /api/jobs/{job_id}/topology-context` e `PATCH /api/history/{job_id}/topology-context`.

## Reporting & UX Evolution (v2.55.0)

A interface principal separa **Executivo**, **Visão Macro**, **Chamadas**, **Topologia** e **Diagnóstico**. **Metodologia** fica no menu **Projeto** e o **Glossário** fica somente em **Ajuda**. O PDF Executivo é resumido; o PDF Técnico preserva a cadeia de evidências completa. JSON passa a ser baixado, DOCX permanece como compatibilidade editável.

A identidade visual ativa continua sendo **VoIP Capture Analyzer**. Para uma eventual mudança futura de nome, existem assets de preparação em `app/static/branding-ready/` para **VCA** e **VC Analyzer**, ambos reutilizando o símbolo atual. Esses arquivos não são referenciados pela interface, login, relatórios, favicon ou documentação de marca em runtime.

Aliases de topologia são opcionais e não alteram inferências:

```bash
export TOPOLOGY_ALIASES_JSON='{"172.29.232.10":"SBC Principal","10.85.11.15":"PBX Cliente"}'
```

O VCA mostra **SDP anunciado × payload RTP observado** como evidências distintas e não presume negociação offer/answer quando a captura não sustenta essa associação.
