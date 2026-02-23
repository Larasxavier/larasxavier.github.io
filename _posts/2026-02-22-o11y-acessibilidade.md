---
title: "Observabilidade com Acessibilidade"
description: "Uma visão prática sobre monitoramento e observabilidade, conectando dados técnicos a decisões estratégicas de negócio."
date: 2026-01-15 00:00:00 -0300
categories: [observabilidade, negocio]
tags: [observabilidade, monitoramento, sre, devops]
---

Já havia comentado em um artigo que dashboard que não responde um problema é apenas arte. Pesou?
 <p style="text-align: justify;">

Sua criação estratégica também precisa de um storytelling de dados, que apesar de não ser um mandamento direto e todos conseguirem criar um, nem todos conseguem interpretar da mesma forma. Quando isso se soma a uma questão de acessibilidade visual, as coisas pioram muito. É importante que você saiba contar uma história com cores, pois na correria do dia a constatação de que o ser humano é visual prevalesce. Não entendeu?
</p>

 <p style="text-align: justify;">

A neurociência é um estudo científico multidisciplinar do sistema nervoso (cérebro, medula espinhal e nervos periféricos), abrangendo sua estrutura, função, desenvolvimento e patologias e constata que o cérebro humano é predominantemente visual, tem uma taxa média de retenção de até 65% em 3 dias. Nosso cérebro não pensa em textos, pensa em imagens e padrões pois elas criam atalhos cognitivos, eduzem esforço mental e aceleram a recuperação da informação.
</p>

 <p style="text-align: justify;">

O FinOps da coisa é que quando trabalhamos isso em um dia cheio de telas e informações o tempo inteiro conseguimos ter uma economia de esforço cerebral. E se quiser ativar seu modo HA (Alta disponibilidade) combine imagem e palavra, assim criamos dois caminhos de memória: verbal e visual. Se um falhar, o outro recupera a informação!
</p>


## Storytelling de dados: Componentes

Para contar uma história aqui também não é diferente, preciso ter:

- Personagem: campos de dados analisados
- Enredo: o insight que surge da narrativa
- Narrativa: estilo usado para comunicar o insight

Deve-se ter um controle para transmitir o ponto de vista de forma conclusiva.

Dicas mais comuns:

- Reduza a saturação das cores para não tirar o foco do público diante da informação ou gerar confusão.
- Cores muito brilhantes podem competir e causar distração, use pelo menos 75~90%
- Redução de brilho acaba reduzindo também a carga cognitiva que seu público precisa lidar. Você pode destacar uma informação e manter as outras em escala de cinza, por exemplo.
- Tenha cuidado com associação de cores

Exemplos de visualizações:
- Mudanças ao longo do tempo;
- Determinando frequência;
- Determinando relacionamentos;

imagem do espectro de cores e emoções

## Combinações
 <p style="text-align: justify;">

A combinação de vermelho e verde tão famosa e usada por nós é um exemplo a ser levado a diante, cores distintas para trazer sensações de alívio ou preocupação.
</p>

 <p style="text-align: justify;">

Cinza normalmente dá sensação de uniformidade e calma podendo ser usada como complementar para outras informações e focar ainda mais no que não é cinza.
</p>

### Consistência de cores
 <p style="text-align: justify;">

Para repetir a ideia com mais de um gráfico ajudando o público a distinguir,se ocorrer mudança significará alteração de ideia.
</p>

 <p style="text-align: justify;">

A importância de escolher a de escolher a cor certa, se deve a estudos comprovarem que mais de 50% dsa pessoas que optam sair de um site nunca mais retornam devido as escolhas de cores de design.
</p>


### Conheça seu público
 <p style="text-align: justify;">

Cores podem significar cores diferentes em culturas distintas, considere associações de cores por setor, cores da marca. Se ela for laranja e verde..faça o jogo de cores.
</p>


Existem 3 razões para as cores da sua marca não funcionarem também na visualização.

- contraste
- quantidade de cores dispersas
- adequações dos dados

### Paleta de cores 
 <p style="text-align: justify;">

- Faça harmonia análoga: Cores vizinhas se complementam e ninguém sobressai (Trace a linha no circulo cromático)
</p>


![Obs](/assets/images/daltonismo/Combinacao_harmonica_das_cores.jpg)


