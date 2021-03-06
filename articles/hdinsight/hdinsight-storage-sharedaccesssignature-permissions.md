<properties
pageTitle="Restringir o acesso a dados do HDInsight usando Assinaturas de Acesso Compartilhado"
description="Saiba como usar Assinaturas de Acesso Compartilhado para restringir o acesso do HDInsight a dados armazenados em blobs de armazenamento do Azure."
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
ms.service="hdinsight"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="big-data"
ms.date="07/05/2016"
ms.author="larryfr"/>

#Usar Assinaturas de Acesso Compartilhado do Armazenamento do Azure para restringir o acesso a dados com o HDInsight

O HDInsight usa os Blobs do Armazenamento do Azure para armazenamento de dados. Embora o HDInsight deva ter acesso completo ao blob usado como armazenamento padrão do cluster, você poderá restringir as permissões aos dados armazenados em outros blobs usados pelo cluster. Por exemplo, talvez queira tornar alguns dados em somente leitura. Faça isso usando as Assinaturas de Acesso Compartilhado.

As Assinaturas de Acesso Compartilhado (SAS) são recursos das contas de armazenamento do Azure que permitem que você limite o acesso aos dados. Por exemplo, oferecendo acesso somente leitura aos dados. Neste documento, você aprenderá a usar a SAS para habilitar o acesso de leitura e somente lista a um contêiner de blob do HDInsight.

##Requisitos

* Uma assinatura do Azure

* C# ou Python. O código de exemplo do C# é fornecido como uma solução do Visual Studio.

    * A versão do Visual Studio deve ser a 2013 ou 2015
    
    * A versão do Python deverá ser a 2.7 ou superior

* Um cluster HDInsight baseado em Linux OU o [Azure PowerShell][powershell]- se você tiver um cluster baseado em Linux existente, poderá usar o Ambari para adicionar uma Assinatura de Acesso Compartilhado ao cluster. Caso contrário, você poderá usar o Azure PowerShell para criar um novo cluster e adicionar uma Assinatura de Acesso Compartilhado durante a criação do cluster.

* Os arquivos de exemplo de [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature). Esse repositório possui o seguinte:

    * Um projeto do Visual Studio que pode criar um contêiner de armazenamento, a política armazenada e a SAS a ser usada com o HDInsight
    
    * Um script Python que pode criar um contêiner de armazenamento, a política armazenada e a SAS a ser usada com o HDInsight
    
    * Um script do PowerShell que pode criar um novo cluster HDInsight e configurá-lo para usar a SAS.

##As Assinaturas de Acesso Compartilhado

Há duas formas de Assinaturas de Acesso Compartilhado:

* Ad hoc: a hora de início, a hora de expiração e as permissões para a SAS são todas especificadas no URI SAS (ou implícitas, quando a hora de início é omitida).

* Política de acesso armazenada: uma política de acesso armazenada é definida em um contêiner de recurso - um contêiner de blob, uma tabela, uma fila ou um compartilhamento de arquivos - e pode ser usada para gerenciar as restrições de uma ou mais assinaturas de acesso compartilhado. Quando você associa uma SAS a uma política de acesso armazenada, a SAS herda as restrições - a hora de início, a hora de expiração e as permissões - definidas para a política de acesso armazenada.

A diferença entre as duas formas é importante para um cenário fundamental: revogação. Uma SAS é uma URL, portanto qualquer pessoa que obtiver a SAS poderá usá-la, independentemente de quem a tiver solicitado em primeiro lugar. Se uma SAS for publicada publicamente, ela poderá ser usada por qualquer pessoa no mundo. Uma SAS distribuída será válida até que ocorra um destes quatro fatores:

1. A hora de expiração especificada na SAS é atingida.

2. A hora de expiração especificada na política de acesso armazenada referenciada pelas SAS é atingida (se uma política de acesso armazenada for referenciada e especificar uma hora de expiração). Isso pode ocorrer porque transcorreu o intervalo ou porque você modificou a política de acesso armazenada para ter uma hora de expiração no passado, que é uma maneira de revogar a SAS.

3. A política de acesso armazenada referenciada pelas SAS é excluída, que é uma outra maneira de revogar a SAS. Observe que, se você recriar a política de acesso armazenada exatamente com o mesmo nome, todos os tokens SAS existentes serão novamente válidos de acordo com as permissões associadas a essa política de acesso armazenada (supondo que a hora de expiração não tenha passado na SAS). Se você estiver pretendendo revogar a SAS, certifique-se de usar um nome diferente se recriar a política de acesso com uma hora de expiração no futuro.

