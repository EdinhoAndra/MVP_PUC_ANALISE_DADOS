# PUC - ANÁLISE DE DADOS E BOAS PRÁTICAS  
## Projeto: Modelagem da Excursão Contrária Futura e Aplicação em Estratégias de Trading com Machine Learning

---

## 📌 Descrição
Este projeto investiga a modelagem da **excursão contrária futura** do mini-índice futuro (WINFUT), isto é, o tamanho do movimento adverso que o preço tende a realizar antes da continuidade ou invalidação do sinal principal.  

O foco do trabalho não foi apenas prever a direção do mercado, mas compreender, modelar e operacionalizar a variável `future_mae_h1` como medida de risco tático e estrutura microdinâmica do movimento seguinte.

A proposta central consistiu em:

- transformar o alvo `future_mae_h1` de diferentes formas;
- analisar como essas transformações alteram a associação com as features;
- selecionar variáveis explicativas com base nessa estrutura;
- construir uma lógica condicional para estimar percentis da excursão contrária;
- integrar essa informação a um backtest com entradas escalonadas, stops e gestão operacional.

---

## 🎯 Objetivos
- Modelar a variável `future_mae_h1` como representação da excursão contrária futura.
- Comparar diferentes transformações do alvo, em especial:
  - diferenciação fracionária com transformação logarítmica;
  - diferenciação fracionária sem transformação logarítmica.
- Investigar como a transformação do alvo altera o padrão de correlação com as features.
- Identificar features mais aderentes à estrutura do fenômeno.
- Construir distribuições condicionais da excursão contrária a partir de combinações discretas de estados de mercado.
- Extrair percentis relevantes (`p10`, `p25`, `p50`, `p75`, `p90`) para uso operacional.
- Integrar um modelo direcional com um modelo/referência condicional de excursão contrária.
- Avaliar o impacto prático da modelagem da excursão na redução do stop loss e na eficiência operacional.

---

## 🧩 Dataset e Features
Os dados foram construídos a partir de preços do mini-índice futuro, com foco em estruturas derivadas de candles e estados de mercado.

### Alvo principal
- `future_mae_h1`: medida da excursão contrária futura.

### Transformações estudadas no alvo
- transformação logarítmica;
- diferenciação fracionária;
- combinação de log + fracdiff;
- comparação entre versões transformadas e não transformadas.

### Famílias de features utilizadas
- **Direção e regime**
  - `dir_1`
  - `renko_dir_ma9`
  - `renko_dir_ma20`
  - `renko_dir_ma50`
  - `renko_dir_ma200`

- **Scores e classes estruturais**
  - `classe_score_ma9`
  - `classe_score_ma20`
  - `classe_score_ma50`
  - `classe_score_ma200`

- **Distâncias e posição relativa**
  - `dist_bricks_ma9_binned`
  - `dist_ma_9_bricks_binned`
  - `dir_sum_3_x_abs_dist_ma_20_binned`

- **Probabilidades/lifts condicionais**
  - `lift_prob_down_h1_ma200_binned`
  - `lift_prob_down_h2_ma200_binned`

- **Outras features contínuas e discretizadas**
  - variáveis ligadas a intensidade, contexto estrutural e defasagens

---

## 🔬 Problema de Pesquisa
O problema central do trabalho foi investigar:

> Como a transformação do alvo `future_mae_h1` altera a estrutura de associação com as features e como essa informação pode ser convertida em melhoria operacional na execução de trades?

A hipótese central foi que a transformação do alvo não atua apenas como pré-processamento, mas redefine quais tipos de variáveis se tornam mais informativas para a modelagem.

---

## 📊 Principais Etapas Metodológicas

### 1. Engenharia e análise do alvo
Foram estudadas diferentes representações de `future_mae_h1`, com especial atenção para:

- redução de dominância dos extremos;
- estabilidade da distribuição;
- estacionariedade parcial;
- impacto na correlação com variáveis contínuas e categóricas.

### 2. Estudo de correlação e seleção de features
Foi observado que:

- a versão com **log + diferenciação fracionária** apresentou maior associação com variáveis categóricas/discretas de regime e direção;
- a versão **sem log** preservou associação mais forte com variáveis contínuas ligadas à magnitude.

Esse resultado serviu como base para orientar a seleção de features.

### 3. Construção de referência condicional da excursão
Foram geradas distribuições condicionais de `future_mae_h1` a partir de combinações discretas de estados de mercado, permitindo extrair:

- `p10_y`
- `p25_y`
- `p50_y`
- `p75_y`
- `p90_y`

Esses percentis passaram a representar níveis prováveis de excursão adversa em cada configuração de mercado.

