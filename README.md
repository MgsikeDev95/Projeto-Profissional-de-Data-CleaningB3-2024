# Projeto de Limpeza e Preparação de Dados da B3 para Análise Financeira

## Objetivo

Mostrar minhas habilidades de data cleaning, organização e preparação de dados para uma análise financeira.  
Estamos lidando com o dataset oficial da B3. Transformarei esses dados em dados limpos e utilizáveis para análise e visualização.

## Introdução

Comecei abrindo os arquivos sobre o formato e o layout do arquivo no link:  
https://www.b3.com.br/pt_br/market-data-e-indices/servicos-de-dados/market-data/historico/mercado-a-vista/cotacoes-historicas/

Nessa mesma página, fiz o download do arquivo de Cotações Históricas de 2024.  
Como não tenho experiência no campo das finanças, fiz uma pesquisa para tentar selecionar os dados mais relevantes e interessantes para que eu pudesse fazer uso, extrair insights ou criar visualizações valiosas.

## Os que escolhi foram

- **Data**: Primordial para análises temporais, então é preciso organizar os dados no tempo. Além disso, é essencial para que possamos identificar certas tendências, períodos voláteis e sazonalidades.

- **Código de Negociação**: Permite a identificação dos ativos negociados. É fundamental para agrupamento dos dados por ativo e comparação entre eles.

- **Preço de Abertura, Máximo, Médio, Mínimo e de Fechamento**: Permitem o cálculo da variação diária (Fechamento - Abertura), da volatilidade intradiária (Máximo - Mínimo) e a construção de indicadores financeiros (médias móveis, RSI etc.). O preço médio ajuda na identificação da pressão de compra/venda e dá uma ideia do preço médio ponderado durante o dia útil.

- **Volume Total Negociado e Quantidade de Negócios**: Essenciais para avaliação da liquidez do ativo. Altas variações podem ser indicadores de movimento institucional ou eventos relevantes.

- **Código ISIN**: O código universal do ativo, útil para cruzamentos com bases externas de dados (diferentes da B3).

## Objetivo

Mostrar minha capacidade de limpar, preparar e estruturar dados financeiros reais para posterior exploração e análise visual.

## Descrição dos Dados

O layout dos dados da B3 está no formato fixed-width (vamos usar read_fwf), os campos vêm todos grudados, então é necessário usar o comprimento exato de cada campo.  
Um exemplo é o dado de data do pregão, que vai de 03 a 10, ou seja, são 8 caracteres de tamanho que precisamos considerar.  
Após a primeira visualização com df.head(), vi que continham 28 colunas, o que batia com os dados do layout da COTAHIST.

## Estratégia de Limpeza

Após verificar em mãos o layout e confirmar a existência das 28 colunas, decidi dar nomes mais acessíveis a elas.  
Percebi também que seria interessante remover linhas que não contêm dados de ações, pois não seriam relevantes para esse tipo de análise.  
Então, com a linha de código df = df[df['tipo_registro'] == 1].copy() fiz um filtro para manter apenas os registros de cotações, ou seja, os dados realmente relevantes.

O próximo passo foi diminuir o "ruído", deixar o dataset mais limpo. Fui imprimindo uma a uma com seus 10 primeiros valores para entender o estado atual.  
Após mais pesquisas sobre o layout e ajuda de IAs, cheguei a este outcome de colunas que deveriam ser descontinuadas com df.drop():

| Coluna                      | Motivo da Exclusão                                               |
|----------------------------|------------------------------------------------------------------|
| tipo_registro              | Já filtramos só os 01.                                           |
| codigo_bdi                 | Pouco usado para análise moderna.                               |
| prazo_mercado_termo        | Pouco útil aqui (apenas para contratos a termo).                |
| moeda_referencia           | Quase sempre BRL, sem variação.                                 |
| preco_exercicio            | Usado para opções — irrelevante aqui.                           |
| indicador_correcao         | Correção de preços — não é o foco.                              |
| data_vencimento            | Relevante apenas para opções e futuros.                         |
| fator_cotacao              | Quase sempre fixo — raramente usado.                            |
| preco_exercicio_pontos     | Também usado em derivativos — não aplicável.                    |
| numero_distribuicao        | Informação contábil interna — não útil aqui.                    |
| sigla_instrucao            | Instrução de negociação — irrelevante.                          |
| tipo_papel                 | Redundante — já temos `especificacao_papel`.                    |

Agora vou lidar com os tipos de dados, convertendo a data com pd.to_datetime usando format='%Y%m%d',  
converter os valores financeiros em float (e dividi-los por 100, pois vêm sem pontos decimais), iterando com um for,  
e converter colunas de contagem em int com .astype(int).

Chegou a hora de tratar os valores nulos (sinto que deveria ter feito isso antes).  
Com df.isnull().sum(), percebi que apenas especificacao_papel tinha 278.455 valores nulos.  
Como é uma coluna que mostra o tipo de ativo negociado, decidi não removê-la por enquanto.

Depois disso, fui padronizar strings e fazer limpeza de texto:  
remover espaços extras, verificar duplicatas e colocar tudo em maiúsculo.  
Escolhi um método legal e escalável: usei df.select_dtypes para pegar só colunas object e salvei em string_cols.  
Iterei com um Loop for: se a coluna for nome_empresa, coloquei apenas a primeira letra em maiúscula,  
e para as demais, tudo maiúsculo. Também usei isinstance(x, str) para garantir que só altero strings (evita erro com NaN).  
Depois disso, apenas printei string_cols e vi que tudo estava ok.

Verifiquei por duplicatas: 2688 no total. Usei df.drop_duplicates() para removê-las.

Para os valores nulos, usei fillna("NÃO INFORMADO") para padronizar.  
Mesmo que eu não use agora, mantenho caso queira usar depois.

Finalizei com print(df.describe()), print(df.info()), print(df.head()) para revisar o resultado.

Notei que alguns dados estavam indo para colunas erradas — erro no uso do layout — e corrigi relendo os widths.  
Após isso, os dados ficaram excelentes.

Reverifiquei duplicatas com df.duplicated().sum() e removi novamente, se necessário.

## Dataset Final Pronto para Análise

Usei print(len(df)) e verifiquei: 2.635.561 linhas após as correções.

##  Imagens 

Acima do arquivo cotahist-a24 tem o DATASET kaggleinputimage, onde tem varios prints que oram tirados durante esse processo.

## Conclusão

O projeto foi essencial para fortalecer meus conhecimentos em limpeza e padronização de dados financeiros brutos.  
Sinto que consegui dar mais clareza ao conteúdo, remover ruídos e preparar a base para uma futura análise exploratória.

Tive alguns desafios ao lidar com termos de ordem técnica e com um DataFrame dessa magnitude.  
Foi necessário realizar diversas consultas sobre o layout disponível no site da B3, além de pesquisar artigos que me ajudaram a tomar decisões quanto à relevância e ao tratamento adequado dos dados.