4. A chave de conta usada para criar as SAS é regenerada. Observe que isso causará uma falha de autenticação de todos os componentes do aplicativo que usam essa chave de conta até que eles sejam atualizados para usar a outra chave de conta válida ou a chave de conta recém-regenerada.

> [AZURE.IMPORTANT] Um URI de assinatura de acesso compartilhado é associado com a chave de conta usada para criar a assinatura e a política de acesso armazenado associada (se houver). Se nenhuma política de acesso armazenado for especificada, a única maneira de revogar uma assinatura de acesso compartilhado é alterar a chave da conta.

É recomendável que você sempre use as políticas de acesso armazenadas para que você possa revogar assinaturas ou estender a data de vencimento conforme necessário. As etapas deste documento usam políticas de acesso armazenado para gerar a SAS.

Para saber mais sobre as Assinaturas de Acesso Compartilhado, consulte [Noções básicas sobre o modelo de SAS](../storage/storage-dotnet-shared-access-signature-part-1.md).

##Criar uma política armazenada e gerar uma SAS

No momento, você deve criar uma política armazenada programaticamente. Você pode encontrar o exemplo em C# e em Python de criação de uma política armazenada e de SAS em [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature).

###Criar uma política armazenada e uma SAS usando C#

1. Abra a solução no Visual Studio.

2. No Gerenciador de Soluções, clique com o botão direito do mouse no projeto __SASToken__ e selecione __Propriedades__.

3. Escolha __Configurações__ e adicione valores às seguintes entradas:

    * StorageConnectionString: a cadeia de conexão da conta de armazenamento para a qual você deseja criar uma política armazenada e uma SAS. O formato deve ser `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey` onde `myaccount` é o nome de sua conta de armazenamento e `mykey` é a chave da conta de armazenamento.
    
    * ContainerName: o contêiner na conta de armazenamento para o qual você deseja restringir o acesso.
    
    * SASPolicyName: o nome a ser usado para a política armazenada que será criada.
    
    * FileToUpload: o caminho para um arquivo que será carregado no contêiner.
    
4. Execute o projeto. Será exibida uma janela de console e as informações semelhantes às seguintes serão exibidas depois que a SAS tiver sido gerada:

        Container SAS token using stored access policy: sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14
        
    Salve o token de política da SAS pois você precisará dele ao associar a conta de armazenamento ao seu cluster HDInsight. Você também precisará do nome da conta de armazenamento e do nome do contêiner.
    
###Criar uma política armazenada e uma SAS usando Python

1. Abra o arquivo SASToken.py e altere os valores a seguir:

    * policy\_name: o nome a ser usado para a política armazenada que será criada.
    
    * storage\_account\_name: o nome da conta de armazenamento.
    
    * storage\_account\_key: a chave da conta de armazenamento.
    
    * storage\_container\_name: o contêiner na conta de armazenamento para o qual você deseja restringir o acesso.
    
    * example\_file\_path: o caminho para um arquivo que será carregado no contêiner.

2. Execute o script. Isso exibirá o token SAS semelhante ao seguinte quando o script for concluído:

        sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14
    
    Salve o token de política da SAS pois você precisará dele ao associar a conta de armazenamento ao seu cluster HDInsight. Você também precisará do nome da conta de armazenamento e do nome do contêiner.
    
##Usar a SAS com o HDInsight

Ao criar um cluster HDInsight, você deverá especificar uma conta de armazenamento primária e, opcionalmente, poderá especificar contas de armazenamento adicionais. Ambos os métodos de adição de armazenamento exigem acesso total às contas de armazenamento e aos contêineres usados.

Para usar uma Assinatura de Acesso Compartilhado para limitar o acesso a um contêiner, você deverá adicionar uma entrada personalizada para a configuração __core-site__ do cluster.

* Para os clusters HDInsight __baseados no Windows__ ou __baseados em Linux__, você pode fazer isso durante a criação do cluster usando o PowerShell.

* Para clusters HDInsight __baseados em Linux__, altere a configuração após a criação do cluster usando o Ambari.

###Criar um novo cluster que use a SAS

Um exemplo de criação de um cluster HDInsight que use a SAS foi incluído no diretório `CreateCluster` do repositório. Para usá-lo, execute estas etapas:

