<properties 
	pageTitle="Explorar dados na Máquina Virtual do SQL Server no Azure | Microsoft Azure" 
	description="Como explorar os dados armazenados em uma VM do SQL Server no Azure." 
	services="machine-learning" 
	documentationCenter="" 
	authors="bradsev" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/14/2016" 
	ms.author="fashah;garye;bradsev" />

#Explorar dados na Máquina Virtual do SQL Server no Azure


Este documento aborda como explorar os dados armazenados em uma VM do SQL Server no Azure. Isso pode ser feito por disputa de dados usando SQL ou usando uma linguagem de programação como Python.

O **menu** abaixo leva a tópicos que descrevem como usar ferramentas para explorar dados de vários ambientes de armazenamento. Esta tarefa é uma etapa no Processo de Análise da Cortana (CAP).

[AZURE.INCLUDE [cap-explore-data-selector](../../includes/cap-explore-data-selector.md)]


> [AZURE.NOTE] As instruções SQL de exemplo neste documento pressupõem que os dados estão no SQL Server. Se não estiverem, consulte o mapa do processo de ciência de dados de nuvem para aprender a mover seus dados para o SQL Server.



## <a name="sql-dataexploration"></a>Explorar dados de SQL com scripts SQL

Aqui estão alguns scripts de SQL de exemplo que podem ser usados para explorar armazenamentos de dados no SQL Server.

1. Obter a contagem de observações por dia

	`SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)`

2. Obter os níveis em uma coluna categórica

	`select  distinct <column_name> from <databasename>`

3. Obter o número de níveis na combinação de duas colunas categóricas

	`select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`

4. Obter a distribuição de colunas numéricas

	`select <column_name>, count(*) from <tablename> group by <column_name>`

> [AZURE.NOTE] Para ver um exemplo prático, você pode usar o [conjunto de dados de Táxis de NYC](http://www.andresmh.com/nyctaxitrips/) e ver o IPNB intitulado [Realizar o wrangling de dados de NYC usando o IPython Notebook e SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb) para obter um treinamento de ponta a ponta.

##<a name="python"></a>Explorar dados de SQL com o Python

Usar o Python para explorar dados e gerar recursos quando os dados estão no SQL Server é semelhante ao processamento de dados no blob do Azure usando o Python conforme documentado em [Processar dados de Blob do Azure em seu ambiente de ciência de dados](machine-learning-data-science-process-data-blob.md). Os dados precisam ser carregados do banco de dados para um quadro de dados pandas, quando então poderão ser processados. Documentamos o processo de conectar-se ao banco de dados e carregar os dados em um quadro de dados nesta seção.

O seguinte formato de cadeia de conexão pode ser usado para se conectar a um banco de dados do SQL Server do Python usando pyodbc (substitua servername, dbname, nome de usuário e senha pelos seus valores específicos):

	#Set up the SQL Azure connection
	import pyodbc	
	conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')

A [Biblioteca Pandas](http://pandas.pydata.org/) no Python fornece um conjunto avançado de estruturas de dados e ferramentas de análise de dados para manipulação de dados para a programação Python. O código a seguir lê os resultados retornados de um banco de dados do SQL Server em um quadro de dados Pandas:

	# Query database and load the returned results in pandas data frame
	data_frame = pd.read_sql('''select <columnname1>, <cloumnname2>... from <tablename>''', conn)

Agora, você pode trabalhar com o quadro de dados Pandas conforme abordado nos tópicos [Processar dados de Blobs do Azure em seu ambiente de ciência de dados](machine-learning-data-science-process-data-blob.md).

## Processo de Análise do Cortana no exemplo de ação

Para obter um exemplo passo a passo completo do Processo do Cortana Analytics usando um conjunto de dados público, confira [O Processo de Ciência de Dados de Equipe em ação: usando o SQL Server](machine-learning-data-science-process-sql-walkthrough.md).

 

<!---HONumber=AcomDC_0622_2016-->