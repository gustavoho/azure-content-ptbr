<properties
 pageTitle="Usar a interface do usuário de gerenciamento de dispositivos do Hub IoT | Microsoft Azure"
 description="Passo a passo do uso da interface do usuário de gerenciamento de dispositivos do Hub IoT do Azure"
 services="iot-hub"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="06/08/2016"
 ms.author="dobett"/>

# Explorar o gerenciamento de dispositivo do Hub IoT do Azure usando a interface do usuário de exemplo

A interação com a interface do usuário de gerenciamento de dispositivo de exemplo ajudará você a solidificar os conceitos e recursos abordados nos artigos de [Visão geral][lnk-dm-overview] e [Introdução][lnk-get-started] do gerenciamento de dispositivo do Hub IoT do Azure. Este artigo orienta você por cada um dos três conceitos principais de gerenciamento de dispositivo – *dispositivo gêmeo*, *consultas de dispositivo* e *trabalhos de dispositivo* – conforme representado na interface do usuário da Web de gerenciamento de dispositivo de exemplo.

Os desenvolvedores que desejam criar sua própria experiência interativa de gerenciamento de dispositivo podem usar o exemplo de código base de interface do usuário como base para um projeto personalizado. Você pode examinar os documentos completos de código e Leiame do projeto, que detalham recursos adicionais de desenvolvedor no repositório GitHub [Interface de usuário de gerenciamento de dispositivo do IoT do Azure][lnk-dm-github].

## Pré-requisitos

Antes de iniciar este tutorial, você deve concluir as etapas no artigo [Introdução ao gerenciamento de dispositivo do Hub IoT do Azure][lnk-get-started]. Se você não tiver feito isso, volte e conclua todas as etapas desse artigo antes de continuar.

Depois de concluir o tutorial "Introdução", o seguinte estará em execução em seu sistema de teste:

- Seis dispositivos **iotdm\_simple\_sample** simulados em execução nas janelas terminal/console, cada um com uma mensagem de êxito de "REGISTRADO".

- O exemplo de aplicativo de interface do usuário de gerenciamento de dispositivo compilado e executado localmente em <http://127.0.0.1:3003>.

## Modo de exibição padrão de dispositivos

A tela inicial padrão para o exemplo de interface do usuário de gerenciamento de dispositivo é o modo de exibição **Dispositivos**, que inclui os cinco componentes a seguir:

![][1]

1.  *Botões de navegação*: modo de exibição **Dispositivos** (selecionado), modo de exibição **Histórico de Trabalhos** e modo de exibição **Adicionar um Dispositivo**.

2. *Pesquisa de dispositivo*: localize e edite um único dispositivo pela ID do dispositivo.

3.  *Ações de dispositivo*: ação **Editar**, ação **Excluir** e ação **Exportar**.

4.  *Trabalhos de dispositivo*: **Reinicializar** dispositivo, **Atualização de Firmware** e **Redefinição de Fábrica**.

5.  *Filtro de grade do dispositivo*: editor de filtro para compilar e salvar exibições personalizadas de grade do dispositivo.

6.  *Grade do dispositivo*: exiba todos os dispositivos registrados com sua instância do Hub IoT com as propriedades padrão (**ID do Dispositivo**, **Status** e **Marcas**).

A [Visão geral do gerenciamento de dispositivo][lnk-dm-overview] apresentou o conceito de *dispositivo gêmeo*, que representa um dispositivo físico (ou simulado) no Hub IoT do Azure. Na grade do dispositivo, você pode selecionar qualquer dispositivo registrado na lista de dispositivos para exibir e editar o dispositivo gêmeo para esse dispositivo.

Entre nesse modo de exibição detalhado em seu primeiro dispositivo simulado, **Device11 7ce4a850**, selecionando a linha correspondente do dispositivo e, em seguida, clicando no botão **Editar** (você também pode clicar duas vezes na linha ou inserir a ID do dispositivo na caixa de pesquisa).

Agora você está vendo a representação completa dos componentes do dispositivo gêmeo, na qual você pode atualizar as propriedades graváveis e executar outras operações de dispositivo, conforme detalhado abaixo:

![][2]

1.  **Editar um Dispositivo Gêmeo**: isso inclui uma opção de **Habilitado/Desabilitado** para o dispositivo.

2.  **Propriedades do Serviço**: isso inclui **Marcas** do dispositivo.

3.  **Propriedades do Dispositivo**: clique para expandir esta seção.

4.  Botão **Atualizar**: recupere as propriedades e valores mais recentes do dispositivo gêmeo.

Para continuar, clique em **Cancelar** no canto inferior direito deste modo de exibição para retornar à página padrão da lista de dispositivos.

## Usar consultas de dispositivo para filtrar o modo de exibição do dispositivo

