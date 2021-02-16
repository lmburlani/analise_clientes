# analise_clientes
Usarei python para tratar um banco de dados e realizar uma análise com base em gráficos e achar o motivo de estarem causando prejuízos para a empresa / I'll use python to perform an analysis based on a company database and make graphs about it, and find why the losses are happening to the company
# objetivo desse código é usar uma base de dados de uma empresa de crédito que está tomando prejuízo pois os clientes estão cancelando os seus cartões
# então irei analisar a tendência de comportamento e como podemos evitar isso, começaremos importando a biblioteca pandas e ploty
import pandas as pd
import plotly.express as px

# importar a base de dados (em anexo no git, aqui irei importar direto do meu drive)
from google.colab import drive
drive.mount('/content/drive')

# iremos atribuir uma variável para ler o arquivo
clientes_df = pd.read_csv(r'/content/drive/MyDrive/análise dados python/ClientesBanco.csv', encoding='latin1')
clientes_df = clientes_df.drop('CLIENTNUM', axis=1)
display(clientes_df)

# tratamento e visualização geral dos dados
clientes_df = clientes_df.dropna()
display(clientes_df.info())
display(clientes_df.describe())

# irei separar os clientes e os cancelados
display(clientes_df['Categoria'].value_counts())
display(clientes_df['Categoria'].value_counts(normalize=True))

# irei gerar gráfiocos com ploty para melhor visualização
def grafico_coluna_categoria(coluna, tabela):
  fig = px.histogram(tabela, x=coluna, color='Categoria') # color_discrete_sequence=['#e6ad12', '#02b013']
  fig.show()
for coluna in clientes_df:
  grafico_coluna_categoria(coluna, clientes_df)

# observando os dados a gente nota-se que:
# 1 quem usa menos o cartão tem maior tendência a cancelar ele, os que tem menos transações
# 2 aparentemente a pessoa que entrou em mais contato, mais ela tende a cancelar, seria por causa de um atendimento ruim?
# 3 observando o perfil do cliente, os da categoria Blue tendem a cancelar mais, podemos ofertar outro plano que atende as suas necessidades?
# com base nessa análise sabemos que quem é Blue tem maior chance de cancelar, iremos focar nessa categoria e na quantidade de transações
cancelados_df = clientes_df.loc[clientes_df['Categoria'] == 'Cancelado', :]
fig = px.histogram(cancelados_df, x='Qtde Transacoes 12m', nbins=5)
fig.show()
fig = px.histogram(cancelados_df, x="Valor Transacoes 12m", nbins=7)
fig.show()

criticos_df = cancelados_df.loc[cancelados_df['Qtde Transacoes 12m'] < 60, :]
fig = px.histogram(criticos_df, x='Contatos 12m')
fig.show()

qtde_ultra_criticos = len(criticos_df.loc[criticos_df['Contatos 12m'] > 2, :])

percentual_criticos = qtde_ultra_criticos / len(cancelados_df)
print(f'{percentual_criticos:.1%}')

# vamos checar como os cancelados estão divididos
for coluna in cancelados_df:
  grafico_coluna_categoria(coluna, cancelados_df)
