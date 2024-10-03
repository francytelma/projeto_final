# Projeto Final


## Descrição


Esta é a primeira parte do Projeto Final realizado como atividade semanal da décima sétima semana do curso de Python: análise de dados, solicitado pela professora Patrícia Bongiovanni da parte de Projeto Guiado.

O projeto faz uso do dataset listings.csv, retirado do site [Inside Airbnb](https://insideairbnb.com/get-the-data/) para análise. 

O conjunto de dados do Airbnb contém informações reais e atualizadas sobre 34.664 hosts(acomodações) do serviço locação na cidade do Rio de Janeiro. Cada registro inclui detalhes como Nome do host, Id do host, Nome do locatário, Tipo de acomodação, Preço, Minimo de noites, Numeros de avaliações, Disponibilidade anual. Além disso, estão disponíveis também dados geográficos, como Bairro, Latitude, Longitude.
Esse conjunto de dados foi criado para que as comunidades sejam capacitadas com conhecimentos e informações para compreender, decidir e controlar o papel do aluguel de casas residenciais a turistas, abrangendo diferentes perfis demográficos, tipos de locação e padrões de uso. Ele é ideal para análises que buscam identificar comportamentos, preferências e interações dos usuários com a plataforma online. Essas informações podem ser usadas  identificar tendências, criar campanhas de marketing direcionadas e melhorar a experiência do usuário na plataforma.

Vale ressaltar que este conjunto de dados por trás do site Inside Airbnb são provenientes de informações publicamente disponíveis no site Airbnb.

## Implementação


O primeiro passo foi baixar todas a bibliotecas usadas para o projeto.

```
import pandas as pd
import matplotlib.pyplot as plt 
import seaborn as sns
import numpy as np
```
Logo em seguida, foi realizada uma ETL, coletando-se os dados do arquivo .csv já baixado na máquina. Apos essa primeira etapa, foi realizado o processo de verificação e transformação de dados para que pudessem atender aos requisitos de qualidade e estrutura necesários para análise.
```
A principio identificar a existencia de nulos , duplicados, valores faltantes
```
nulos_por_colunas = df.isnull().sum
print(nulos_por_colunas)
```
df_dados['price'] = df_dados['price'].fillna(df_dados['price'].mean()), onde foi estabelecida a média da coluna price para os valores NaN da coluna
```
df_dados = df_dados.drop(columns=["neighbourhood_group","license"]), onde foi excluída colunas do df que não seriam de uso para analise
```
df_dados = df_dados.fillna(0) Substituindo NaN pelo valor 0
```
def visualizar_as_duplicadas(df):  
    duplicados = df[df.duplicated(keep=False)]

    return duplicados

linhas_duplicadas = visualizar_as_duplicadas(df)
print (linhas_duplicadas)
```
info_df = df_dados.info()
print(info_df)
```
Após a verificação de dados nulos e duplicados, exclusão das coluna de "neighbourhood_group" e "license" para fins didáticos foi renomeadas todas as colunas restantes para o idioma português.

df_dados.rename(columns={              
'name':'Nome',                           
'host_id': 'Id do host',                          
'host_name': 'Nome do host',                        
'neighbourhood': 'Bairro',                       
'latitude': 'Latitude',                        
'longitude': 'Longitude',                      
'room_type': 'Tipo de acomodação',                           
'price':'Preço',                              
'minimum_nights':'Minimo de noites',                       
'number_of_reviews':'Numeros de avaliações',                   
'last_review':'Ultima avaliação',                      
'reviews_per_month':'Avaliações mensais',                   
'calculated_host_listings_count':'Listagens', 
'availability_365':'Disponibilidade anual',                   
'number_of_reviews_ltm':'Numero de avaliações anuais' 
}, inplace=True)
```
Foi feito o arredondamento das casas decimais da coluna 'Preço' para apenas uma casa
df_dados['Preço'] = df_dados['Preço'].round(1)

Em seguida, foi feito o arredondamento para baixo de valores iguais a 731.2 na coluna 'Preço', visto que essa era a média que substituiu os valores faltantes na coluna e se diferenciava dos demais dados com numero diferente de 0 depois da vírgula:
df_dados['Preço'] = df_dados['Preço'].apply(lambda x: np.floor(x) if x == 731.2 else x)
```
Depois de feita toda limpeza de dados foi criado um novo dataframe.
df_dados.to_csv("Projeto_airbnb_rj_tratadoII.csv",index=False)
```
## Análise

#O primeiro questionamento analisado foi a somatória total de acomodações por bairros.

contagem_bairros = df_dados["Bairro"].value_counts()
print (contagem_bairros)
```
Obter os 10 bairros com mais disponibilidade de acomodações 
top_10 = contagem_bairros.nlargest (10, keep='all')
print(top_10)
```
# Qual a média dos preços e a variação por bairro?

Antes foi descoberto o preço médio total
preco_medio_total = df_dados['Preço'].mean()
print("\nPreço médio total:", preco_medio_total)
```
#Agora o preço médio por bairro

preco_medio_por_bairro = df_dados.groupby('Bairro')['Preço'].mean()
print(preco_medio_por_bairro)
```
A coluna 'Preço' não estava em tipo numérico. Para uma posterior merge entre 'top_10' e 'preço_medio_por_bairro' era necessaria essa transformação.
df_dados['Preço'] = pd.to_numeric(df_dados['Preço'], errors='coerce')

Calculando o preço médio dos 10 bairros com mais acomodaçoes:
resultado = pd.merge(top_10, preco_medio_por_bairro, on='Bairro')
print(resultado)
```
Depois foi só ordenar os bairros com valores médios mais altos pra melhor visualização
resultado_ordenado = resultado.sort_values(by='Preço', ascending=False)
print(resultado_ordenado)
```
# Criação do primeiro grafico (Média de preço entre os bairros e a comparação com a média total)
```
Grafico de Barras
plt.figure(figsize=(10, 6))
sns.barplot(x='Bairro', y='Preço', data=resultado_ordenado)
plt.axhline(preco_medio_total, color='red', linestyle='--', label='Preço Médio Total') #média de preços total
plt.xlabel('Bairros')
plt.ylabel('Preço Médio')
plt.title('Preço Médio das Acomodações por Bairro')
plt.xticks(rotation=80)
plt.show()
``` 
# Criação do segundo gráfico (Quantidade de acomodações por bairro)
```
Gráfico de Barras

plt.figure(figsize=(10, 6))
sns.barplot(x='Bairro', y='count', data=resultado_ordenado, color='#6A5ACD')
plt.xlabel('Bairros')
plt.ylabel('count')
plt.title('Quantidade de Acomodações por Bairro')
plt.xticks(rotation=80)
plt.show()
```
# Criação do terceiro gráfico (A quantidae de Airbnb por bairro influencia no preço da acomodação?)
```
Gráfico de Barras Comparativo

bairros = ['Barra da Tijuca','Leblon','Ipanema','Recreio dos Bandeirantes','Copacabana','Jacarepaguá','Santa Teresa','Botafogo','Flamengo','Centro']
contagem = [3242,1622,3299,1730,10783,1541,1116,1466,795,1311]
precos = [965.274954, 918.362762, 827.791331, 636.331908, 624.498210, 585.144711, 475.548387, 469.564120, 440.642516, 315.228528]

largura_barra = 0.5
espacamento = 0.2
posicao1 = np.arange(len(bairros))* (largura_barra * 2 + espacamento)
posicao2 = [p + largura_barra for p in posicao1]

plt.figure(figsize=(12, 6))
plt.bar(posicao1, contagem, width=largura_barra, label='Quantidade de acomodações')
plt.bar(posicao2, precos, width=largura_barra, label='Preço médio')

plt.xlabel('Bairros')
plt.ylabel('Valores')
plt.title('Comparação Distribuição X Preço Médio por Bairro')
plt.xticks([p + largura_barra/2 for p in posicao1], bairros, rotation=90)

plt.legend()
plt.show()
```
Descobrindo o total de acomodações em todos os bairros
tipo_acomodação = df_dados["Tipo de acomodação"].value_counts()
print (tipo_acomodação)
```
#Criando o quarto gráfico para visualização do total das acomodações e a porcentagem referente a cada uma delas.
```
Gráfico de Pizza
plt.figure(figsize=(12, 7))

plt.pie(tipo_acomodacao, labels=None, startangle=65, 
        colors=sns.color_palette("pastel"))   

Adicionar uma legenda com as porcentagens

porcentagens = [f'{label}: {val} ({val / tipo_acomodacao.sum() * 100:.1f}%)' 
                for label, val in zip(tipo_acomodacao.index, tipo_acomodacao)]

plt.legend(porcentagens, title="Tipo de Acomodação", loc="lower right", bbox_to_anchor=(1.4, 0))
plt.title('Distribuição dos Tipos de Acomodação')
plt.show()
```
Descobrindo a média de noites minimas exigidas pelo locatários
media_noite_exigida = df_dados.groupby('Tipo de acomodação')['Minimo de noites'].mean()
print(media_noite_exigida)
```
Como Copacabana foi o bairro com mais acomodações, foi feito um filtro apenas das acomodações desse bairro 
copacabana_acomodações = df_dados[df_dados['Bairro'] == 'Copacabana']
tipo_count = copacabana_acomodações['Tipo de acomodação'].value_counts()
print(tipo_count)
```
Qual host apresenta mais numeros de feedbacks?

maximoavaliações = df_dados["Numeros de avaliações"].max()
print("O host com mais avaliaçõs tem:",maximoavaliações)
```
Depois de descoberto o numero máximo de feedbacks, a pergunta era que host é esse com essa quantidade de opiniões, onde esta situado, qual o tipo de acomodação e o preço?

linhas_max_avaliações = df_dados[df_dados["Numeros de avaliações"] == maximoavaliações]
print(linhas_max_avaliações[["Nome", "Bairro","Tipo de acomodação","Preço"]])
```
Qual host é o mais barato?

df_dados_sem_zero = df_dados[df_dados["Preço"] > 0] 
maisbarato = df_dados_sem_zero["Preço"].min()
host_mais_barato = df_dados_sem_zero[df_dados_sem_zero["Preço"] == maisbarato]
print(host_mais_barato[["Bairro", "Minimo de noites", "Preço"]])
```
Qual host é o mais caro?

df_dados_sem_zero = df_dados[df_dados["Preço"] > 0]
maiscaro = df_dados_sem_zero["Preço"].max()
host_mais_caro = df_dados_sem_zero[df_dados_sem_zero["Preço"] == maiscaro]
print(host_mais_caro[["Bairro", "Minimo de noites", "Preço"]])







































