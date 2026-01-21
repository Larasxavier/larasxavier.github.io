---
title: "Dominando o tempo no Grafana: Você sabe usar as opções de período disponíveis?"
description: "Uma visão prática sobre monitoramento e observabilidade, conectando dados técnicos a decisões estratégicas de negócio."
date: 2026-01-15 00:00:00 -0300
categories: [observabilidade, negocio]
tags: [observabilidade, monitoramento, sre, devops]
---

Se tem uma coisa que define observabilidade, é tempo.
Saber quando aconteceu algo é tão importante quanto saber o quê.

E no Grafana, isso vai muito além de clicar em “Last 1h” ou “Last 24h”.

<p style="text-align: justify;">
Existe todo um arsenal de formas de manipular o tempo…algumas conhecidas, outras escondidas nas configurações de painel ou dentro das queries.
</p>

Esse artigo é quase um modo discovery do que dá para fazer com **tempo no Grafana.**

### 1°) Time Range Global

![Lara](/assets/images/tempo-grafana/01.png)

**O clássico:** no topo do dashboard você escolhe quanto tempo quer ver.

**Valores prontos:** Last 5m, Last 7d, Last 30d.

**Absoluto:** marcar data/hora específica.

**Atalho:** “Zoom out” e “Zoom in” no gráfico. Isso controla o contexto geral do dashboard inteiro.

### 2°) Relative Time (por painel)

![Lara](/assets/images/tempo-grafana/02.webp)

<p style="text-align: justify;">
Às vezes, você não quer que todos os gráficos sigam o mesmo range. O dashboard pode estar em 7 dias, mas um painel mostra só as últimas 2h.
</p>
Exemplo:Um dashboard de disponibilidade mostra 7 dias, mas você cria um painel só para “última hora de CPU” .

### 3°) Time Shift

![Lara](/assets/images/tempo-grafana/03.webp)

Aqui o Grafana brinca de viagem no tempo. Serve para comparar períodos diferentes.

Exemplo: 1d → compara essa semana com o dia anterior.


*Caso prático: Tráfego da aplicação de hoje vs. tráfego de ontem, lado a lado no mesmo gráfico.*

### 4°) Intervalos por Query (Overrides de Tempo)

![Lara](/assets/images/tempo-grafana/04.png)

Max data points e Interval override permitem controlar granularidade das series via query.

Relative time por query → **$__interval, $__rate_interval**

**$__timeFilter()** → limita dados ao range escolhido.

**$__interval** → ajusta a granularidade (5s, 1m, 1h).

**$__rate_interval** → usado em métricas de Prometheus para cálculos de taxa.

Exemplo com Prometheus:

```increase(http_requests_total[$__rate_interval])```

Assim a query se adapta sozinha, sem você ter que adivinhar se vai ser 1m, 5m ou 1h.5°)

Como venho atuando com mais frequência com a stack OSS, estão aqui alguns que mais utilizo.


#### Prometheus / Loki / Tempo

**time():** retorna o timestamp atual.

**offset:** desloca séries no tempo (metric offset 5m).

**$__timeFilter(column):** macro SQL-like (em datasources SQL).

**$__timeFrom() / $__timeTo():** limites inferior/superior do range.

**$__interval e $__rate_interval:** granularidade dinâmica.

### 6°) QL Datasources (Postgres, MySQL, etc.)

![Lara](/assets/images/tempo-grafana/05.png)

<p style="text-align: justify;">
Ao plugar bancos de dados como datasource, você também pode acrescentar variáveis que funcionem em sua query. Teste com período no banco e depois substitua no frontend do Grafana.
</p>

```WHERE $__timeFilter(datetime_column)```

```GROUP BY $__interval(datetime_column)```

```date_trunc('hour', $__timeGroup(datetime_column,'1h'))```

*Esses macros viram filtros de tempo reais na query, otimizando o que vem do banco.*

#### Insights e boas práticas…

**Auto-refresh:** 5s, 30s, 1m… perfeito para dashboards de NOC.

**Variáveis de tempo:** usar now-1h como variável de query.

**Templating de intervalos:** deixar o usuário escolher granularidade (ex.: 5m, 15m, 1h).

<p style="text-align: justify;">

- Não use ranges gigantes com granularidade de segundos: você vai matar o banco e não ganhar insight.

- Relative time é para detalhe, time shift é para comparação.

- Sempre adicione contexto com annotations — olhar métricas sem saber que rolou um deploy é perda de tempo.
</p>