1. Abra o arquivo `CreateCluster\HDInsightSAS.ps1` em um editor de texto e modifique os valores a seguir no início do documento.

        # Replace 'mycluster' with the name of the cluster to be created
        $clusterName = 'mycluster'
        # Valid values are 'Linux' and 'Windows'
        $osType = 'Linux'
        # Replace 'myresourcegroup' with the name of the group to be created
        $resourceGroupName = 'myresourcegroup'
        # Replace with the Azure data center you want to the cluster to live in
        $location = 'North Europe'
        # Replace with the name of the default storage account to be created
        $defaultStorageAccountName = 'mystorageaccount'
        # Replace with the name of the SAS container created earlier
        $SASContainerName = 'sascontainer'
        # Replace with the name of the SAS storage account created earlier
        $SASStorageAccountName = 'sasaccount'
        # Replace with the SAS token generated earlier
        $SASToken = 'sastoken'
        # Set the number of worker nodes in the cluster
        $clusterSizeInNodes = 2
        
    Por exemplo, altere `'mycluster'` para o nome do cluster que você deseja criar. Os valores da SAS devem corresponder aos valores das etapas anteriores durante a criação de uma conta de armazenamento e de um token SAS.
    
    Depois de alterar os valores, salve o arquivo.

1. Abra um novo prompt do Azure PowerShell. Se você não estiver familiarizado com o Azure PowerShell ou se não o tiver instalado, consulte [Instalar e configurar o Azure PowerShell][powershell].

2. No prompt, use o seguinte para se autenticar em sua assinatura do Azure:

        Login-AzureRmAccount
    
    Quando solicitado, faça logon com as informações da sua assinatura do Azure.
    
    Se o logon estiver associado a várias assinaturas do Azure, talvez seja necessário utilizar `Select-AzureRmSubscription` para selecionar a assinatura que você deseja usar.

2. No prompt, altere os diretórios para o diretório `CreateCluster` que contém o arquivo HDInsightSAS.ps1. Em seguida, use o seguinte para executar o script
        
        .\HDInsightSAS.ps1
    
    À medida que o script for executado, ele registrará a saída em log do prompt do PowerShell enquanto cria o grupo de recursos e as contas de armazenamento. Em seguida, ele solicitará que você insira o usuário HTTP para o cluster HDInsight. Essa é a conta de usuário usada para proteger o acesso HTTP/s ao cluster.
    
    Se você estiver criando um cluster baseado em Linux, também será solicitado um nome de conta de usuário SSH e uma senha. Isso é usado para fazer logon remoto no cluster.
    
    > [AZURE.IMPORTANT] Quando o nome de usuário ou a senha HTTP/s ou SSH forem solicitados, você deverá fornecer uma senha que atenda a estes critérios:
    >
    > - Deve ter pelo menos 10 caracteres de comprimento
    > - Deve conter pelo menos um dígito
    > - Deve conter pelo menos um caractere não alfanumérico
    > - Deve conter pelo menos uma letra maiúscula ou minúscula


Esse script demora um pouco para terminar, normalmente cerca de 15 minutos. Quando o script for concluído sem erros, o cluster terá sido criado.

###Atualizar um cluster existente para usar a SAS

Se você tiver um cluster baseado em Linux existente, poderá adicionar a SAS à configuração __core-site__ usando as seguintes etapas:

1. Abra a UI da Web do Ambari para seu cluster. O endereço dessa página é https://YOURCLUSTERNAME.azurehdinsight.net. Quando solicitado, faça a autenticação no cluster usando o nome do administrador (admin) e a senha usados na criação do cluster.

2. No lado esquerdo da interface do usuário da Web do Ambari, escolha __HDFS__ e selecione a guia __Configurações__ na metade da página.

3. Selecione a guia __Avançado__ e então role até encontrar a seção __Core-site personalizado__.

4. Expanda a seção __Core-site personalizado__, role até o fim e escolha o link __Adicionar propriedade...__. Use os valores a seguir para os campos __Chave__ e __Valor__:

    * __Chave__: fs.azure.sas.CONTAINERNAME.STORAGEACCOUNTNAME.blob.core.windows.net
    
    * __Valor__: a SAS retornada pelo aplicativo C# ou Python executado anteriormente
    
    Substitua __CONTAINERNAME__ pelo nome do contêiner usado com o aplicativo C# ou SAS. Substitua __STORAGEACCOUNTNAME__ pelo nome da conta de armazenamento usada.

5. Clique no botão __Adicionar__ para salvar essa chave e esse valor e clique no botão __Salvar__ para salvar as alterações de configuração. Quando solicitado, adicione uma descrição da alteração ("adicionando acesso de armazenamento SAS", por exemplo) e clique em __Salvar__.

    Clique em __OK__ quando as alterações forem concluídas.

    > [AZURE.IMPORTANT] Isso salva as alterações de configuração, mas você deverá reiniciar vários serviços antes que a alteração entre em vigor.

6. Na IU da Web do Ambari, escolha __HDFS__ na lista à esquerda e selecione __Reiniciar Todos__ na lista suspensa __Ações de Serviço__ à direita. Quando solicitado, selecione __Ativar o modo de manutenção__ e selecione \_\_Conforme Reiniciar Todos".

    Repita esse processo para as entradas MapReduce2 e YARN a partir da lista à esquerda da página.