Consultas de dispositivo são uma maneira eficiente de localizar rapidamente um dispositivo ou um grupo de dispositivos com uma propriedade específica (como uma marca específica). O exemplo de interface do usuário usa consultas de dispositivo para preencher a listagem de dispositivos na página de lista de dispositivos padrão. O exemplo de interface do usuário permite a adição e remoção de propriedades do serviço na tabela e a filtragem de algumas dessas propriedades.

Execute as etapas a seguir para criar um filtro de cliente na marca de propriedade de serviço "bacon":

1.  Clique no ícone de funil para abrir o modo de exibição de edição de consulta do dispositivo:

    ![][3]

2.  Clique em **+ Adicionar um filtro** para expandir o editor. Selecione **Marcas** na lista suspensa de propriedades, digite **bacon** no campo de texto e clique em **Aplicar**. Agora, a lista de dispositivos exibe apenas os três dispositivos com a marca "bacon":

    ![][4]

3.  No campo de título da consulta (ao lado do ícone de funil), dê o nome **Somente Bacon** à consulta e clique em **Salvar**:

    ![][5]

4.  Clique no ícone de funil para sair do editor de consulta do dispositivo.

## Usar um trabalho de dispositivo para simular reinicializações do dispositivo 

Como você aprendeu na visão geral do gerenciamento de dispositivo, os trabalhos de dispositivo permitem que você coordene ações simples ou complexas em um ou mais dispositivos físicos. Nesta seção, você criará um trabalho de dispositivo no exemplo de interface do usuário para executar uma operação de reinicialização em todos os dispositivos de simulação com a marca "bacon":

1.  Da lista de consultas de dispositivo **Somente Bacon**, clique em cada linha de dispositivo para marcá-la para a operação do trabalho de reinicialização:

    ![][6]

2.  Clique no botão **Reinicializar** na barra de tarefas de trabalho do dispositivo para criar o trabalho de reinicialização. Depois de confirmar com **Sim**, clique no hiperlink **Histórico de Trabalhos** na caixa de diálogo **O Trabalho para o Dispositivo foi iniciado** resultante para navegar até o modo de exibição Trabalhos do Dispositivo.

    ![][7]

Agora você criou um único trabalho pai que envolve três trabalhos filhos, cada um deles executa a operação de reinicialização em um dos três dispositivos "bacon" marcados:

![][8]

A atualização dessa tela após alguns instantes altera o status do trabalho pai e dos três trabalhos filhos para **concluído**, indicando que as operações de reinicialização foram bem-sucedidas e confirmadas pelos dispositivos simulados. Use a coluna **ID de Dispositivo** para determinar quais trabalhos estão associados a quais dispositivos.


> [AZURE.NOTE] Se os trabalhos filho retornarem o status "Falha", verifique se os dispositivos simulados ainda estão em execução no sistema de teste. Caso contrário, execute o script simulate.bat ou simulate.sh novamente e repita as etapas do trabalho de reinicialização de dispositivo acima.

## Próximas etapas

Você concluiu uma exploração guiada dos conceitos de gerenciamento de dispositivo, por exemplo, usando o exemplo de experiência de interface do usuário de gerenciamento de dispositivo. Se você quiser obter uma compreensão mais avançada das APIs de gerenciamento de dispositivos e experimentar alguns exemplos de código, visite os seguintes tutoriais para desenvolvedores:

- [Como usar o dispositivo gêmeo][lnk-tutorial-twin]
- [Como encontrar dispositivos gêmeos usando consultas][lnk-tutorial-queries]
- [Como usar trabalhos do dispositivo para atualizar o firmware do dispositivo][lnk-tutorial-jobs]
- [Introdução à biblioteca de clientes gerenciamento de dispositivos do Hub IoT do Azure][lnk-library-c]

[1]: media/iot-hub-device-management-ui-sample/image1.png
[2]: media/iot-hub-device-management-ui-sample/image2.png
[3]: media/iot-hub-device-management-ui-sample/image3.png
[4]: media/iot-hub-device-management-ui-sample/image4.png
[5]: media/iot-hub-device-management-ui-sample/image5.png
[6]: media/iot-hub-device-management-ui-sample/image6.png
[7]: media/iot-hub-device-management-ui-sample/image7.png
[8]: media/iot-hub-device-management-ui-sample/image8.png

[lnk-dm-overview]: iot-hub-device-management-overview.md
[lnk-get-started]: iot-hub-device-management-get-started.md
[lnk-dm-github]: https://github.com/Azure/azure-iot-device-management/
[lnk-library-c]: iot-hub-device-management-library.md
[lnk-tutorial-twin]: iot-hub-device-management-device-twin.md
[lnk-tutorial-queries]: iot-hub-device-management-device-query.md
[lnk-tutorial-jobs]: iot-hub-device-management-device-jobs.md

<!---HONumber=AcomDC_0615_2016-->