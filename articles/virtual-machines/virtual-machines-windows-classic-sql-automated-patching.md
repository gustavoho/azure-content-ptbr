<properties
	pageTitle="Aplicação de patch automatizada para VMs do SQL Server (Clássico) | Microsoft Azure"
	description="Explica o recurso de Aplicação de Patch Automatizada para Máquinas Virtuais do SQL Server em execução no Azure e usando o modo de implantação clássico."
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management" />
<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="05/18/2016"
	ms.author="jroth" />

# Aplicação de patch automatizada para o SQL Server em Máquinas Virtuais do Azure (Clássico)

> [AZURE.SELECTOR]
- [Gerenciador de Recursos](virtual-machines-windows-sql-automated-patching.md)
- [Clássico](virtual-machines-windows-classic-sql-automated-patching.md)

A aplicação de patch automatizada estabelece uma janela de manutenção para uma Máquina Virtual do Azure que executa o SQL Server. Atualizações automáticas só podem ser instaladas durante esta janela de manutenção. Para o SQL Server, isso garante que as atualizações do sistema e qualquer reinicialização associada ocorrerão no melhor momento possível para o banco de dados. A aplicação de patch automatizada depende da [Extensão do Agente IaaS do SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] 
Para exibir a versão do Resource Manager deste artigo, consulte [Aplicação de Patch Automatizada para o SQL Server em Máquinas Virtuais do Azure do Resource Manager](virtual-machines-windows-sql-automated-patching.md).

## Pré-requisitos

Para usar a Aplicação de Patch Automatizada, considere os seguintes pré-requisitos:

**Sistema operacional**:

- Windows Server 2012
- Windows Server 2012 R2

**Versão do SQL Server**:

- SQL Server 2012
- SQL Server 2014
- SQL Server 2016

**Azure PowerShell**:

- [Instalar os comandos mais recentes do Azure PowerShell](../powershell-install-configure.md) se você planeja configurar a Aplicação de Patch Automatizada com o PowerShell.

>[AZURE.NOTE] A aplicação de Patch automatizada depende da Extensão do Agente IaaS do SQL Server. As imagens atuais da galeria da máquina virtual do SQL adicionam essa extensão por padrão. Para obter mais informações, consulte [Extensão do agente IaaS do SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

## Configurações

A tabela a seguir descreve as opções que podem ser configuradas para Aplicação de Patch Automatizada. As etapas de configuração reais variam dependendo de se você usar os comandos do Portal do Azure ou do Azure Windows PowerShell.

|Configuração|Valores possíveis|Descrição|
|---|---|---|
|**Aplicação de patch automatizada**|Habilitar/desabilitar (Desabilitado)|Habilita ou desabilita a Aplicação de Patch Automatizada para uma máquina virtual do Azure.|
|**Agenda de manutenção**|Todos os dias, segunda-feira, terça-feira, quarta-feira, quinta-feira, sexta-feira, sábado e domingo|A agenda para baixar e instalar atualizações do Windows, do SQL Server e do Microsoft para sua máquina virtual.|
|**Hora de início da manutenção**|0h a 24h|A hora de início local para atualizar a máquina virtual.|
|**Duração da janela de manutenção**|30-180|O número de minutos permitidos para concluir o download e a instalação de atualizações.|
|**Categoria de patch**|Importante|A categoria de atualizações para baixar e instalar.|

## Configuração no Portal

Use o [Portal do Azure](http://go.microsoft.com/fwlink/?LinkID=525040&clcid=0x409) para configurar a Aplicação de Patch Automatizada quando criar uma nova Máquina Virtual do SQL Server no modelo de implantação clássico.

A captura de tela do portal do Azure a seguir mostra essas opções em **CONFIGURAÇÃO OPCIONAL** | **PATCHING AUTOMATIZADO DO SQL**.

![Aplicação de Patch Automática do SQL no Portal do Azure](./media/virtual-machines-windows-classic-sql-automated-patching/IC778484.jpg)

Para as máquinas virtuais existentes do SQL Server 2012 ou 2014, selecione as configurações de **Aplicação de patch automática** na seção **Configuração** das propriedades da máquina virtual. Na janela de **Aplicação de patch automatizada**, você pode habilitar o recurso, definir o agendamento de manutenção e a hora de início e escolher a duração da janela de manutenção. Isso é mostrado na captura de tela a seguir.

![Configuração de Aplicação de Patch Automatizada no Portal do Azure](./media/virtual-machines-windows-classic-sql-automated-patching/IC792132.jpg)

>[AZURE.NOTE] Quando você habilita a Aplicação de Patch Automatizada pela primeira vez, o Azure configura o Agente IaaS do SQL Server em segundo plano. Durante esse tempo, o portal do Azure não mostrará que a Aplicação de Patch Automatizada está configurada. Aguarde alguns minutos para que o agente seja instalado e configurado. Depois disso, o portal do Azure refletirá as novas configurações.

## Configuração com o PowerShell

Você também pode usar o PowerShell para configurar a aplicação de patch automatizada.

No exemplo a seguir, o PowerShell é usado para configurar a Aplicação de Patch Automatizada em uma VM existente do SQL Server. O comando **New-AzureVMSqlServerAutoPatchingConfig** configura uma nova janela de manutenção para atualizações automáticas.

    $aps = New-AzureVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoPatchingSettings $aps | Update-AzureVM

Com base neste exemplo, a tabela a seguir descreve o efeito prático sobre a VM do Azure de destino:

|Parâmetro|Efeito|
|---|---|
|**DayOfWeek**|Patches instalados toda quinta-feira.|
|**MaintenanceWindowStartingHour**|Inicia as atualizações às 11h.|
|**MaintenanceWindowsDuration**|Os patches devem ser instalados dentro de 120 minutos. Com base na hora de início, eles devem estar concluídos até 13h.|
|**PatchCategory**|A única configuração possível para esse parâmetro é "Important".|

Pode demorar vários minutos para instalar e configurar o Agente IaaS do SQL Server.

Para desabilitar a Aplicação de Patch Automatizada, execute o mesmo script sem o parâmetro -Enable para New-AzureVMSqlServerAutoPatchingConfig. Assim como acontece com a instalação, pode demorar vários minutos para desabilitar a Aplicação de Patch Automatizada.

## Próximas etapas

Para obter informações sobre outras tarefas de automação disponíveis, consulte [Extensão do agente IaaS do SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

Para obter mais informações sobre como executar o SQL Server em VMs do Azure, consulte [Visão geral do SQL Server em Máquinas Virtuais do Azure](virtual-machines-windows-sql-server-iaas-overview.md).

<!---HONumber=AcomDC_0629_2016-->