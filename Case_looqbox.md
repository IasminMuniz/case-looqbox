```python
#IMPORTANDO BIBLIOTECAS
import pandas as pd
from sqlalchemy import create_engine

# Conexão direta com o servidor
engine = create_engine("mysql+pymysql://looqbox-challenge:looq-challenge@35.199.115.174/")

# Bancos de dados 
df_databases = pd.read_sql("SHOW DATABASES;", engine)

print(df_databases)
```

********** TESTE SQL **********


```python
#1.	Quais são os 10 produtos mais caros da empresa?

dez_prod_maior_valor= pd.read_sql_query("SELECT PRODUCT_NAME, PRODUCT_VAL FROM `looqbox-challenge`.data_product ORDER BY PRODUCT_VAL DESC LIMIT 10", engine)
print(dez_prod_maior_valor)

# Criei uma querie com a tabela produtos ordenando os mesmos em forma decrescente por valor limitando aos 10 mais caros.
```


```python
#2.	Quais seções os departamentos 'BEBIDAS' e 'PADARIA' possuem?

secoes_dept_padaria= pd.read_sql_query("SELECT SECTION_NAME AS 'SEÇÕES PADARIA' FROM `looqbox-challenge`.data_product WHERE DEP_NAME = 'PADARIA' GROUP BY SECTION_NAME", engine)
print(secoes_dept_padaria)

secoes_dept_bebidas= pd.read_sql_query("SELECT SECTION_NAME AS 'SEÇÕES BEBIDAS' FROM `looqbox-challenge`.data_product WHERE DEP_NAME = 'BEBIDAS' GROUP BY SECTION_NAME", engine)
print(secoes_dept_bebidas)

#Criei duas queries com a tabela produto para apresentas as seções dos departamentos solicitados separadamente.
```


```python
# 3. Qual foi a venda total de produtos (em $) de cada Área de Negócio (Business Area) no primeiro trimestre de 2019?"

vendas_primeiro_trimestre_areas = """ 
SELECT 
    lojas.BUSINESS_NAME AS 'Área de Negócio',
    SUM(vendas_prod.SALES_VALUE * vendas_prod.SALES_QTY) AS 'Total Numero', 
    
    CONCAT('R$ ', FORMAT(SUM(vendas_prod.SALES_VALUE * vendas_prod.SALES_QTY), 2, 'pt_BR')) AS 'Venda Total'
    
FROM `looqbox-challenge`.data_product_sales AS vendas_prod 

INNER JOIN `looqbox-challenge`.data_store_cad AS lojas
    ON vendas_prod.STORE_CODE = lojas.STORE_CODE

WHERE vendas_prod.DATE BETWEEN '2019-01-01' AND '2019-03-31'
GROUP BY lojas.BUSINESS_NAME
ORDER BY SUM(vendas_prod.SALES_VALUE * vendas_prod.SALES_QTY) DESC;
"""

df = pd.read_sql_query(vendas_primeiro_trimestre_areas, engine)
print(df)

#Realizei um INNER JOIN entre as tabelas vendas por produtos e por área para, criando duas colunas que somavam o total de vendas entre as datas 01/01/2019... 
#até 31/03/2019, agrupando por área, a coluna 'Total Numero' mantive para agrupamento e a coluna 'Venda Total' para cumprir o requisito em R$ formatada. 

# OBS: Utilizado IA para transformação da coluna 'VENDA TOTAL' em "R$".
```

********** TESTE PYTHON **********


```python
import pandas as pd

def retrieve_data(product_code=None, store_code=None, date=None):
    # 1. Validação de segurança: se faltar algo, avisa e não quebra o fluxo com erro feio
    if product_code is None or store_code is None or date is None:
        print("Você esqueceu de preencher algum dos parâmetros obrigatórios!")
        return None
        
    query = f"""
    SELECT * FROM `looqbox-challenge`.data_product_sales
    WHERE PRODUCT_CODE = {product_code}
      AND STORE_CODE = {store_code}
      AND DATE BETWEEN '{date[0]}' AND '{date[1]}';
    """
    
    return pd.read_sql_query(query, engine)

my_data = retrieve_data(18, 1, ['2019-01-01', '2019-01-05'])

my_data    

# OBS: Utilizado a IA para entendimento do problema e resolução parcial da função como por exemplo, onde inserir a variável em SQL.
```


