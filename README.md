# Detecção de Fraudes em Cartões de Crédito

Pipeline de Machine Learning para identificar transações fraudulentas em um dataset
extremamente desbalanceado, usando o dataset [Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)
(mlg-ulb), disponível no Kaggle.

O dataset original contém 284.807 transações de cartões de crédito europeus registradas em
setembro de 2013, das quais 492 (0,17%) são fraudulentas. A execução deste projeto foi feita
sobre uma amostra de 17.918 transações, das quais 81 são fraudulentas (0,45%), mantendo o
mesmo desafio de desbalanceamento extremo entre as classes. O objetivo é construir um modelo
capaz de identificar essas fraudes sem gerar um volume inviável de falsos alarmes.

## Resultados

Avaliação no conjunto de teste (3.584 transações, 16 fraudes):

| Modelo                                  | Precision (fraude) | Recall (fraude) | F1-score (fraude) |
|------------------------------------------|:---:|:---:|:---:|
| Regressão Logística (baseline)           | 0,250 | 1,000 | 0,400 |
| Random Forest (class_weight)             | 0,882 | 0,938 | 0,909 |
| Random Forest + SMOTE                    | 0,800 | 1,000 | 0,889 |
| Random Forest + threshold ajustado       | 0,938 | 0,938 | 0,938 |

A acurácia não é usada como métrica de avaliação porque, com mais de 99% das transações
concentradas em uma única classe, ela não diferencia um bom modelo de um modelo que
simplesmente prevê "não fraude" sempre. A métrica usada para comparar os modelos é a PR-AUC
(Precision-Recall AUC), mais adequada do que ROC-AUC quando as classes são tão desbalanceadas.
O melhor resultado final foi obtido com a Random Forest após ajuste do threshold de decisão,
chegando a 93,8% de precisão e 93,8% de recall sobre a classe de fraude.

## Estrutura do notebook

O notebook `Deteccao_Fraude_Cartao_Credito.ipynb` está dividido em etapas, cada uma em células
separadas com explicação em markdown:

1. Introdução e contexto do problema
2. Configuração do ambiente
3. Carregamento dos dados (com fallback sintético caso o CSV não esteja presente)
4. Limpeza dos dados (nulos e duplicadas)
5. Exploração inicial
6. Análise do desbalanceamento de classes
7. Distribuição de Time e Amount por classe
8. Correlações entre variáveis e a classe-alvo
9. Pré-processamento (RobustScaler em Time/Amount)
10. Split treino/teste estratificado
11. Estratégias para desbalanceamento (class_weight e SMOTE)
12. Modelo baseline — Regressão Logística
13. Modelos avançados — Random Forest (com e sem SMOTE)
14. Comparação formal das métricas entre modelos
15. Ajuste de threshold de decisão
16. Importância das variáveis
17. Conclusões e recomendações de negócio

## Estrutura do repositório

```
.
├── Deteccao_Fraude_Cartao_Credito.ipynb
├── creditcard.csv          (não incluído no repositório, ver abaixo)
├── requirements.txt
└── README.md
```

## Como executar

Clone o repositório:

```bash
git clone https://github.com/<seu-usuario>/<seu-repositorio>.git
cd <seu-repositorio>
```

Crie um ambiente virtual e instale as dependências:

```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Baixe o dataset. O arquivo `creditcard.csv` não está incluído neste repositório por ter cerca
de 150 MB e por questões de licença do Kaggle. Baixe manualmente em
https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud ou via Kaggle API:

```bash
kaggle datasets download -d mlg-ulb/creditcardfraud
unzip creditcardfraud.zip
```

Coloque o `creditcard.csv` na raiz do repositório, na mesma pasta do notebook. Os resultados
reportados acima foram obtidos com uma amostra de 17.918 linhas do dataset original; usando o
arquivo completo (284.807 linhas) os números podem variar um pouco, mas a tendência entre os
modelos se mantém.

Se o arquivo não for encontrado, o notebook gera automaticamente um dataset sintético
equivalente apenas para demonstrar o pipeline de ponta a ponta. Nesse caso os resultados são
apenas ilustrativos e não representam o dataset real.

Execute o notebook:

```bash
jupyter notebook Deteccao_Fraude_Cartao_Credito.ipynb
```

## Tecnologias utilizadas

- Python 3
- pandas e numpy para manipulação de dados
- matplotlib e seaborn para visualização
- scikit-learn para modelagem e métricas
- imbalanced-learn para oversampling (SMOTE)

## Decisões técnicas

- Split estratificado (`stratify=y`) para preservar a proporção de fraudes em treino e teste,
  evitando avaliações instáveis.
- RobustScaler em vez de StandardScaler para Amount, por ser menos sensível a outliers.
- SMOTE aplicado apenas no conjunto de treino, após o split, para não vazar informação e
  inflar artificialmente as métricas.
- PR-AUC como métrica de seleção de modelo, em vez de acurácia ou ROC-AUC isoladas.
- Threshold de decisão ajustável, já que o limiar padrão de 0,5 raramente é o ideal em
  problemas com classes desbalanceadas. O ponto de corte deve refletir o custo de negócio entre
  falso positivo e falso negativo.

## Limitações

As variáveis V1 a V28 são componentes resultantes de uma transformação PCA aplicada pelos
autores originais do dataset por motivos de confidencialidade, o que limita a interpretabilidade
direta de negócio.

O dataset cobre apenas 2 dias de transações de um conjunto específico de cartões europeus em
2013. Um modelo de produção real precisaria ser revalidado com dados mais recentes e diversos,
considerando que o comportamento de fraudadores muda com o tempo.

## Licença

Projeto disponibilizado para fins educacionais e de portfólio. O dataset utilizado é de autoria
de Andrea Dal Pozzolo, Olivier Caelen, Reid A. Johnson e Gianluca Bontempi, disponível no Kaggle
sob a licença descrita na página original do dataset.
