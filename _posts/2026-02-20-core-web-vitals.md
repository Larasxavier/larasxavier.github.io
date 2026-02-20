---
title: "Monitoramento sintético (RUM): Você já usa o Core Web Vitals?"
description: "Uma visão prática sobre monitoramento e observabilidade, conectando dados técnicos a decisões estratégicas de negócio."
date: 2026-02-20 00:00:00 -0300
categories: [observabilidade, negocio]
tags: [observabilidade, monitoramento, sre, devops]
---

## O que é?
<p style="text-align: justify;">
Os Core Web Vitals são métricas do Google assim como os Golden Signals, RED e USE. Mas o foco delas é avaliar a experiência real do usuário através do desempenho de páginas web, interação e carregamento visual (usabilidade) da página.
</p>

## Quais são as métricas?
<p style="text-align: justify;">

- **Maior exibição de conteúdo (LCP):** avalia o desempenho do carregamento. Para oferecer uma boa experiência ao usuário, os sites devem fazer com que a LCP ocorra nos primeiros segundos do início do carregamento da página.
</p>
<p style="text-align: justify;">

- **Interaction to Next Paint (INP):** mede a capacidade de resposta. Para oferecer uma boa experiência ao usuário, busque uma INP de menos de x milissegundos.
</p>
<p style="text-align: justify;">

- **Cumulative Layout Shift (CLS):** avalia a estabilidade visual. Para oferecer uma boa experiência ao usuário, tenha uma pontuação de CLS inferior a x.
</p>

![Obs](/assets/images/core-web/cwv.png)


## Como coletar?

<p style="text-align: justify;">
Na ObservabilityCON, a Casas Bahia levou um case usando o K6 Browser para testar seu frontend e prepará-lo para eventos de vendas. 
</p>
<p style="text-align: justify;">
Eu utilizo muito o k6 para teste de carga e validação das minhas stacks de observabilidade, prever liberações e afins. Mas, a Grafana libera uma série de compenentes com essa vertente pelo K6.
</p>

- Teste de carga (Free)

- Desempenho do navegador (Teste grátis 15 dias, mas é pago)

- Monitoramento de desempenho e sintético (Usa outro componente da Grafana para agendar os testes)

- Automação de testes de desempenho(OSS e utiliza o xk6-disruptor para o Kubernetes)

- Testes de caos e resiliência

- Testes de infraestrutura ( ele possui algumas extensões para script)

 <p style="text-align: justify;">
 E aí talvez você se pergunte se o item web browser do zabbix, por exemplo, não traga essas métricas ou então o webscenário. Ele trará algumas métricas importantes de tempo de resposta, mas lá você precisará mapear os paths da página para cadastrar, enquanto no K6 Drowser será adcionado o link da página e autenticação se for o caso e essa varredura é automática gerando um script de teste no final da análise. Incluindo também pontos de atenção na interface. 
 </p>
<p style="text-align: justify;">
 Por isso, criei um painel comparativo após testar a capacidade de cada método de coleta apresentados pelo Zabbix e Grafana (ferramentas mais utilizadas de mercado)
</p>

![Obs](/assets/images/core-web/tabela.png)


## Overview das ferramentas

### HTTP Agent

![Obs](/assets/images/core-web/httpagent.png)

Fonte: Zabbix Documentation — HTTP agent items

Zabbix Documentation — HTTP agent item
https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/http


### Web Scenário

![Obs](/assets/images/core-web/scenario_details.png)

Fonte: Zabbix Documentation — Web monitoring scenarios

Zabbix Documentation — Web monitoring
https://www.zabbix.com/documentation/current/en/manual/web_monitoring

### Item Browser

![Obs](/assets/images/core-web/browser_item.png)

Fonte: Zabbix Documentation — Browser monitoring (Chromium)

Zabbix Documentation — Browser monitoring
https://www.zabbix.com/documentation/current/en/manual/web_monitoring/browser_monitoring

Zabbix Blog — Browser monitoring explained
https://blog.zabbix.com/browser-monitoring-in-zabbix/23755/



**DICA!**

Tem um vídeo do Bernardo Lankheet que pode ajudar melhor no aprofundamento de uso: https://www.youtube.com/watch?v=T5bid7sop7I

### K6 Browser

![Obs](/assets/images/core-web/k6-test.png)

Fonte: Grafana k6 — Browser testing documentation

![Obs](/assets/images/core-web/k6-perf.png)

Fonte: Grafana k6 — Browser testing documentation

![Obs](/assets/images/core-web/k6-trace.png)

Fonte: Grafana k6 — Browser testing documentation

Grafana k6 Documentation — Browser testing
https://grafana.com/docs/k6/latest/using-k6/browser-testing/

Grafana Blog — Introducing k6 Browser
https://grafana.com/blog/2023/02/02/introducing-k6-browser/


## Como aplicar na prática?

Iniciando pela definição do que precisa para o seu caso.
<p style="text-align: justify;">
Se você precisa de informações de disponibilidade, health checks, status code, tempo de resposta da página e alertas rápidos poderá utilizar dentre esses itens da Zabbix. Já se precisar de monitoramento sintético, que é o RUM, pode tanto utilizar o K6 Browser como concatenar o uso das duas tecnologias. Tendo o escopo bem definido par aqual caso usar terá um aproveitamento melhor dos dois mundos. 
</p>

## Medir as Core Web Vitals em JavaScript
<p style="text-align: justify;">
Posso também usar scripts em Javascript para obter essas métricas, pelo Zabbix ou outra forma de coleta. 
</p>

<p style="text-align: justify;">

1. Utilize a biblioteca JavaScript web-vitals, um wrapper pequeno e pronto para produção das APIs da Web que mede cada métrica de maneira precisa
</p>

Doc: https://github.com/GoogleChrome/web-vitals#api

```yaml
import {onCLS, onINP, onLCP} from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  // Use `navigator.sendBeacon()` if available, falling back to `fetch()`.
  (navigator.sendBeacon && navigator.sendBeacon('/analytics', body)) ||
    fetch('/analytics', {body, method: 'POST', keepalive: true});
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);

```
<p style="text-align: justify;">

2. Depois de configurar para medir, envie os dados das Core Web Vitals a um endpoint de análise.
</p>
<p style="text-align: justify;">

3. Analise se as páginas estão atendendo aos limites recomendados para pelo menos 75% das visitas.
</p>
<p style="text-align: justify;">

4. Os desenvolvedores que preferem medir essas métricas diretamente usando as APIs da Web tem essas aquique auxiliam o estado das implantações:
</p>


##### **LCP no Javascript:** https://web.dev/articles/lcp?hl=pt-br#measure_lcp_in_javascript

##### **INP no Javascript:** https://web.dev/articles/inp?hl=pt-br#how-to-measure-inp

##### **CLS no Javascript:** https://web.dev/articles/cls?hl=pt-br#measure_cls_in_javascript


Link artigo (A velocidade do site ainda está impactando sua taxa de conversão.): https://portent.com/blog/analytics/research-site-speed-hurting-everyones-revenue.htm
Link artigo (Fluxos de trabalho das Principais métricas da Web com ferramentas do Google): https://web.dev/articles/vitals-tools?hl=pt-br
Link artigo (Entendendo e usando as Core Web Vitals): https://akshayranganath.github.io/Understanding-And-Using-Core-Web-Vitals/




E aí, já conhecia esse outro lado? Devs e Observabilidade se unindo mais uma vez! hahaha