### 4. Integração com backtest
O backtest passou a combinar:

- um **modelo principal direcional**, responsável por prever compra ou venda;
- uma **lógica condicional de excursão contrária**, responsável por:
  - escalonar entradas;
  - definir níveis prováveis de adversidade;
  - permitir a construção de um stop mais aderente ao comportamento esperado do preço.

---

## ⚙️ Estratégia Operacional no Backtest
A estratégia implementada utilizou:

- entrada obrigatória na abertura do sinal;
- entradas adicionais em níveis percentílicos da excursão prevista;
- reversão de posição quando surge sinal contrário;
- stop fixo;
- stop opcional baseado em percentis previstos da excursão contrária;
- fechamento intradiário.

### Camadas de entrada
Exemplo de estrutura:
- entrada inicial obrigatória;
- reforço em `p50`;
- reforço em `p25/p75`;
- reforço em `p10/p90`.

### Nova lógica de stop
Foi incorporado um stoploss opcional baseado em percentis previstos:

- para compra: acionado se o preço ultrapassar `x` pontos abaixo do centil inferior previsto;
- para venda: acionado se o preço ultrapassar `x` pontos acima do centil superior previsto.

---

## 📈 Principais Resultados e Conclusões
O trabalho mostrou que a modelagem da excursão contrária futura trouxe ganhos importantes sob dois aspectos:

### 1. Melhoria metodológica
A transformação do alvo alterou de forma clara o tipo de estrutura estatística visível no problema:

- **log + fracdiff** favoreceu variáveis categóricas e de regime;
- **sem log** favoreceu variáveis contínuas de magnitude.

Isso reforça a ideia de que a escolha da transformação do alvo é parte central da modelagem.

### 2. Melhoria operacional
A inclusão do modelo/referência de excursão contrária ajudou significativamente a:

- reduzir de forma mais racional o tamanho do stop loss;
- distinguir melhor entre oscilação adversa plausível e invalidação do sinal;
- melhorar a lógica de entradas escalonadas;
- criar a possibilidade de atingir resultados semelhantes com menor número de contratos, graças a uma execução mais eficiente e informacionalmente orientada.

Em vez de depender apenas de exposição bruta, a estratégia passou a explorar melhor o comportamento esperado da adversidade do preço.

---

## ⚠️ Limitações do Trabalho
Apesar dos avanços, o trabalho ainda não esgota o problema. Entre os principais pontos que ficaram em aberto:

- nem todas as features identificadas nos testes de correlação foram efetivamente testadas na modelagem e no backtest;
- faltou comparar, de forma mais sistemática, grupos distintos de features:
  - contínuas;
  - categóricas;
  - híbridas;
- faltou maior exploração de robustez temporal:
  - walk-forward;
  - subperíodos;
  - regimes de volatilidade;
- faltou testar mais amplamente:
  - parâmetros do stop por centil;
  - mínimo de amostra por combinação;
  - diferentes composições de contratos por camada;
  - mais estruturas de custo e slippage;
- faltou aprofundar a calibração probabilística dos percentis previstos.

---

## 🚀 Próximos Passos
- Testar as demais features extraídas dos estudos de correlação.
- Comparar sistematicamente grupos de features contínuas versus categóricas.
- Validar a robustez do modelo em regime walk-forward.
- Avaliar a cobertura empírica dos percentis previstos.
- Testar se a estratégia realmente mantém desempenho com menor número de contratos.
- Incorporar marcação a mercado mais detalhada para análise da equity curve.
- Expandir a análise para diferentes janelas temporais e diferentes estruturas de mercado.

---

## 🛠️ Tecnologias Utilizadas
- Python 3.11
- pandas
- numpy
- matplotlib
- scikit-learn
- lightgbm
- statsmodels
- optuna
- notebooks Jupyter / Google Colab

---

## ✅ Síntese Final
Este projeto mostrou que a excursão contrária futura pode ser tratada como um objeto legítimo de modelagem estatística e não apenas como um detalhe operacional. A principal contribuição foi demonstrar que a modelagem dessa variável melhora a tradução entre análise de dados e decisão prática, fornecendo uma base técnica para:

- seleção mais coerente de features;
- execução escalonada mais informada;
- stop loss mais aderente ao comportamento histórico;
- e potencial redução da necessidade de exposição excessiva por contrato.

Em termos acadêmicos, o trabalho reforça que a transformação do alvo altera a própria estrutura do problema e, em termos práticos, mostra que o modelo de excursão contrária é uma ferramenta promissora para aumentar a eficiência de estratégias de trading baseadas em Machine Learning.
