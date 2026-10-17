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
- [ ] Ampliação do histórico e investigação de categorias mais específicas
- [ ] Engenharia de atributos de clientes e artigos
- [ ] Investigação de segmentação de clientes
- [ ] Baselines de recomendação de artigos
- [ ] Modelos de recomendação e ranqueamento
- [ ] Validação temporal e comparação por MAP@12
- [ ] Avaliação de desempenho com a base completa

## Análise exploratória atual

O trabalho está concentrado no [etapa1.ipynb](etapa1.ipynb). O notebook integra transações, clientes e grupos de produtos no `df_main` e investiga:

- Relações entre preço, canal e medidas agregadas por cliente, usando Pearson e Spearman.
- Participação de cada `product_group_name` no histórico individual, com uma matriz cliente × categoria.
- Afinidade relativa: participação da categoria no cliente dividida pela participação da categoria nos itens da base.
- Concentração na categoria líder, empates e repetição em dias diferentes.
- Comparação entre a categoria líder individual e a categoria mais popular do histórico em um período posterior.

As contagens representam **itens vendidos, não pedidos distintos**. Categoria líder significa maior participação observada no histórico; não é uma preferência declarada pelo cliente. Uma categoria não comprada também não significa rejeição.

## Resultados preliminares

Resultados da execução documentada em **17/10/2026**, com as primeiras **1.000.000 de transações**. Esse recorte segue a ordem do arquivo e **não é uma amostra aleatória**.

Na EDA de preferências, foram excluídas 607 linhas com categoria desconhecida e 16.404 itens do último dia, possivelmente incompleto pelo limite de leitura. Restaram **982.989 itens, 212.497 clientes e 14 categorias**, entre **20/09/2018 e 10/10/2018**.

Para a checagem temporal, o histórico vai de **20/09 a 03/10/2018** e o período posterior de **04/10 a 10/10/2018**. A comparação usa os mesmos **4.110 clientes** nos dois critérios: pelo menos cinco itens em dois dias no histórico, categoria líder sem empate e alguma compra no período posterior. As preferências e a popularidade são calculadas apenas no histórico anterior.

| Medida no período posterior | Categoria líder individual | Categoria mais popular do histórico |
| --- | ---: | ---: |
| Clientes que compraram a categoria ao menos uma vez | 67,3% | 72,2% |
| Participação da categoria nos itens, em média por cliente | 46,3% | 50,6% |

**Nesta janela, a categoria líder individual não superou a popularidade geral nas duas medidas.** A comparação cobre apenas **5,7% dos clientes do período posterior**; não permite generalizar para toda a base nem descartar personalização. O próximo passo exploratório é ampliar o histórico, examinar recência e categorias mais específicas, como `product_type_name`.

As correlações fracas entre preço e canal também **não demonstram que preço seja irrelevante para a escolha do artigo**. Essa pergunta ainda precisa ser investigada.

## Avaliação

A métrica principal da competição é **MAP@12 (Mean Average Precision at 12)**, que avalia a qualidade e a ordem das recomendações para cada cliente.

As medidas exploratórias acima avaliam repetição de **categorias**. Não são MAP@12, não medem acerto de `article_id` e não representam desempenho de um modelo treinado.

## Dados

Os arquivos de origem devem ser obtidos diretamente na [página da competição no Kaggle](https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations/data). Os dados brutos da competição não são redistribuídos neste repositório.

Coloque os arquivos na mesma pasta do notebook:

| Arquivo | Uso |
| --- | --- |
| `transactions_train.csv` | Histórico de transações |
| `customers.csv` | Cadastro de clientes |
| `articles.csv` | Cadastro e categorias dos artigos |
| `sample_submission.csv` | Referência para a futura submissão; não usado na EDA atual |

## Como explorar

1. Prepare um ambiente Python com `numpy`, `pandas`, `pingouin`, `scikit-learn` e `plotly`, além de suporte a notebooks, como JupyterLab e `ipykernel`.
2. Abra [etapa1.ipynb](etapa1.ipynb) com a pasta deste projeto como diretório de trabalho, pois algumas leituras usam caminhos relativos.
3. Execute as células em ordem, desde os imports. Os gráficos interativos são produzidos com Plotly.
4. Na seção **Olhando aquele cliente**, altere `CLIENTE_ID` para um ID presente em `perfil_preferencia.index` e reexecute a célula para inspecionar seu perfil.

Parâmetros disponíveis no notebook:

| Parâmetro | Valor atual | Finalidade |
| --- | ---: | --- |
| `N_LINHAS` | `1_000_000` | Limite inicial de leitura; `None` lê o arquivo completo |
| `MIN_ITENS_PREFERENCIA` | `5` | Mínimo de itens para os perfis comparados |
| `MIN_DIAS_PREFERENCIA` | `2` | Mínimo de dias distintos com compras |
| `DIAS_POSTERIORES` | `7` | Janela posterior da checagem temporal |

Ao mudar os parâmetros, reexecute as células dependentes. O arquivo de transações tem aproximadamente **3,5 GB**, e sua representação em memória pode ocupar mais espaço. Alterar o recorte também altera os resultados e pode deixar histórico insuficiente para a checagem temporal. Os números deste README descrevem a execução indicada acima.

## Status

**Em desenvolvimento — fase de EDA.** O notebook foi executado do início ao fim no recorte atual. Ainda não há modelo de recomendação treinado, avaliação por MAP@12 ou submissão gerada. Os próximos experimentos serão documentados conforme forem realizados.
