# H&M Personalized Fashion Recommendations

Projeto de Ciência de Dados em desenvolvimento, baseado na competição [H&M Personalized Fashion Recommendations](https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations), do Kaggle.

## Objetivo

Construir e avaliar um sistema de recomendação de moda capaz de sugerir até 12 artigos por cliente, usando o histórico de transações e características de clientes e produtos para prever compras nos sete dias seguintes.

## Etapas

- [x] Preparação inicial e integração dos dados em um recorte de desenvolvimento
- [x] Visualizações exploratórias e estatísticas de preço
- [x] Correlações de Pearson e Spearman por item e por cliente
- [x] Exploração de preferências individuais por categoria de produto
- [x] Checagem temporal exploratória de repetição de categorias
- [x] Recorte de 2020 em parquet para iterar com o fim do histórico
- [x] Validação temporal em três semanas e implementação do MAP@12
- [x] Baselines de recomendação de artigos
- [ ] Geração de candidatos e medição de recall
- [ ] Engenharia de atributos de clientes e artigos
- [ ] Modelos de ranqueamento comparados na mesma validação
- [ ] Avaliação final na semana de teste reservada e submissão

## Notebooks

| Notebook | Conteúdo |
| --- | --- |
| [etapa1.ipynb](etapa1.ipynb) | Análise exploratória em um recorte do início do histórico (2018) |
| [etapa2.ipynb](etapa2.ipynb) | Recorte de 2020, régua de validação temporal, MAP@12 e baselines |

## Etapa 1 — análise exploratória

O notebook integra transações, cadastro completo de clientes e grupos de produtos no `df_main` e está dividido em cinco seções:

1. **Carga dos dados**: leitura e merges.
2. **Visão geral**: conferências, vendas por categoria, evolução diária e estatísticas de preço.
3. **Correlações de preço e canal**: Pearson e Spearman por item e por cliente.
4. **Preferência por categoria**: participação de cada `product_group_name` no histórico individual, matriz cliente × categoria e afinidade relativa (participação no cliente dividida pela participação na base).
5. **A preferência reaparece depois?**: comparação entre a categoria líder individual e a categoria mais popular do histórico em um período posterior.

As contagens representam **itens vendidos, não pedidos distintos**. Categoria líder significa maior participação observada no histórico; não é uma preferência declarada pelo cliente. Uma categoria não comprada também não significa rejeição.

### Resultados preliminares

Execução documentada em **17/10/2026**, com as primeiras **1.000.000 de transações**. O arquivo está em ordem de data: esse recorte cobre o início do histórico (set–out/2018) e **não é uma amostra aleatória**.

Na análise de preferências, foram excluídas 607 linhas com categoria desconhecida e 16.404 itens do último dia, possivelmente incompleto pelo limite de leitura. Restaram **982.989 itens, 212.497 clientes e 14 categorias**, entre **20/09/2018 e 10/10/2018**.

Para a checagem temporal, o histórico vai de **20/09 a 03/10/2018** e o período posterior de **04/10 a 10/10/2018**. A comparação usa os mesmos **4.110 clientes** nos dois critérios: pelo menos cinco itens em dois dias no histórico, categoria líder sem empate e alguma compra no período posterior. As preferências e a popularidade são calculadas apenas no histórico anterior.

| Medida no período posterior | Categoria líder individual | Categoria mais popular do histórico |
| --- | ---: | ---: |
| Clientes que compraram a categoria ao menos uma vez | 67,3% | 72,2% |
| Participação da categoria nos itens, em média por cliente | 46,3% | 50,6% |

**Nesta janela, a categoria líder individual não superou a popularidade geral nas duas medidas.** A diferença fica mais clara entre os 1.328 clientes cuja líder é diferente da categoria popular: 47,4% contra 62,7% de presença e 28,0% contra 41,3% de participação. Nos demais, os dois critérios coincidem por construção.

A comparação cobre apenas **5,7% dos clientes do período posterior** e usa um grupo amplo de produtos; não permite generalizar para toda a base nem descartar personalização em categorias mais específicas, como `product_type_name`.

As correlações fracas entre preço e canal (Pearson 0,116; Spearman 0,111 por item) também **não demonstram que preço seja irrelevante para a escolha do artigo**.

## Etapa 2 — validação e baselines

### Recorte

A competição pede a semana seguinte a **22/09/2020**, então a modelagem usa o fim do histórico. Na primeira execução, o notebook lê do CSV apenas as transações a partir de **01/06/2020** e grava `transactions_2020.parquet`: **5.151.470 itens, 599.002 clientes e 45.975 artigos**, até 22/09/2020.

### Validação temporal

Cada semana avaliada simula a competição: as recomendações usam apenas o histórico anterior a ela e são comparadas com as compras dentro dela.

| Ensaio | Histórico visível | Semana avaliada | Clientes avaliados |
| --- | --- | --- | ---: |
| Semana A | 01/06 a 25/08/2020 | 26/08 a 01/09/2020 | 80.253 |
| Semana B | 01/06 a 01/09/2020 | 02/09 a 08/09/2020 | 75.822 |
| Semana C | 01/06 a 08/09/2020 | 09/09 a 15/09/2020 | 72.019 |
| Teste | 01/06 a 15/09/2020 | 16/09 a 22/09/2020 | reservado |

