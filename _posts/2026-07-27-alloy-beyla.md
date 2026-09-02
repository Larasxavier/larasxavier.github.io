---
layout: post
title: "Beyla e Alloy: instrumentação sem código e o Collector que virou distribuição"
date: 2026-07-20 03:00:00 +0000
categories: observabilidade negocio
description: "A sequência prática do artigo de OpenTelemetry: como o Grafana Alloy reaproveita tudo que você já aprendeu de Collector e como o Beyla gera telemetria sem tocar no código, com eBPF."
---

No [artigo anterior de OpenTelemetry](https://larasxavier.github.io/observabilidade/negocio/2026/04/28/opentelemetry.html) a gente montou a base: receivers, processors, exporters, connectors, tail sampling, PII e os 10 mandamentos da cardinalidade. Se você não leu, leia primeiro, porque aqui eu vou puxar aqueles conceitos o tempo todo. É neles que Beyla e Alloy se apoiam.

A pergunta que fecha o ciclo do artigo passado é: *"beleza, entendi o Collector... mas e quando eu não posso mexer no código da aplicação? E como eu opero isso em escala sem virar zelador de YAML?"* É aqui que entram nossos dois convidados.


## Sumário

- [O mapa: onde Beyla e Alloy entram no cabo C](#1-o-mapa)
- [Grafana Alloy: o Collector que virou distribuição](#2-alloy)
  * [De pipeline em YAML para grafo de componentes](#21-de-pipeline-para-grafo)
  * [O de-para: seus processors viram componentes](#22-o-de-para)
  * [O que o Alloy ganha além do Collector (WAL e clustering)](#23-alem-do-collector)
  * [Tail sampling no Alloy (e por que o load balancing exporter volta aqui)](#24-tail-sampling)
  * [Quando NÃO usar o Alloy (vs OCB)](#25-quando-nao-alloy)
- [Grafana Beyla: telemetria sem código com eBPF](#3-beyla)
  * [Como ele enxerga por dentro](#31-como-enxerga)
  * [RED e o problema das rotas (oi, cardinalidade)](#32-red-rotas)
  * [Ilhas de trace: a limitação que define a arquitetura](#33-ilhas)
  * [PII no eBPF (a segurança não folga)](#34-pii)
  * [Quando NÃO usar o Beyla](#35-quando-nao-beyla)
- [Juntando tudo: piso, teto e o carteiro no meio](#4-juntando)

---

## 1. O mapa: onde Beyla e Alloy entram no cabo C {#1-o-mapa}

Lembra da analogia do **Cabo C** que a Marilya usa? O OpenTelemetry é o cabo universal, agnóstico, que conecta sua aplicação (o USB) a qualquer backend (Grafana, Datadog, Jaeger...). Vamos esticar essa analogia:

- A **SDK/instrumentação** é você soldando o USB no seu aparelho: controle total, detalhe máximo, mas precisa abrir o aparelho (mexer no código e fazer redeploy).
- O **Beyla** é um adaptador que lê o aparelho sem abrir: encaixa por fora e já começa a mandar telemetria, mesmo num aparelho que você não fabricou (app legado, de terceiro, poliglota).
- O **Alloy** é a régua/hub onde todos esses cabos chegam: ele recebe, transforma (os mesmos processors do artigo passado!) e distribui pros backends. É o Collector, só que numa embalagem com mais coisa dentro.

![Duas categorias: Alloy é coleta, Beyla é geração sem código](/assets/images/beyla-alloy/01_duas_categorias.png)

O erro nº1 de arquitetura é achar que Beyla e Alloy competem. Não competem: **um gera** telemetria, **o outro coleta**. Muita gente usa os dois juntos.

---

## 2. Grafana Alloy: o Collector que virou distribuição {#2-alloy}

Vou ser direta pra não te enrolar: o **Alloy é uma distribuição do OpenTelemetry Collector** (mais o mundo Prometheus embutido). Ou seja, **tudo** que você aprendeu no artigo passado (receiver, batch, memory_limiter, attributes, filter, tail sampling, exporter) **continua valendo**. Muda a linguagem de configuração e você ganha uns superpoderes de operação. Vamos por partes.

### 2.1 De pipeline em YAML para grafo de componentes {#21-de-pipeline-para-grafo}

No Collector, você declara `service.pipelines` ligando listas de receivers, processors e exporters. No Alloy, esse pipeline deixa de ser uma lista e vira um **grafo**: cada componente tem uma saída (`output`) que você pluga na entrada (`input`) do próximo. É tipo uma planilha: mudou uma célula, o que depende dela recalcula sozinho.

![Alloy como grafo de componentes (DAG)](/assets/images/beyla-alloy/03_alloy_dag.png)

Na prática técnica: por baixo existe um *component controller* que monta um DAG, valida ciclo, resolve a ordem e reavalia em cascata quando algo muda (um `local.file` que rotacionou, um secret novo). O tráfego entre componentes é **in-memory**, no mesmo processo.

### 2.2 O de-para: seus processors viram componentes {#22-o-de-para}

Aqui está a parte gostosa. Pega aquele pipeline do artigo passado e veja o mesmo em Alloy. Repare que os **parâmetros são idênticos** (`send_batch_size`, `timeout`, `check_interval`, `limit`):

```alloy
// receiver otlp igualzinho ao seu receivers.otlp
otelcol.receiver.otlp "in" {
  grpc { endpoint = "0.0.0.0:4317" }
  http { endpoint = "0.0.0.0:4318" }
  output {
    traces  = [otelcol.processor.memory_limiter.default.input]
    metrics = [otelcol.processor.memory_limiter.default.input]
    logs    = [otelcol.processor.memory_limiter.default.input]
  }
}

// memory_limiter com os mesmos números do artigo (2GB / 1.5GB / 5s)
otelcol.processor.memory_limiter "default" {
  check_interval = "5s"
  limit          = "2GiB"
  spike_limit    = "1500MiB"
  output {
    traces  = [otelcol.processor.batch.default.input]
    metrics = [otelcol.processor.batch.default.input]
    logs    = [otelcol.processor.batch.default.input]
  }
}

// batch com send_batch_size 1000, timeout 5s (idem)
otelcol.processor.batch "default" {
  send_batch_size = 1000
  timeout         = "5s"
  output {
    traces  = [otelcol.exporter.otlp.tempo.input]
    metrics = [otelcol.exporter.prometheus.default.input]
    logs    = [otelcol.exporter.otlphttp.loki.input]
  }
}

otelcol.exporter.otlp "tempo" {
  client { endpoint = "tempo:4317"  tls { insecure = true } }
}
```

O de-para mental é esse:

| No Collector (YAML) | No Alloy |
|---|---|
| `receivers.otlp` | `otelcol.receiver.otlp` |
| `processors.batch` | `otelcol.processor.batch` |
| `processors.memory_limiter` | `otelcol.processor.memory_limiter` |
| `processors.attributes` (hash/delete PII) | `otelcol.processor.attributes` |
| `processors.filter` (OTTL) | `otelcol.processor.filter` |
| `processors.tail_sampling` | `otelcol.processor.tail_sampling` |
| `exporters.otlp` / `prometheus` | `otelcol.exporter.otlp` / `prometheus.remote_write` |
| `service.pipelines` (a lista) | as ligações `output` para `input` (o grafo) |

Aquele bloco de **attributes** que você usou pra hash de e-mail e delete de `db.statement`? Vira `otelcol.processor.attributes` com as mesmas actions. Aquele **filter** OTTL pra cortar `.*password.*`? Vira `otelcol.processor.filter`. **Nada do que você aprendeu se perde.**

### 2.3 O que o Alloy ganha além do Collector (WAL e clustering) {#23-alem-do-collector}

Se é o mesmo Collector, por que usar Alloy? Por causa de duas heranças do mundo Prometheus que resolvem dor de operação.

**WAL no remote_write.** Quando o backend cai, o Alloy guarda as amostras num *write-ahead log* em disco e reenvia depois. Mas atenção ao perigo (e isso conecta com seu mandamento nº8, "não sacrificarás o backend"): a retenção do WAL é por **segmento**, não por idade. Em queda longa, o truncamento descarta os segmentos mais antigos e **você perde dado silenciosamente**. Dimensione pelo *pior* outage, não pelo médio.

![O WAL do remote_write e a perda silenciosa](/assets/images/beyla-alloy/04_alloy_wal.png)

**Clustering.** Vários Alloys dividem os alvos de scrape usando *consistent hashing*: quando um nó entra ou sai, só cerca de 1/N dos alvos se remaneja (nada de rebalance geral). É o que te deixa escalar horizontal sem KV externo.

### 2.4 Tail sampling no Alloy (e por que o load balancing exporter volta aqui) {#24-tail-sampling}

Lembra que o tail sampling foi "o tema mais polêmico e que faz diferença financeira"? Ele existe igualzinho no Alloy. Olha as **mesmas policies** do artigo passado, agora em sintaxe Alloy:

```alloy
otelcol.processor.tail_sampling "policy" {
  decision_wait = "30s"
  num_traces    = 50000

  policy {
    name = "erros"
    type = "status_code"
    status_code { status_codes = ["ERROR"] }
  }
  policy {
    name = "lentos"
    type = "latency"
    latency { threshold_ms = 8000 }
  }
  policy {
    name = "rotas_criticas"
    type = "string_attribute"
    string_attribute {
      key    = "http.route"
      values = ["/checkout", "/payment"]
    }
  }
  policy {
    name = "amostra_saudavel"
    type = "probabilistic"
    probabilistic { sampling_percentage = 5 }
  }

  output { traces = [otelcol.exporter.otlp.tempo.input] }
}
```

Agora o pulo do gato, no artigo do opentelemetry havia falado sobre o *load balancing exporter* com `routing_key = traceID` porque "traces não são dados independentes, são conjuntos de spans que precisam se encontrar no mesmo lugar". Pois é **exatamente** por isso que você **não pode** simplesmente subir 3 réplicas de Alloy fazendo tail sampling e achar que acabou. Cada réplica precisa ver **todos** os spans de um mesmo trace pra decidir. A solução é uma camada de balanceamento por trace-ID **na frente** do tail sampling:

```alloy
otelcol.exporter.loadbalancing "tail" {
  routing_key = "traceID"
  resolver { dns { hostname = "alloy-tail-headless" } }
  protocol  { otlp { client { tls { insecure = true } } } }
}
```

Ou seja: esse componente é a peça que torna o tail sampling **escalável**. 

### 2.5 Quando NÃO usar o Alloy (vs OCB) {#25-quando-nao-alloy}

*"Contrib é para experimentar, Core é para aprender, OCB é para operar."* O Alloy é o oposto filosófico do OCB, e isso é um trade-off honesto:

- **OCB:** você monta um binário **enxuto**, só com o que usa, superfície de ataque mínima.
- **Alloy:** binário **baterias-incluídas** (Collector + Prometheus + Beyla embutido), mais pesado na memória, com uma linguagem de config própria.

Então **não** use Alloy quando: você padronizou no Collector upstream + Operator com YAML/GitOps (a linguagem do Alloy vai brigar com esse mundo); ou quando você precisa de um coletor mínimo e auditável, e aí o OCB ganha. Use Alloy quando quer um agente único que já fala Prometheus, faz clustering e roda Beyla sem instalar mais nada.

---

## 3. Grafana Beyla: telemetria sem código com eBPF {#3-beyla}

Toda a conversa do artigo passado assume que a telemetria **chega** no receiver. Mas e o serviço legado que ninguém recompila? O binário de terceiro? O job em Cobol que roda desde 2009? Essa é a **cauda longa** que fica cega, e o Beyla ataca ela.

O Beyla usa **eBPF** pra observar a aplicação **de fora do processo**, sem SDK, sem redeploy. Pensa em câmeras de segurança no corredor do prédio (o kernel) em vez de uma câmera dentro de cada apartamento (cada app).

### 3.1 Como ele enxerga por dentro {#31-como-enxerga}

![Como o Beyla gera o span: uprobe em Go, kprobe no kernel](/assets/images/beyla-alloy/06_beyla_spans.png)

Depende da linguagem: pra **Go**, ele coloca *uprobes* na entrada/saída de funções conhecidas (tipo `ServeHTTP`) e até rastreia goroutines pra ligar entrada e saída. Pras **outras linguagens**, desce pro kernel com *kprobes* (`sys_accept`, `tcp_recvmsg`, `tcp_sendmsg`) e remonta o HTTP/gRPC do tráfego cru.

E o HTTPS? Ele não fica lendo pacote cifrado. Coloca *uprobe* nas funções da lib TLS (`SSL_read`/`SSL_write`) e lê o **texto claro** no instante antes de cifrar e depois de decifrar.

Ligar o Beyla à sua stack do artigo passado é trivial, porque ele **fala OTLP**. É só apontar pro seu Collector/Alloy:

```yaml
env:
  - name: BEYLA_OPEN_PORT
    value: "8080"                       # o que observar
  - name: BEYLA_KUBE_METADATA_ENABLE
    value: "true"
  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://alloy:4318"          # cai no seu pipeline
```

E dá pra rodar o Beyla **dentro do próprio Alloy**, como componente (aí ele vira mais um nó do grafo, e você aplica os mesmos processors de PII/cardinalidade no que ele gera):

```alloy
beyla.ebpf "auto" {
  open_port = "8080"
  attributes { kubernetes { enable = "true" } }
  output { traces = [otelcol.processor.tail_sampling.policy.input] }
}
```

### 3.2 RED e o problema das rotas (oi, cardinalidade) {#32-red-rotas}

O Beyla é craque em **métricas RED** (Rate, Errors, Duration), os três números que respondem "meu serviço tá saudável?". Só que ele vê a **URL crua**. E aí, lembra do**mandamento nº4, "Santificarás as rotas"**? *"/order/983742 é um ataque silencioso à sua stack."* Pois o Beyla é justamente onde esse ataque acontece se você não configurar os padrões de rota:

```yaml
routes:
  patterns:
    - /users/{id}
    - /orders/{id}
  unmatched: heuristic
```

Sem o *routes decorator*, cada `/order/983742` vira uma série nova e você detona a cardinalidade do Mimir (e a fatura). **O Beyla não é opcional nesse ponto, é o lugar onde os teus mandamentos de cardinalidade têm que ser aplicados na origem.**

### 3.3 Ilhas de trace: a limitação que define a arquitetura {#33-ilhas}

Essa é a parte que você **precisa** internalizar antes de prometer trace distribuído com Beyla.

![HTTP propaga; HTTPS atrás de proxy L7 quebra o contexto](/assets/images/beyla-alloy/08_beyla_ilhas_trace.png)

Costurar spans de serviços diferentes num trace só exige passar o `traceparent` adiante. Em **Go, HTTP simples**, o Beyla faz isso lindo. Mas em **HTTPS com um proxy L7 / load balancer no meio** (leia-se: quase toda malha real), o Beyla injeta o contexto no nível do pacote TCP/IP, e o proxy **reescreve o envelope**. O contexto se perde e o teu trace vira **ilha** (spans soltos por serviço, sem o caminho completo).


### 3.4 PII no eBPF (a segurança não folga) {#34-pii}

Como disse, *"se você não trata segurança na origem, você está criando um data lake de risco."* Com eBPF isso fica **mais** verdadeiro, porque o Beyla enxerga URL, query string e headers crus. Ou seja, PII pode entrar na telemetria sem você escrever uma linha. A boa notícia: como tudo cai no seu pipeline, você aplica **os mesmos processors da sua seção de segurança**. Hash no que precisa correlacionar, delete no que não precisa:

```alloy
otelcol.processor.attributes "pii" {
  action { key = "url.query"                         action = "delete" }
  action { key = "http.request.header.authorization" action = "delete" }
  action { key = "user.email"                         action = "hash" }
  output { traces = [otelcol.processor.tail_sampling.policy.input] }
}
```

Zero-code na coleta **não** significa zero-cuidado com PII. O eBPF facilita a captura, e captura sem tratamento é vazamento esperando pra acontecer.

### 3.5 Quando NÃO usar o Beyla {#35-quando-nao-beyla}

- Cluster onde você **não pode** rodar workload privilegiado/`hostPID` (o eBPF exige isso, e em ambiente regulado isso é um "não" político antes de técnico).
- Serviços que **já têm SDK boa**, porque o Beyla acrescenta pouco onde já há profundidade.
- Necessidade de **trace distribuído confiável** numa frota poliglota atrás de proxies/LBs (vai virar ilha).
- Caminhos de **altíssimo RPS e latência crítica**, onde o custo dos uprobes pesa (o "overhead zero" é marketing; uprobe tem custo por evento).
- Quando você precisa de **atributo de negócio** (aquele `user.role`, `plan=enterprise`), porque isso é SDK.

---

## 4. Juntando tudo: piso, teto e o carteiro no meio {#4-juntando}

![Arquitetura de referência: Beyla piso, SDK teto, Alloy no meio](/assets/images/beyla-alloy/10_arquitetura_referencia.png)

A combinação madura não é "um ou outro":

- **Beyla é o piso** de cobertura: RED + topologia de **toda** a frota, inclusive o legado, sem redeploy.
- **SDK (OpenTelemetry) é o teto** de profundidade: spans internos, atributos de negócio, propagação W3C completa, nos serviços que importam (tier-1).
- **Alloy é o carteiro no meio**: recebe os dois, aplica *routes decorator* (cardinalidade), PII (hash/delete), tail sampling (com load balancing por trace-ID quando escalar) e entrega no Tempo/Mimir/Loki.

---



Era pra ser sobre "as duas ferramentas", mas acho que já deu pra fazer outra boa caminhada, né? hahaha

