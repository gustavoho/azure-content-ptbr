<properties
	pageTitle="Introdução ao Gerenciador de Armazenamento (Preview) | Microsoft Azure"
	description="Gerenciar os recursos de armazenamento do Azure com o Gerenciador de Armazenamento (Visualização)"
	services="visual-studio-online"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor="" />

 <tags
	ms.service="visual-studio-online"
	ms.devlang="multiple"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="na"
	ms.workload="na"
	ms.date="07/06/2016"
	ms.author="tarcher" />

# Introdução ao Gerenciador de Armazenamento (Preview)

## Visão geral 

O Gerenciador de Armazenamento do Microsoft Azure (Preview) é um aplicativo autônomo que permite que você trabalhe facilmente com dados do Armazenamento do Azure no Windows, OSX e Linux. Neste artigo, você aprenderá as várias maneiras de se conectar e gerenciar suas contas de armazenamento do Azure.

## Pré-requisitos

- [Baixe e instale o Gerenciador de Armazenamento (preview)](http://www.storageexplorer.com)

## Conectar-se a uma conta de armazenamento ou serviço

O Gerenciador de Armazenamento (Preview) fornece várias maneiras de se conectar às contas de armazenamento. Isso inclui se conectar a contas de armazenamento associadas às suas assinaturas do Azure, conectar-se às contas de contas de armazenamento e serviços compartilhados por outras assinaturas do Azure e até mesmo se conectar e gerenciar o armazenamento local usando o Emulador de Armazenamento do Azure:

- [Conectar-se a uma assinatura do Azure](#connect-to-an-azure-subscription) - Gerencie os recursos de armazenamento que pertencem à sua assinatura do Azure.
- [Trabalhar com o armazenamento de desenvolvimento local](#work-with-local-development-storage) - gerencie o armazenamento local usando o Emulador de Armazenamento do Azure.
- [Anexar ao armazenamento externo](#attach-or-detach-an-external-storage-account) - Gerencie os recursos de armazenamento que pertencem a outra assinatura do Azure usando o nome de conta e a chave da conta de armazenamento.
- [Anexar conta de armazenamento usando SAS](#attach-storage-account-using-sas) - gerencie os recursos que pertencem a outra assinatura do Azure usando um SAS.
- [Anexar usando SAS](#attach-service-using-sas) - Gerencie um serviço de armazenamento específico (contêiner de blob, fila ou tabela) que pertencem a outra assinatura do Azure usando um SAS.

## Conectar-se a uma assinatura do Azure

> [AZURE.NOTE] Se não tiver uma conta do Azure, você poderá [inscrever-se para uma avaliação gratuita](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F) ou [ativar seus benefícios de assinante do Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).

1. No Gerenciador de Armazenamento (Visualização), selecione **Configurações de conta do Azure**.

	![][0]

1. O painel esquerdo agora exibirá todas as contas da Microsoft em que você fez logon. Para se conectar a outra conta, selecione **Adicionar uma conta** e siga os diálogos para entrar com uma conta da Microsoft associada a pelo menos uma assinatura ativa do Azure.

	![][1]

1. Depois de entrar com êxito com uma conta da Microsoft, o painel esquerdo preencherá com as assinaturas do Azure associadas à conta. Selecione as assinaturas do Azure com as quais você deseja trabalhar e selecione **Aplicar**. (A escolha dos botões **Todas as assinaturas** seleciona todas ou nenhuma das assinaturas do Azure listadas.)

	![][3]

1. O painel esquerdo agora exibirá as contas de armazenamento associadas às assinaturas do Azure selecionadas.

	![][4]

## Trabalhar com armazenamento de desenvolvimento local

O Gerenciador de Armazenamento (Preview) permite que você trabalhe no armazenamento local usando o Emulador de Armazenamento do Azure. Isso permite que você escreva códigos e teste o armazenamento sem necessariamente ter uma conta de armazenamento implantada no Azure (uma vez que a conta de armazenamento está sendo emulada pelo Emulador de Armazenamento do Azure).

>[AZURE.NOTE] No momento, o Emulador de Armazenamento do Azure tem suporte somente para Windows.

1. No painel esquerdo do Gerenciador de Armazenamento (Visualização), expanda o nó **(Local e Anexados** > **Contas de Armazenamento** > **(Desenvolvimento)**.

	![][21]

1. Se você ainda não tiver instalado o Emulador de Armazenamento do Azure, receberá uma solicitação para fazer isso por meio de uma barra de informações. Se a barra de informações for exibida, escolha **Baixar a versão mais recente** e instale o emulador.

	![][22]

1. Após a instalação do emulador, você poderá criar e trabalhar com blobs, filas e tabelas locais. Para aprender a trabalhar com cada tipo de conta de armazenamento, escolha o link apropriado abaixo:

	- [Gerenciar recursos de Armazenamento de Blobs do Azure](./vs-azure-tools-storage-explorer-blobs.md)
	- Gerenciar recursos de armazenamento de compartilhamento de arquivos do Azure - *em breve*
	- Gerenciar recursos de Armazenamento de filas do Azure - *em breve*
	- Gerenciar recursos de Armazenamento de tabelas do Azure - *em breve*

## Conexão ou desconexão de uma conta de armazenamento externo

O Gerenciador de Armazenamento (Preview) permite anexar contas de armazenamento externo para facilitar o compartilhamento das contas de armazenamento. Esta seção explica como anexar e desanexar contas do armazenamento externo.

### Obter as credenciais da conta de armazenamento

Para compartilhar uma conta de armazenamento externo, o proprietário dessa conta precisa primeiro obter as credenciais da conta, nome e chave, para depois compartilhar essas informações com a pessoa que deseja ser anexada a essa conta (externa). As credenciais da conta de armazenamento podem ser obtidas no Portal do Azure, executando estas etapas:

1.	Entre no [Portal do Azure](https://portal.azure.com).
1.	Selecione **Procurar**.
1.	Selecione **Contas de Armazenamento**.
1.	Na folha **Contas de Armazenamento**, escolha a conta de armazenamento desejada.
1.	Na folha **Configurações** da conta de armazenamento selecionada, escolha **Chaves de acesso**.

	![][5]
	
1.	Na folha **Chaves de acesso**, copie os valores de **NOME DA CONTA DE ARMAZENAMENTO** e **CHAVE 1** que serão usados para anexação à conta de armazenamento.

	![][6]

### Anexação a uma conta de armazenamento externo
Para anexar a uma conta de armazenamento externo, você precisará do nome da conta e da chave. A seção *Obter as credenciais de conta de armazenamento* explica como obter esses valores no portal do Azure. No entanto, observe que, no portal, a chave da conta se chama "chave 1" e, portanto, quando o Gerenciador de Armazenamento (Visualização) solicitar uma chave de conta, você inserirá (ou colará) o valor "chave 1".
 
1.	No Gerenciador de Armazenamento (Visualização), selecione **Conectar ao armazenamento do Azure**.

	![][23]

1.	Na caixa de diálogo **Conectar ao armazenamento do Azure**, especifique a chave de conta (o valor "chave 1" no portal do Azure) e selecione **Avançar**.

	![][24]

1.	Na caixa de diálogo **Anexar Armazenamento Externo**, digite o nome da conta de armazenamento na caixa** Nome da conta**, especifique outras configurações desejadas e selecione **Avançar** para concluir.

	![][8]

1.	Na caixa de diálogo **Resumo da Conexão**, verifique as informações. Se você quiser alterar alguma coisa, selecione **Voltar** e insira novamente as configurações desejadas. Quando tiver concluído, selecione **Conectar**.

1.	Depois de conectada, a conta de armazenamento externo será exibida com o texto **(Externo)** anexado ao nome da conta de armazenamento.

	![][9]

### Desconexão de uma conta de armazenamento externo

1. 	Clique com o botão direito na conta de armazenamento externo que você deseja desconectar e, no menu de contexto, escolha **Desanexar**.

	![][10]

1.	Quando a caixa de mensagem de confirmação aparecer, escolha **Sim** para confirmar a desconexão da conta de armazenamento externo.

## Anexar conta de armazenamento usando SAS

Uma [SAS (Assinatura de Acesso Compartilhado)](storage/storage-dotnet-shared-access-signature-part-1.md) proporciona ao administrador de uma assinatura do Azure a capacidade de conceder acesso temporário a uma conta de armazenamento sem precisar fornecer as credenciais de assinatura do Azure.

Para ilustrar isso, vamos supor que o UsuárioA é um administrador de uma assinatura do Azure, e o UsuárioA deseja permitir que o UsuárioB acesse uma conta de armazenamento por um período limitado com determinadas permissões:

1. O UsuárioA gera uma SAS (formada pela cadeia de conexão da conta de armazenamento) para um período específico e com as permissões desejadas.
1. O UsuárioA compartilha a SAS com a pessoa que deseja acessar a conta de armazenamento - em nosso exemplo, o UsuárioB.
1. O UsuárioB usa o Gerenciador de Armazenamento (Preview) para se anexar à conta pertencente ao UsuárioA usando a SAS fornecida.

### Obter uma SAS para a conta que você deseja compartilhar

1.	No Gerenciador de Armazenamento (Visualização), clique com botão direito do mouse na conta de armazenamento que você deseja compartilhar e, no menu de contexto, escolha **Obter Assinatura de Acesso Compartilhado**.

	![][13]

1. Na caixa de diálogo **Assinatura de Acesso Compartilhado**, especifique o intervalo e as permissões que você deseja para a conta e escolha **Criar**.

	![][14]
 
1. Uma segunda caixa de diálogo **Assinatura de Acesso Compartilhado** aparecerá exibindo a SAS. Escolha **Copiar** ao lado de **Cadeia de Conexão** para copiá-la na área de transferência. Escolha **Fechar** para ignorar a caixa de diálogo.

### Anexar à conta compartilhada usando a SAS

1.	No Gerenciador de Armazenamento (Visualização), selecione **Conectar ao armazenamento do Azure**.

	![][23]

1.	Na caixa de diálogo **Conectar ao armazenamento do Azure**, especifique a cadeia de conexão e selecione **Avançar**.

	![][24]

1.	Na caixa de diálogo **Resumo da Conexão**, verifique as informações. Se você quiser alterar alguma coisa, selecione **Voltar** e insira novamente as configurações desejadas. Quando tiver concluído, selecione **Conectar**.

1.	Depois de anexada, a conta de armazenamento será exibida com o texto (SAS) anexado ao nome da conta fornecido.

	![][17]

## Anexar o serviço usando SAS

A seção [Anexar a conta usando SAS](#attach-account-using-sas) ilustra como um administrador de assinatura do Azure pode conceder acesso temporário a uma conta de armazenamento gerando (e compartilhando) uma SAS para a conta de armazenamento. Da mesma forma, uma SAS pode ser gerada para um serviço específico (contêiner de blob, fila ou tabela) em uma conta de armazenamento.

### Gerar uma SAS para o serviço que você deseja compartilhar

Nesse contexto, um serviço pode ser um contêiner de blob, fila ou tabela. As seções a seguir explicam como gerar a SAS para o serviço listado:

- [Obter a SAS para um contêiner de blob](./vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container)
- Obter a SAS para um compartilhamento de arquivos - *em breve*
- Obter a SAS para uma fila - *em breve*
- Obter a SAS para uma tabela - *em breve*

### Conectar ao serviço de conta compartilhada usando a SAS

1.	No Gerenciador de Armazenamento (Visualização), selecione **Conectar ao armazenamento do Azure**.

	![][23]

1.	Na caixa de diálogo **Conectar ao armazenamento do Azure**, especifique o URI de SAS e selecione **Avançar**.

	![][24]

1.	Na caixa de diálogo **Resumo da Conexão**, verifique as informações. Se você quiser alterar alguma coisa, selecione **Voltar** e insira novamente as configurações desejadas. Quando tiver concluído, selecione **Conectar**.

1.	Depois de anexado o serviço recém-anexado será exibido abaixo do nó **(SAS do Serviço)**.

	![][20]

## Pesquisar nas contas de armazenamento

Se você tiver uma lista longa de contas de armazenamento, uma maneira rápida de localizar uma determinada conta de armazenamento é usar a caixa de pesquisa na parte superior do painel esquerdo.

À medida que você digita na caixa de pesquisa, o painel esquerdo exibirá somente as contas de armazenamento que correspondem ao valor de pesquisa inserido até esse ponto. A captura de tela a seguir ilustra um exemplo no qual eu pesquisei por todas as contas de armazenamento cujo nome contém o texto "tarcher".

![][11]
	
Para limpar a pesquisa, escolha o botão **x** na caixa de pesquisa.

## Próximas etapas
- [Gerenciar os recursos de armazenamento de blobs do Azure com o Gerenciador de Armazenamento (Preview)](./vs-azure-tools-storage-explorer-blobs.md)

[0]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/settings-icon.png
[1]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/add-account-link.png
[3]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/subscriptions-list.png
[4]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-accounts-list.png
[5]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/access-keys.png
[6]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/access-keys-copy.png
[8]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-external-storage-dlg.png
[9]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/external-storage-account.png
[10]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/detach-external-storage.png
[11]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-account-search.png
[12]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/detach-external-storage-confirmation.png
[13]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-sas-context-menu.png
[14]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-sas-dlg1.png
[17]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-account-using-sas-finished.png
[20]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-service-using-sas-finished.png
[21]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/local-storage-drop-down.png
[22]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/download-storage-emulator.png
[23]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-icon.png
[24]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-next.png

<!---HONumber=AcomDC_0713_2016-->