A semana de teste não é usada para escolher modelos nem hiperparâmetros: ela será aberta uma única vez, no fim do projeto. O MAP@12 é calculado sobre os clientes que compraram na semana avaliada; quem não recebeu nenhum acerto entra com zero.

### Baselines

Regras sem aprendizado que definem a nota mínima para qualquer modelo:

| Baseline | Regra | Sem. A | Sem. B | Sem. C | Média |
| --- | --- | ---: | ---: | ---: | ---: |
| Recompra recente | últimos artigos comprados pelo cliente, completados com a última semana | 0,0194 | 0,0223 | 0,0248 | **0,0221** |
| Popular na faixa etária | 12 mais vendidos na última semana na faixa de idade do cliente | 0,0085 | 0,0091 | 0,0076 | 0,0084 |
| Popularidade da última semana | 12 mais vendidos nos 7 dias antes do corte | 0,0068 | 0,0067 | 0,0066 | 0,0067 |
| Popularidade geral | 12 mais vendidos em todo o histórico visível | 0,0029 | 0,0041 | 0,0034 | 0,0035 |

**Recompra recente** venceu nas três semanas. Na semana C, a diferença média de AP por cliente em relação à segunda colocada tem intervalo de 95% entre **0,0173 e 0,0191**. Popularidade recente supera a acumulada, e a maior parte dos clientes não recebe nenhum acerto, o que torna a geração de candidatos o próximo gargalo.

## Avaliação

A métrica principal da competição é **MAP@12 (Mean Average Precision at 12)**, que avalia a qualidade e a ordem das recomendações para cada cliente. Ela está implementada na etapa 2 e conferida com um exemplo calculado à mão.

Os critérios para escolher um modelo são: superar o melhor baseline nas três semanas de validação, com diferença por cliente cujo intervalo de confiança não cruze zero, antes de qualquer avaliação na semana de teste.

As medidas da etapa 1 avaliam repetição de **categorias**: não são MAP@12, não medem acerto de `article_id` e não representam desempenho de um modelo treinado.

## Dados

Os arquivos de origem devem ser obtidos diretamente na [página da competição no Kaggle](https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations/data). Os dados brutos da competição não são redistribuídos neste repositório.

Coloque os arquivos na mesma pasta dos notebooks:

| Arquivo | Uso |
| --- | --- |
| `transactions_train.csv` | Histórico de transações |
| `customers.csv` | Cadastro de clientes |
| `articles.csv` | Cadastro e categorias dos artigos |
| `sample_submission.csv` | Referência para a futura submissão; ainda não usado |

O `transactions_2020.parquet` é gerado pela etapa 2. Arquivos `.csv` e `.parquet` são ignorados pelo Git.

## Como explorar

1. Prepare um ambiente Python com `numpy`, `pandas`, `pyarrow`, `pingouin`, `scikit-learn` e `plotly`, além de suporte a notebooks, como JupyterLab e `ipykernel`.
2. Abra os notebooks com a pasta deste projeto como diretório de trabalho.
3. Execute as células em ordem, desde os imports. Os gráficos interativos são produzidos com Plotly.
4. Na etapa 1, seção **Olhando aquele cliente**, altere `CLIENTE_ID` para um ID presente em `perfil_preferencia.index` e reexecute as células para inspecionar seu perfil.
5. Na etapa 2, a primeira execução lê o CSV completo para gerar o parquet; as seguintes usam o parquet.

Parâmetros disponíveis:

| Notebook | Parâmetro | Valor atual | Finalidade |
| --- | --- | ---: | --- |
| etapa 1 | `N_LINHAS` | `1_000_000` | Limite de leitura das transações; `None` lê o arquivo completo |
| etapa 1 | `MIN_ITENS_PREFERENCIA` | `5` | Mínimo de itens para os perfis comparados |
| etapa 1 | `MIN_DIAS_PREFERENCIA` | `2` | Mínimo de dias distintos com compras |
| etapa 1 | `DIAS_POSTERIORES` | `7` | Janela posterior da checagem temporal |
| etapa 2 | `DATA_INICIO` | `2020-06-01` | Início do recorte salvo em parquet |
| etapa 2 | `CORTE_A`, `CORTE_B`, `CORTE_C` | `2020-08-26`, `09-02`, `09-09` | Início de cada semana de validação |
| etapa 2 | `CORTE_TESTE` | `2020-09-16` | Início da semana de teste reservada |

Ao mudar os parâmetros, reexecute as células dependentes. Para mudar `DATA_INICIO`, apague o parquet existente para que ele seja recriado. O arquivo de transações tem aproximadamente **3,5 GB**, e sua representação em memória pode ocupar mais espaço. Os números deste README descrevem as execuções indicadas acima.

## Status

**Em desenvolvimento — fase de validação e baselines.** Os dois notebooks foram executados do início ao fim. A régua de validação e os baselines estão prontos; ainda não há modelo treinado nem submissão gerada. O próximo passo é a geração de candidatos e a medição de recall.