7. Depois que elas tiverem sido reiniciadas, selecione cada uma e desabilite o modo de manutenção na lista suspensa __Ações de Serviço__.

##Testar o acesso restrito

Para verificar se você tem acesso restrito, use estes métodos:

* Para clusters HDInsight __baseados no Windows__, use a Área de Trabalho Remota para se conectar ao cluster. Veja [Conectar ao HDInsight usando RDP](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp) para saber mais.

    Uma vez conectado, use o ícone __Linha de Comando do Hadoop__ na área de trabalho para abrir um prompt de comando.

* Para clusters HDInsight __baseados em Linux__, use SSH para conectar-se ao cluster. Veja um dos artigos a seguir para obter informações sobre o uso de SSH com os clusters baseados em Linux:

    * [Usar SSH com Hadoop baseado em Linux no HDInsight do Linux, OS X e Unix](hdinsight-hadoop-linux-use-ssh-unix.md)
    * [Usar SSH com Hadoop baseado em Linux no HDInsight no Windows](hdinsight-hadoop-linux-use-ssh-windows.md)
    
Uma vez conectado ao cluster, use as etapas a seguir para verificar se você só pode ler e listar itens na conta de armazenamento SAS:

1. No prompt, digite o seguinte. Substitua __SASCONTAINER__ pelo nome do contêiner criado para a conta de armazenamento SAS. Substitua __SASACCOUNTNAME__ pelo nome da conta de armazenamento usada para a SAS:

        hdfs dfs -ls wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/
    
    Isso listará o conteúdo do contêiner, que deve incluir o arquivo carregado quando o contêiner e a SAS foram criados.
    
2. Use o seguinte para verificar se você pode ler o conteúdo do arquivo. Substitua __SASCONTAINER__ e __SASACCOUNTNAME__ como na etapa anterior. Substitua __FILENAME__ pelo nome do arquivo exibido no comando anterior:

        hdfs dfs -text wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/FILENAME
        
    Isso listará o conteúdo do arquivo.
    
3. Use o seguinte para baixar o arquivo para o sistema de arquivos local:

        hdfs dfs -get wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/FILENAME testfile.txt
    
    Isso baixará o arquivo para um arquivo local chamado __testfile.txt__.

4. Use o seguinte para carregar o arquivo local para um novo arquivo chamado __testupload.txt__ no armazenamento SAS:

        hdfs dfs -put testfile.txt wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/testupload.txt
    
    Você receberá uma mensagem semelhante à seguinte:
    
        put: java.io.IOException
        
    Esse erro só ocorrerá porque o local do armazenamento é somente leitura+lista. Use o seguinte para colocar os dados no armazenamento padrão do cluster, que pode ser gravável:
    
        hdfs dfs -put testfile.txt wasb:///testupload.txt
        
    Dessa vez, a operação deverá ser concluída com êxito.
    
##Solucionar problemas

###Uma tarefa foi cancelada

__Sintomas__: ao criar um cluster usando o script do PowerShell, talvez você receba a seguinte mensagem de erro:

    New-AzureRmHDInsightCluster : A task was canceled.
    At C:\Users\larryfr\Documents\GitHub\hdinsight-azure-storage-sas\CreateCluster\HDInsightSAS.ps1:62 char:5
    +     New-AzureRmHDInsightCluster `
    +     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : NotSpecified: (:) [New-AzureRmHDInsightCluster], CloudException
        + FullyQualifiedErrorId : Hyak.Common.CloudException,Microsoft.Azure.Commands.HDInsight.NewAzureHDInsightClusterCommand

__Causa__: esse erro pode ocorrer se você usar uma senha para o usuário admin/HTTP do cluster ou, para clusters baseados em Linux, o usuário SSH.

__Resolução__: use uma senha que atenda aos seguintes critérios:

- Deve ter pelo menos 10 caracteres de comprimento
- Deve conter pelo menos um dígito
- Deve conter pelo menos um caractere não alfanumérico
- Deve conter pelo menos uma letra maiúscula ou minúscula

##Próximas etapas

Agora que você aprendeu a adicionar armazenamento de acesso limitado ao seu cluster HDInsight, conheça outras maneiras de trabalhar com dados no cluster:

* [Usar o Hive com o HDInsight](hdinsight-use-hive.md)

* [Usar o Pig com o HDInsight](hdinsight-use-pig.md)

* [Usar o MapReduce com o HDInsight](hdinsight-use-mapreduce.md)

[powershell]: ../powershell-install-configure.md

<!---HONumber=AcomDC_0706_2016-->