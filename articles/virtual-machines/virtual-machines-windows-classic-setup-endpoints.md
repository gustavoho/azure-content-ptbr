<properties
	pageTitle="Configurar pontos de extremidade em uma VM do Windows clássica | Microsoft Azure"
	description="Saiba como configurar pontos de extremidade no portal clássico do Azure para permitir a comunicação com uma máquina virtual do Windows no Azure."
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/19/2016"
	ms.author="cynthn"/>

# Como configurar pontos de extremidade em uma máquina virtual clássica no Azure


Todas as máquinas virtuais do Windows criadas no Azure usando o modelo de implantação clássico podem se comunicar automaticamente com outras máquinas virtuais no mesmo serviço de nuvem ou rede virtual por um canal de rede privada. No entanto, os computadores na Internet ou outras redes virtuais requerem pontos de extremidade para direcionar o tráfego de rede de entrada para uma máquina virtual. Este artigo também está disponível para [máquinas virtuais Linux](virtual-machines-linux-classic-setup-endpoints.md).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] Saiba como [executar estas etapas usando o modelo do Resource Manager](virtual-machines-windows-nsg-quickstart-portal.md). No modelo de implantação **Resource Manager**, os pontos de extremidade são configurados usando **NSGs (Grupos de Segurança de Rede)**. Para saber mais, veja [Permitir o acesso externo à sua VM usando o Portal do Azure](virtual-machines-windows-nsg-quickstart-portal.md).

Quando você cria uma máquina virtual do Windows no portal clássico do Azure, pontos de extremidade comuns como aqueles para a Área de Trabalho Remota e a Comunicação Remota do Windows PowerShell normalmente são criados para você automaticamente. Você pode configurar pontos de extremidade adicionais criando a máquina virtual ou posteriormente, conforme a necessidade.



[AZURE.INCLUDE [virtual-machines-common-classic-setup-endpoints](../../includes/virtual-machines-common-classic-setup-endpoints.md)]

## Próximas etapas

* Para usar um cmdlet do Azure PowerShell para configurar um ponto de extremidade de VM, veja [Add-AzureEndpoint](https://msdn.microsoft.com/library/azure/dn495300.aspx).

* Para usar um cmdlet do Azure PowerShell para gerenciar uma ACL em um ponto de extremidade, veja [Como gerenciar ACLs (listas de controle de acesso) para pontos de extremidade usando o PowerShell](../virtual-network/virtual-networks-acl-powershell.md).

* Se você criou uma máquina virtual no modelo de implantação do Resource Manager, é possível usar o Azure PowerShell para [criar grupos de segurança de rede](../virtual-network/virtual-networks-create-nsg-arm-ps.md) para controlar o tráfego para a VM.

<!---HONumber=AcomDC_0629_2016-->