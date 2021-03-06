<properties
   pageTitle="Como configurar o roteamento de um circuito da Rota Expressa para o modelo de implantação clássica usando o PowerShell | Microsoft Azure"
   description="Este artigo fornece uma orientação sobre as etapas de criação e de provisionamento do emparelhamento público, privado e da Microsoft de um circuito de Rota Expressa. Este artigo também mostra como verificar o status, atualizar ou excluir emparelhamentos de seu circuito."
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="06/27/2016"
   ms.author="ganesr"/>

# Criar e modificar o roteamento de um circuito da Rota Expressa

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](expressroute-howto-routing-portal-resource-manager.md)
[PowerShell - Resource Manager](expressroute-howto-routing-arm.md)
[PowerShell - Classic](expressroute-howto-routing-classic.md)



Este artigo fornece uma orientação pelas etapas de criação e de gerenciamento da configuração de roteamento de um circuito da Rota Expressa usando o PowerShell e o modelo de implantação clássico. As etapas a seguir também mostrarão a você como verificar o status, atualizar ou excluir e desprovisionar emparelhamentos de um circuito da Rota Expressa.


**Sobre modelos de implantação do Azure**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## Pré-requisitos de configuração

- Você precisará da versão mais recente dos módulos do Azure PowerShell. Baixe o módulo mais recente do PowerShell na seção PowerShell da [página Downloads do Azure](https://azure.microsoft.com/downloads/). Siga as instruções na página [Como instalar e configurar o Azure PowerShell](../powershell-install-configure.md) para obter orientações passo a passo sobre como configurar seu computador para usar os módulos do Azure PowerShell.
- Certifique-se de que você leu as páginas de [pré-requisitos](expressroute-prerequisites.md), [requisitos de roteamento](expressroute-routing.md) e [fluxos de trabalho](expressroute-workflows.md) antes de começar a configuração.
- Você deve ter um circuito da Rota Expressa ativo. Antes de continuar, siga as instruções para [criar um circuito da Rota Expressa](expressroute-howto-circuit-classic.md) e para que o circuito seja habilitado pelo provedor de conectividade. O circuito da Rota Expressa deve estar em um estado provisionado e habilitado e para que você possa executar os cmdlets descritos abaixo.

>[AZURE.IMPORTANT] Estas instruções se aplicam apenas a circuitos criados com provedores de serviço que oferecem serviços de conectividade de Camada 2. Se você estiver usando um provedor de serviços que oferece serviços gerenciados de Camada 3 (normalmente um IPVPN, como MPLS), seu provedor de conectividade configurará e gerenciará o roteamento para você.

Você pode configurar um, dois ou todos os três emparelhamentos (privado e público do Azure e da Microsoft) para um circuito da Rota Expressa. Você pode configurar emparelhamentos em qualquer ordem escolhida. No entanto, você deve concluir a configuração de um emparelhamento por vez.

## Emparelhamento privado do Azure

Esta seção fornece instruções sobre como criar, obter, atualizar e excluir a configuração de emparelhamento privado do Azure para um circuito da Rota Expressa.

### Criar um emparelhamento privado do Azure

1. **Importe o módulo do PowerShell para Rota Expressa.**
	
	Para começar a usar os cmdlets da Rota Expressa, é necessário importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell. Execute os seguintes comandos para importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell.

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **Criar um circuito da Rota Expressa.**
	
	Siga as instruções para criar um [circuito da Rota Expressa](expressroute-howto-circuit-classic.md) e para que o circuito seja provisionado pelo provedor de conectividade. Caso seu provedor de conectividade ofereça serviços gerenciados de camada 3, solicite a ele a habilitação do emparelhamento privado do Azure. Nesse caso, não será necessário seguir as instruções listadas nas seções a seguir. No entanto, se o seu provedor de conectividade não gerenciar o roteamento, após a criação do circuito, siga as instruções abaixo.

3. **Verifique se o circuito da Rota Expressa foi provisionado.**

	Primeiro, verifique se o circuito da Rota Expressa está Provisionado e Habilitado. Veja o exemplo abaixo.

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : MyTestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	Verifique se o circuito aparece como Provisionado e Habilitado. Caso contrário, converse com seu provedor de conectividade para que o circuito fique com o status e o estado exigidos.

		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled


4. **Configure o emparelhamento privado do Azure para o circuito.**

	Verifique se você tem os seguintes itens antes de continuar com as próximas etapas:

	- Uma sub-rede /30 para o link principal. Ela não deve fazer parte de qualquer espaço de endereçamento reservado para redes virtuais.
	- Uma sub-rede /30 para o link secundário. Ela não deve fazer parte de qualquer espaço de endereçamento reservado para redes virtuais.
	- Uma ID válida de VLAN para estabelecer esse emparelhamento. Verifique se nenhum outro emparelhamento no circuito usa a mesma ID de VLAN.
	- Número de AS para emparelhamento. Você pode usar um número de AS de 2 e de 4 bytes. Você pode usar um número de AS privado para esse emparelhamento. Não use 65515.
	- Um Hash MD5, se você optar por usar um. **Isso é opcional**.
	
	Você pode executar o seguinte cmdlet para configurar o emparelhamento privado do Azure para seu circuito.

		New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100

	Use o cmdlet abaixo se você optar por usar um hash MD5.

		New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"

	>[AZURE.IMPORTANT] Especifique o número de AS como um ASN de emparelhamento, não um ASN de cliente.

### Para exibir detalhes sobre o emparelhamento privado do Azure

Você pode obter detalhes de configuração usando o seguinte cmdlet

	Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
	
	AdvertisedPublicPrefixes       : 
	AdvertisedPublicPrefixesState  : Configured
	AzureAsn                       : 12076
	CustomerAutonomousSystemNumber : 
	PeerAsn                        : 1234
	PrimaryAzurePort               : 
	PrimaryPeerSubnet              : 10.0.0.0/30
	RoutingRegistryName            : 
	SecondaryAzurePort             : 
	SecondaryPeerSubnet            : 10.0.0.4/30
	State                          : Enabled
	VlanId                         : 100


### Atualizar a configuração de emparelhamento privado do Azure

Você pode atualizar qualquer parte da configuração usando o cmdlet a seguir. No exemplo abaixo, a ID de VLAN do circuito está sendo atualizada de 100 para 500.

	Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"

### Excluir um emparelhamento privado do Azure

Você pode remover a configuração de emparelhamento executando o seguinte cmdlet.

>[AZURE.WARNING] Verifique se todas as redes virtuais estão desvinculadas do circuito da Rota Expressa antes de executar esse cmdlet.

	Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"


## Emparelhamento público do Azure

Esta seção fornece instruções sobre como criar, obter, atualizar e excluir a configuração de emparelhamento público do Azure para um circuito da Rota Expressa.

### Criar o emparelhamento público do Azure

1. **Importe o módulo do PowerShell para Rota Expressa.**
	
	Para começar a usar os cmdlets da Rota Expressa, é necessário importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell. Execute os seguintes comandos para importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell.

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **Criar um circuito da Rota Expressa**
	
	Siga as instruções para criar um [circuito da Rota Expressa](expressroute-howto-circuit-classic.md) e para que o circuito seja provisionado pelo provedor de conectividade. Caso seu provedor de conectividade ofereça serviços gerenciados de camada 3, solicite a ele a habilitação do emparelhamento público do Azure. Nesse caso, não será necessário seguir as instruções listadas nas seções a seguir. No entanto, se o seu provedor de conectividade não gerenciar o roteamento, após a criação do circuito, siga as instruções abaixo.

3. **Verifique se o circuito da Rota Expressa foi provisionado**

	Primeiro, verifique se o circuito da Rota Expressa está Provisionado e Habilitado. Veja o exemplo abaixo.

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : MyTestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	Verifique se o circuito aparece como Provisionado e Habilitado. Caso contrário, converse com seu provedor de conectividade para que o circuito fique com o status e o estado exigidos.

		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled

	

4. **Configure o emparelhamento público do Azure para o circuito**

	Tenha as informações a seguir antes de prosseguir:

	- Uma sub-rede /30 para o link principal. Precisa ser um prefixo IPv4 público válido.
	- Uma sub-rede /30 para o link secundário. Precisa ser um prefixo IPv4 público válido.
	- Uma ID válida de VLAN para estabelecer esse emparelhamento. Verifique se nenhum outro emparelhamento no circuito usa a mesma ID de VLAN.
	- Número de AS para emparelhamento. Você pode usar um número de AS de 2 e de 4 bytes.
	- Um Hash MD5, se você optar por usar um. **Isso é opcional**.
	
	Você pode executar o cmdlet a seguir a fim de configurar o emparelhamento público do Azure para seu circuito.

		New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200

	Use o cmdlet abaixo se você optar por usar um hash MD5

		New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"

	>[AZURE.IMPORTANT] Especifique o número de AS como um ASN de emparelhamento e não um ASN de cliente.

### Para exibir detalhes sobre o emparelhamento público do Azure

Você pode obter detalhes de configuração usando o seguinte cmdlet

	Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
	
	AdvertisedPublicPrefixes       : 
	AdvertisedPublicPrefixesState  : Configured
	AzureAsn                       : 12076
	CustomerAutonomousSystemNumber : 
	PeerAsn                        : 1234
	PrimaryAzurePort               : 
	PrimaryPeerSubnet              : 131.107.0.0/30
	RoutingRegistryName            : 
	SecondaryAzurePort             : 
	SecondaryPeerSubnet            : 131.107.0.4/30
	State                          : Enabled
	VlanId                         : 200


### Atualizar a configuração de emparelhamento público do Azure

Você pode atualizar qualquer parte da configuração usando o cmdlet a seguir

	Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"

No exemplo acima, a ID de VLAN do circuito está sendo atualizada de 200 para 600.

### Excluir o emparelhamento público do Azure

Você pode remover a configuração de emparelhamento executando o seguinte cmdlet

	Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"

## Emparelhamento da Microsoft

Esta seção fornece instruções sobre como criar, obter, atualizar e excluir a configuração de emparelhamento da Microsoft para um circuito da Rota Expressa.

### Criar emparelhamento da Microsoft

1. **Importe o módulo do PowerShell para Rota Expressa.**
	
	Para começar a usar os cmdlets da Rota Expressa, é necessário importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell. Execute os seguintes comandos para importar os módulos do Azure e da Rota Expressa para a sessão do PowerShell.

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **Criar um circuito da Rota Expressa**
	
	Siga as instruções para criar um [circuito da Rota Expressa](expressroute-howto-circuit-classic.md) e para que o circuito seja provisionado pelo provedor de conectividade. Caso seu provedor de conectividade ofereça serviços gerenciados de camada 3, solicite a ele a habilitação do emparelhamento privado do Azure. Nesse caso, não será necessário seguir as instruções listadas nas seções a seguir. No entanto, se o seu provedor de conectividade não gerenciar o roteamento, após a criação do circuito, siga as instruções abaixo.

3. **Verifique se o circuito da Rota Expressa foi provisionado**

	Primeiro, verifique se o circuito da Rota Expressa está no estado Provisionado e Habilitado.

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : MyTestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	Verifique se o circuito aparece como Provisionado e Habilitado. Caso contrário, converse com seu provedor de conectividade para que o circuito fique com o status e o estado exigidos.

		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled


4. **Configurar o emparelhamento da Microsoft para o circuito**

	Você precisa ter as seguintes informações antes de continuar:

	- Uma sub-rede /30 para o link principal. Este valor deve ser um prefixo IPv4 público válido próprio e registrado em um RIR/IRR.
	- Uma sub-rede /30 para o link secundário. Este valor deve ser um prefixo IPv4 público válido próprio e registrado em um RIR/IRR.
	- Uma ID válida de VLAN para estabelecer esse emparelhamento. Verifique se nenhum outro emparelhamento no circuito usa a mesma ID de VLAN.
	- Número de AS para emparelhamento. Você pode usar um número de AS de 2 e de 4 bytes.
	- Prefixos anunciados: forneça uma lista com todos os prefixos que você pretende anunciar na sessão BGP. Somente prefixos de endereços IP públicos são aceitos. Caso você planeje enviar um conjunto de prefixos, envie uma lista separada por vírgulas. Esses prefixos devem ser registrados em seu nome em um RIR/IRR.
	- ASN de cliente: se você estiver anunciando prefixos não registrados com o número AS de emparelhamento, especifique o número AS com o qual eles estão registrados. **Isso é opcional**.
	- Nome do registro de roteamento: você pode especificar o RIR/IRR com base no qual o número de AS e os prefixos estão registrados.
	- Um hash MD5, se você optar por usar um. **Isso é opcional.**
	
	Execute o cmdlet a seguir para configurar o emparelhamento da Microsoft para seu circuito

		New-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"


### Para exibir detalhes de emparelhamento da Microsoft

Você pode obter detalhes de configuração usando o cmdlet a seguir.

	Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
	
	AdvertisedPublicPrefixes       : 123.0.0.0/30
	AdvertisedPublicPrefixesState  : Configured
	AzureAsn                       : 12076
	CustomerAutonomousSystemNumber : 2245
	PeerAsn                        : 1234
	PrimaryAzurePort               : 
	PrimaryPeerSubnet              : 10.0.0.0/30
	RoutingRegistryName            : ARIN
	SecondaryAzurePort             : 
	SecondaryPeerSubnet            : 10.0.0.4/30
	State                          : Enabled
	VlanId                         : 300


### Atualizar a configuração de emparelhamento da Microsoft

Você pode atualizar qualquer parte da configuração usando o cmdlet a seguir.

		Set-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"

### Excluir emparelhamento da Microsoft

Você pode remover a configuração de emparelhamento executando o seguinte cmdlet.

	Remove-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"

## Próximas etapas

Em seguida, [Vincular uma Rede Virtual a um circuito da Rota Expressa](expressroute-howto-linkvnet-classic.md).


-  Para obter mais informações sobre fluxos de trabalho, veja [Fluxos de trabalho da Rota Expressa](expressroute-workflows.md).
-  Para obter mais informações sobre o emparelhamento de circuito, veja [Circuitos e domínios de roteamento da Rota Expressa](expressroute-circuit-peerings.md).

<!---HONumber=AcomDC_0629_2016-->