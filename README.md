# Detecção de Ocupação com Sensores IoT

Pipeline preditivo para identificar se uma sala está ocupada a partir de sensores ambientais, aplicado à automação predial e à eficiência energética na Indústria 4.0.

Projeto avaliativo do curso Integração de IA com Dados (SCTEC Aperfeiçoar).

## O problema

Um complexo industrial equipado com sensores IoT precisa saber, em tempo real, se suas salas técnicas e escritórios estão ocupados. Com essa informação, o ar-condicionado e a iluminação podem ser desligados automaticamente quando não há ninguém, reduzindo o consumo de energia.

O objetivo deste projeto foi treinar e comparar dois modelos de classificação (KNN e Árvore de Decisão) para prever a ocupação (`Occupancy`: 0 = desocupado, 1 = ocupado) a partir das leituras de temperatura, umidade, luminosidade e CO2.

## Base de dados

Foi utilizada a [Occupancy Detection Dataset](https://archive.ics.uci.edu/dataset/357/occupancy+detection) (UCI Machine Learning Repository), no arquivo `occupancy_completo.csv`, que reúne os três arquivos originais da base, com 20.560 registros.

| Variável | Descrição |
|---|---|
| Temperature | Temperatura do ambiente (°C) |
| Humidity | Umidade relativa do ar (%) |
| Light | Luminosidade (Lux) |
| CO2 | Concentração de dióxido de carbono (ppm) |
| HumidityRatio | Razão de umidade, derivada da temperatura e da umidade |
| Occupancy | Variável alvo (0 = desocupado, 1 = ocupado) |

Durante a análise, percebi que parte dos valores do arquivo veio com o ponto decimal fora do lugar (por exemplo, `272.675` em vez de `27.2675`), provavelmente por causa do formato regional da planilha de origem. Por isso, as colunas dos sensores foram lidas como texto. A correção foi feita no próprio notebook: em Temperature, Humidity, HumidityRatio e CO2, foi possível recolocar o decimal na posição correta com base na faixa de valores de cada sensor. Na Light, onde mais de uma posição era possível, os valores incorretos foram transformados em ausentes e tratados na etapa de limpeza.

## Etapas do pipeline

1. Análise Exploratória (EDA): dimensões, tipos de dados, ajuste de formato, resumo estatístico, gráficos (distribuição da variável alvo, histogramas, mapa de calor de correlação de Pearson e gráficos de dispersão) e análise dos padrões encontrados.
2. Limpeza e Tratamento: remoção de 1.443 linhas duplicadas, imputação dos 682 valores ausentes da Light pela mediana e análise de outliers com boxplots.
3. Feature Engineering: criação da variável `razao_co2_luz = CO2 / (Light + 1)`, escolhida após comparação com o `indice_calor_umidade`.
4. Divisão dos Dados: separação entre X e y e divisão em treino (80%) e teste (20%), com `stratify=y` e `random_state=42`.
5. Escalonamento: StandardScaler aplicado apenas aos dados do KNN, com `fit_transform` no treino e `transform` no teste.
6. Ajuste de Hiperparâmetros: KNN com K = 3, 5 e 7, e Árvore de Decisão com `max_depth` = 3, 5 e None, comparando a acurácia de treino e teste para identificar overfitting.
7. Avaliação e Veredito Final: comparação da acurácia dos melhores modelos no conjunto de teste.

## Resultados

| Modelo | Configuração | Acurácia no teste |
|---|---|---|
| KNN | K = 5 | 99,08% |
| Árvore de Decisão | max_depth = 5 | 97,78% |

A Árvore de Decisão sem limite de profundidade chegou a 100% no treino e caiu no teste, um caso claro de overfitting. Entre as configurações estáveis, o KNN com K = 5 teve o melhor desempenho, com acurácias de treino e teste praticamente iguais. Por isso, ele é o modelo recomendado para o sistema de automação predial.

## Tecnologias utilizadas

* [Python](https://python.org) 3.13
* [Pandas](https://pandas.pydata.org) e [NumPy](https://numpy.org): manipulação dos dados
* [Matplotlib](https://matplotlib.org) e [Seaborn](https://seaborn.pydata.org): visualizações
* [Scikit-learn](https://scikit-learn.org): divisão dos dados, escalonamento, modelos e métricas
* Jupyter Notebook (no VS Code)

## Estrutura do repositório

```
occupancy-detection-ml/
├── data/
│   └── occupancy_completo.csv
├── occupancy_detection.ipynb
├── requirements.txt
├── .gitignore
└── README.md
```

## Como executar

1. Clone o repositório:
   ```bash
   git clone https://github.com/AnaysaLopes/occupancy-detection-ml.git
   cd occupancy-detection-ml
   ```

2. Crie e ative um ambiente virtual (recomendado):
   * No Windows:
     ```bash
     python -m venv .venv
     .venv\Scripts\activate
     ```
   * No Linux/macOS:
     ```bash
     python3 -m venv .venv
     source .venv/bin/activate
     ```

3. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```

4. Abra o notebook `occupancy_detection.ipynb` no VS Code (com a extensão Jupyter) e execute todas as células em ordem (Run All).

A base de dados já está na pasta `data/` e é lida por caminho relativo, então não é preciso alterar nenhum caminho no código.

## Melhorias futuras

* Testar a remoção da `Humidity`, que tem correlação de 0,93 com a `HumidityRatio`, para verificar se a redundância afeta o KNN.
* Estimar os valores ausentes da Light a partir das leituras dos minutos vizinhos, em vez de usar a mediana.
* Avaliar o uso da hora do dia e do dia da semana, extraídos da coluna `date`, como novas variáveis.
* Complementar a acurácia com outras métricas, como precisão, recall e F1-score, já que as classes são desbalanceadas.

## Autora

Anaysa Pereira Lopes

* GitHub: [AnaysaLopes](https://github.com/AnaysaLopes)
* LinkedIn: [anaysa-lopes](https://www.linkedin.com/in/anaysa-lopes)