```python
# 1) The Dev Team was tired of developing the same old queries just varying the filters accordingly to their boss demands.

Query 1:
SQL
SELECT
      STORE_CODE,
      STORE_NAME,
      START_DATE,
      END_DATE,
      BUSINESS_NAME,
      BUSINESS_CODE
FROM data_store_cad
Query 2:
SQL
SELECT
        STORE_CODE,
        DATE,
        SALES_VALUE,
        SALES_QTY
FROM data_store_sales
WHERE DATE BETWEEN '2019-01-01' AND '2019-12-31'
Além disso, ele te deu este conjunto de instruções:
•	Use as queries exatamente como estão (não as modifique e não crie uma nova);
•	Por favor, filtre o período entre este intervalo específico: ['2019-10-01', '2019-12-31']
•	Nós precisamos desta visualização! Por favor, crie-a utilizando Python:
Loja	Categoria	TM
Bahia	Atacado	15.39
Bangkok	Posto	13.67
Belem	Proximidade	15.37
Berlin	Proximidade	15.39
Buenos Aires	Atacado	15.39
Chicago	Varejo	15.53
Dubai	Atacado	15.39
Hong Kong	Farma	26.35
London	Farma	28.99
Madri	Farma	29.03
Miami	Posto	13.67
New York	Proximidade	15.39
Paris	Proximidade	15.39
Rio de Janeiro	Farma	29.59
Roma	Varejo	15.39
Salvador	Atacado	15.39
Sao Paulo	Varejo	15.39
Sidney	Posto	13.67
Tokio	Varejo	15.39
Vancouver	Posto	13.67
(Nota: TM geralmente significa Ticket Médio, que é o Valor das Vendas dividido pela Quantidade de Vendas). 

```


```python
# 2) A brand new client sent you two ready-to-go queries. Those are listed below:

import pandas as pd

# 1. Executar as queries EXATAMENTE como o cliente enviou
query_1 = """
SELECT
      STORE_CODE,
      STORE_NAME,
      START_DATE,
      END_DATE,
      BUSINESS_NAME,
      BUSINESS_CODE
FROM `looqbox-challenge`.data_store_cad
"""

query_2 = """
SELECT
        STORE_CODE,
        DATE,
        SALES_VALUE,
        SALES_QTY
FROM `looqbox-challenge`.data_store_sales
WHERE DATE BETWEEN '2019-01-01' AND '2019-12-31'
"""

df_cadastro = pd.read_sql(query_1, engine)
df_vendas = pd.read_sql(query_2, engine)

# 2.
df_vendas['DATE'] = pd.to_datetime(df_vendas['DATE']) # Garante que está no formato data
df_vendas_filtro_data = df_vendas[(df_vendas['DATE'] >= '2019-10-01') & (df_vendas['DATE'] <= '2019-12-31')]

# 3.
df_junto = pd.merge(df_vendas_filtro_data, df_cadastro, on='STORE_CODE', how='inner')

# 4.
df_agrupado = df_junto.groupby(['STORE_NAME', 'BUSINESS_NAME']).agg({
    'SALES_VALUE': 'sum',
    'SALES_QTY': 'sum'
}).reset_index()

# 5.
df_agrupado['TM'] = round(df_agrupado['SALES_VALUE'] / df_agrupado['SALES_QTY'], 2)

# 6.
df_final = df_agrupado[['STORE_NAME', 'BUSINESS_NAME', 'TM']].rename(columns={
    'STORE_NAME': 'Loja',
    'BUSINESS_NAME': 'Categoria'})

 
df_final = df_final.sort_values(by='Loja').reset_index(drop=True)

df_final


# ETAPAS LISTADAS ABAIXO FORAM UTILIZADAS IA JUNTAMENTE COM O ENTENDIMENTO GERAL DO DESAFIO
# 3. Cruzamento de dados (Merge/Join) pelo STORE_CODE
# 4. Agrupar por Loja e Categoria e somar os valores numéricos
# 6. Selecionar e renomear as colunas para ficar idêntico ao pedido do cliente ---> SUGESTÃO DA IA <---
```


```python
# 3) Building your own visualization

import seaborn as sns

### LANÇAMENTO DE FILMES POR ANO ###

lancamento_por_ano= """ SELECT Year, COUNT(*) 'Lançamento por Ano' FROM `looqbox-challenge`.IMDB_movies GROUP BY Year ORDER BY Year ASC;"""
df = pd.read_sql_query(lancamento_por_ano, engine)
print(df)

plt.figure(figsize=(10, 6)) # Aumenta o tamanho para caber os nomes grandes
sns.lineplot(data=df, x='Year', y='Lançamento por Ano') # Ou y='Lançamento por Ano' dependendo da sua métrica
plt.title('Filmes lançados por Ano')
plt.xlabel('Ano')
plt.ylabel('Filmes Lançados')
plt.xticks(rotation=45, ha='right')
plt.show()


### DEZ FILMES MAIS VOTADOS ###

filme_mais_votado= """ SELECT Title, Votes 'Filme mais Votado' FROM `looqbox-challenge`.IMDB_movies ORDER BY Votes DESC LIMIT 10;"""
df = pd.read_sql_query(filme_mais_votado, engine)
print(df)

plt.figure(figsize=(10, 6)) # Aumenta o tamanho para caber os nomes grandes
sns.barplot(data=df, x='Title', y='Filme mais Votado') # Ou y='Lançamento por Ano' dependendo da sua métrica
plt.title('Dez filmes mais votados')
plt.xlabel('Filme')
plt.ylabel('Votos')
plt.xticks(rotation=45, ha='right')
plt.show()

# Apresentei duas visualizações gráficas, um como acompanhamento de Quantidade de filmes lançados por ano utilizando o gráfico de linhas... 
# e o outro como um ranking dos dez filmes com maior número de votos utilizando o gráfico de barras.
```
