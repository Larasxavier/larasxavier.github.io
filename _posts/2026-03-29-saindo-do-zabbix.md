---
title: "Saindo do mundo Zabbix para Opentelemetry: O que preciso saber?"
description: "Uma visão prática sobre monitoramento e observabilidade, conectando dados técnicos a decisões estratégicas de negócio."
date: 2026-03-29 00:00:00 -0300
categories: [observabilidade, negocio]
tags: [observabilidade, monitoramento, sre, devops]
---


Um guia prático (e sem trauma) para sair do mundo do polling e entrar na era da telemetria.
<p style="text-align: justify;">

Porque todo mundo está falando de opentelemetry agora? É importante que você entenda que um não irá substituir o outro, mas a medida que as infraestruturas avançam, tecnologias nascem e morrem para facilitar as coletas e enriquecê-las. Zabbix nasceu para monitorar infraestrutura. OpenTelemetry nasceu para entender sistemas distribuídos.
</p>

<p style="text-align: justify;">

Quando falamos de Zabbix, temos uma estrutura de server, proxy, agent e banco de dados com coletas baseadas em polling criando itens (informações), triggers (regras de alerta) que fazem parte de templates prontos ou customizáveis (modelos prontos de organização dos dados coletados). Baseado em um modelo mental de perguntar constantemente se algo está bem ou mal. Logo, de forma bruta ele acaba sendo fundamental para estado, capacidade e infraestrutura.
</p>

**Quando o usuário do Zabbix começa a sentir dor?** 

Dificuldade para:

- Tracing distribuído

- Entender jornada de uma requisição

- Relacionar erro com impacto no usuário

- Muitas métricas ≠ entendimento real do problema

- Correlação manual (dashboards + experiência do operador)
<p style="text-align: justify;">

*O alerta dispara, mas o porquê não está claro. Isso vai necessitar de uma maturidade do operador de compreender o sistema como um todo e observar possíveis pontos monitorados.*
</p>

<p style="text-align: justify;">

É  nessas dores que o opentelemetry entra, ele é um framework de código aberto que coleta as métricas, logs e traces de forma agnóstica a ferramentas. Como diz Marilya Gutierrez, o Cabo tipo C da telemetria. Com ele é possível correlacionar essas informações. Achei mais interessante fazer tabelas comparativas sobre o foco de cada um, como se complementam e um breve glossário nos próximos capítulos, espero que ajude.
</p>


## Diferencial do formato de coleta entre Zabbix e OTel

![Obs](/assets/images/otel-zabbix/01-otel.png)

![Obs](/assets/images/otel-zabbix/coleta.png)


## Métricas, Logs e Traces

![Obs](/assets/images/otel-zabbix/metric.png)

## Foco dos domínios em cada uma

![Obs](/assets/images/otel-zabbix/foco.png)

## Quem responde o quê?

![Obs](/assets/images/otel-zabbix/quem.png)

## Glossário do Zabbix e Opentelemetry

![Obs](/assets/images/otel-zabbix/glossario.png)

## Quando usar Zabbix ou Opentelemetry? Podem ser juntas?

![Obs](/assets/images/otel-zabbix/otel01.png)


<p style="text-align: justify;">

Nesse caso, eles podem ser complementares. Uma não anula a outra e no mundo de observabilidade dificilmente conseguirá trabalhar apenas com uma ferramenta, porque as necessidades vão surgindo ao longo do tempo e os custos também hehehe, mas isso é papo pra outro artigo.
</p>

E aí, curtiu? Se tiver algo a complementar, fala comigo que add e te menciono!

