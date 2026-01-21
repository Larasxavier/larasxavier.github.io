---
title: "Monitorando versões de ferramentas com Zabbix: notificações automáticas de update"
description: "Uma visão prática sobre monitoramento e observabilidade, conectando dados técnicos a decisões estratégicas de negócio."
date: 2026-01-15 00:00:00 -0300
categories: [observabilidade, negocio]
tags: [observabilidade, monitoramento, sre, devops]
---

Manter ferramentas e stacks atualizadas é sempre um desafio.
O que normalmente acontece? Você descobre uma versão nova porque alguém manda no grupo, ou porque deu problema em produção.

<p style="text-align: justify;">

Mas dá para resolver isso com observabilidade preventiva: o próprio Zabbix pode monitorar endpoints de release e notificar quando uma nova versão sair.
</p>

Neste artigo mostro duas formas que implementei:

- Via HTTP Agent (buscando no GitHub e tratando com JavaScript).

- Via Script + Low-Level Discovery (LLD), automatizando para múltiplos softwares.

### Método 1: HTTP Agent + JavaScript
**O mais direto:** consultar a API ou página de releases (ex.: GitHub) e extrair a versão.

#### Passos
Criar um item do tipo HTTP agent apontando para o endpoint de releases.

Exemplo:

```https://api.github.com/repos/<org>/<repo>/releases/latest```



Configurar o **preprocessing → JavaScript** para extrair o campo da versão:

```var data = JSON.parse(value);

return data.tag_name;
```

Criar uma trigger para comparar com a versão esperada (macro ou valor anterior).

### Método 2: Script externo + Discovery (LLD)

Para cenários com várias ferramentas, montei um script que coleta versões e deixa o Zabbix fazer o resto.

### Exemplo (Python)

```python
#!/usr/bin/env python3

import requests
import json

repos = {
    "zabbix": "zabbix/zabbix",
    "grafana": "grafana/grafana",
    "prometheus": "prometheus/prometheus"
}

result = {"data": []}

for name, repo in repos.items():
    url = f"https://api.github.com/repos/{repo}/releases/latest"
    version = requests.get(url).json()["tag_name"]

    result["data"].append({
        "{#TOOL}": name,
        "{#VERSION}": version
    })

print(json.dumps(result))
```

#### No Zabbix:
Criar um item de discovery que executa o script.
Gerar itens filhos com chave do tipo:

```tool.version[{#TOOL}]```

- Triggers para detectar quando {#VERSION} mudar.

**Vantagem:** Escalável para dezenas de ferramentas.

**Limite:** Exige manutenção do script e API key do GitHub em casos de rate limit.

<p style="text-align: justify;">

*Lembre-se que a API do Github tem um limite de consultas e tu não vai atualizar versão a cada minuto. Sugestão, colocar intervalo de coleta 1 vez ao dia.*
</p>

Após isso, envie as notificações para onde preferir, basta ter a integração no zabbix. Ou crie um painel no Grafana.

![Lara](/assets/images/atualizacoes/01.png)


![Lara](/assets/images/atualizacoes/02.png)


![Lara](/assets/images/atualizacoes/03.webp)