- Use complementares com conotação positiva/negativa (evite a cor da marca como negativo)
- Use quase complementares para destacar, por exemplo, duas séries onde uma é o foco principal. Chamam também de regra 33%
- Evite fundos coloridos demais;
- Conheça seus dispositivos para dar contexto de utilização
- Utilize gradientes (!!!!) para quando tiver dificuldade para comparar e contrastar dados. Evite para dados categóricos pois pode causar confusão.

## Acessibilidade e Daltonismo 
 
 Daltonismo é a deficiência de visão cromática de uma forma mais bruta. 
 <p style="text-align: justify;">

 Aproximadamente 1 em cada 12 homens e 1 em cada 200 mulheres apresentam alguma forma de deficiência de visão cromática. A maioria das pessoas ainda percebem as cores, mas são transmitidas com codificação diferente.

</p>

 Como identificar a real?

Apps que podem te auxiliar

1. Ferramenta Coblis Color Blindness Simulator
2. Color Oracle

**ATENÇÃO** 

### Combinações a evitar

- vermelho, verde e marrom: podem ser indistinguíveis (tons de marrom e amarelo escuro também)
- Rosa, turquesa e cinza: todos parecem cinza para quem tem daltonismo com vermelho
- Roxo e azul: pode parecer só azul.

### Melhores práticas

- Azul e Laranja são um ótimo ponto de partida.
- Preto e branco é uma combinação que não tem erro. Faça um versão em preto e branco para ver se a visualização e distinção faz sentido como prova dos 9.
- Luminosidade da cor ajuda.
- Combine elementos como linhas, formas, simbolos.


### Armadilhas comuns do uso de cores no storytelling

- Adcionar informações irrelevantes ou em excesso
(Regra dos 3 a 5 categorias de cores)

- Usar cores não monotônicas para valores de dados
A diferença de cores deve refletir a diferença de valores nos dados

- Criar codficações de dados que não levam em consideração pessoas com deficiência cromática.

- Não criar associação com cores
Crie um padrão ou legenda para seu seu público leia o seu trabalho

- Não usar cores contrastantes para contrastar informações
cores e número tem semelhança.

- Não destacar informações importantes
Seu trabalho não é ser imparcial

- Usar muitas cores: O cérebro sofre para processar tanta informação.
Pesquisas apontam que 7 é o número máximo de itens que o cérebro pode armazenar por vez. Razão pelo qual a maioria dos números de telefone tem 7 digitos hahaha nada é por acaso.

- Não usar mapa de calor


*Sim, como boa pesquisadora de neurociências eu vou ter boas referências. hahaha*

**Teoria do Duplo Código (base do visual + verbal)**

- PAIVIO, Allan. Imagery and verbal processes. New York: Holt, Rinehart and Winston, 1971.

- PAIVIO, Allan. Mental representations: a dual coding approach. New York: Oxford University Press, 1986.

**Superioridade da Imagem (Picture Superiority Effect)**

- STANDING, Lionel. Learning 10,000 pictures. Quarterly Journal of Experimental Psychology, London, v. 25, n. 2, p. 207–222, 1973. DOI: 10.1080/14640747308400340.

**Aprendizagem multimídia e retenção visual**

- MAYER, Richard E. Multimedia learning. 2. ed. New York: Cambridge University Press, 2009.

- MAYER, Richard E. Applying the science of learning to medical education. Medical Education, v. 44, n. 6, p. 543–549, 2010.

**Neurociência, memória e processamento visual**

- MEDINA, John. Brain rules: 12 principles for surviving and thriving at work, home, and school. Seattle: Pear Press, 2008.

- KOSSLYN, Stephen M.; GANIS, Giorgio; THOMPSON, William L. Neural foundations of imagery. Nature Reviews Neuroscience, London, v. 2, n. 9, p. 635–642, 2001.

**Processamento visual e eficiência cognitiva**

- WARE, Colin. Information visualization: perception for design. 3. ed. Waltham: Morgan Kaufmann, 2013.
LIDWELL, William; HOLDEN, Kritina; BUTLER, Jill. Universal principles of design. Beverly: Rockport Publishers, 2010.

**A cor dos dados**

- STRACHNYI, Kate. Color-wise: a data storyteller’s guide to the intentional use of color. Hoboken: Wiley, 2020.