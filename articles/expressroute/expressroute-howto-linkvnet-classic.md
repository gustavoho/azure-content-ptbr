<properties
   pageTitle="Vincular uma rede virtual a um circuito de Rota Expressa usando o modelo de implantação clássico e o PowerShell | Microsoft Azure"
   description="Este documento fornece uma visão geral de como vincular as redes virtuais (VNets) aos circuitos de Rota Expressa usando o modelo de implantação clássico e o PowerShell."
   services="expressroute"
   documentationCenter="na"
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
   ms.date="04/14/2016"
   ms.author="ganesr" />

# Vincular uma rede virtual a um circuito de Rota Expressa

> [AZURE.SELECTOR]
- [Portal do Azure - Gerenciador de Recursos](expressroute-howto-linkvnet-portal-resource-manager.md)
- [PowerShell – Resource Manager](expressroute-howto-linkvnet-arm.md)
- [PowerShell - clássico](expressroute-howto-linkvnet-classic.md)



Este artigo o ajudará a vincular as redes virtuais (VNets) aos circuitos de Rota Expressa do Azure usando o modelo de implantação clássico e o PowerShell. As redes virtuais podem estar na mesma assinatura ou fazer parte de outra assinatura.

**Sobre modelos de implantação do Azure**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## Pré-requisitos de configuração

1. Você precisa da última versão dos módulos do Azure PowerShell. Baixe os módulos mais recente do PowerShell na seção PowerShell da [página Downloads do Azure](https://azure.microsoft.com/downloads/). Siga as instruções em [Como instalar e configurar o Azure PowerShell](../powershell-install-configure.md) para obter orientações passo a passo sobre como configurar o computador para usar os módulos do Azure PowerShell.
2. Leia os [pré-requisitos](expressroute-prerequisites.md), os [requisitos de roteamento](expressroute-routing.md) e os [fluxos de trabalho](expressroute-workflows.md) antes de começar a configuração.
3. Você deve ter um circuito da Rota Expressa ativo.
	- Siga as instruções para [criar um circuito da Rota Expressa](expressroute-howto-circuit-classic.md) e para que o provedor de conectividade habilite o circuito.
	- Verifique se o emparelhamento privado do Azure está configurado para seu circuito. Confira o artigo [Configurar roteamento](expressroute-howto-routing-classic.md) para obter instruções sobre roteamento.
	- Verifique se o emparelhamento privado do Azure está configurado e se o emparelhamento BGP entre sua rede e a Microsoft está ativo para que você possa habilitar a conectividade de ponta a ponta.
    - É necessário ter uma rede virtual e um gateway de rede virtual criados e totalmente provisionados. Siga as instruções para [configurar uma rede virtual para Rota Expressa](expressroute-howto-vnet-portal-classic.md).

Você pode vincular até 10 redes virtuais a um circuito de Rota Expressa. Todos os circuitos da Rota Expressa devem estar na mesma região geopolítica. É possível vincular um grande número de redes virtuais ao circuito da Rota Expressa se você tiver habilitado o complemento premium da Rota Expressa. Confira as [perguntas frequentes](expressroute-faqs.md) para obter mais detalhes sobre o complemento premium.

## Conectar uma rede virtual na mesma assinatura a um circuito

Você pode vincular uma rede virtual a um circuito da Rota Expressa usando o cmdlet a seguir. Verifique se o gateway de rede virtual foi criado e se está pronto para vinculação antes de executar o cmdlet.

	New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
	Provisioned

## Conectar uma rede virtual em uma assinatura diferente a um circuito

Você pode compartilhar um circuito da Rota Expressa entre várias assinaturas. A figura a seguir mostra um esquema simples de como funciona o compartilhamento de circuitos da Rota Expressa entre várias assinaturas.

Cada uma das nuvens menores dentro da nuvem grande é usada para representar assinaturas pertencentes a diferentes departamentos dentro de uma organização. Cada um dos departamentos dentro da organização pode usar sua própria assinatura para implantar seus serviços, mas os departamentos podem compartilhar um único circuito da Rota Expressa para se conectar de volta à respectiva rede local. Um único departamento (neste exemplo: TI) pode ter o circuito da Rota Expressa. Outras assinaturas dentro da organização podem usar o circuito de Rota Expressa.

>[AZURE.NOTE] As cobranças por conectividade e largura de banda do circuito dedicado serão aplicadas ao proprietário do circuito da Rota Expressa. Todas as redes virtuais compartilham a mesma largura de banda.

![Conectividade entre assinaturas](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### Administração

O *proprietário do circuito* é o administrador/coadministrador da assinatura na qual o circuito da Rota Expressa foi criado. O proprietário do circuito pode autorizar administradores/coadministradores de outras assinaturas, conhecidos como *usuários do circuito*, a usar o circuito dedicado que eles possuem. Os usuários do circuito autorizados a usar o circuito da Rota Expressa da organização podem vincular a rede virtual em sua assinatura ao circuito da Rota Expressa depois que são autorizados.

O proprietário do circuito tem a capacidade de modificar e revogar autorizações a qualquer momento. Revogar uma autorização fará com que todos os links sejam excluídos da assinatura cujo acesso foi revogado.

### Operações do proprietário do circuito

#### Criando uma autorização

O proprietário do circuito autoriza os administradores de outras assinaturas a usar o circuito especificado. No exemplo a seguir, o administrador do circuito (TI da Contoso) habilita o administrador de outra assinatura (Desenvolvimento/Teste) a vincular até duas redes virtuais ao circuito. O administrador de TI da Contoso habilita isso especificando a ID da Microsoft de Desenvolvimento/Teste. O cmdlet não envia um email para a ID da Microsoft especificada. O proprietário do circuito precisa notificar explicitamente o outro proprietário da assinatura de que a autorização foi concluída.

	New-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -Description "Dev-Test Links" -Limit 2 -MicrosoftIds 'devtest@contoso.com'

	Description         : Dev-Test Links
	Limit               : 2
	LinkAuthorizationId : **********************************
	MicrosoftIds        : devtest@contoso.com
	Used                : 0

#### Examinando autorizações

O proprietário do circuito pode examinar todas as autorizações emitidas em um circuito específico executando o seguinte cmdlet:

	Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: "**************************"

	Description         : EngineeringTeam
	Limit               : 3
	LinkAuthorizationId : ####################################
	MicrosoftIds        : engadmin@contoso.com
	Used                : 1

	Description         : MarketingTeam
	Limit               : 1
	LinkAuthorizationId : @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	MicrosoftIds        : marketingadmin@contoso.com
	Used                : 0

	Description         : Dev-Test Links
	Limit               : 2
	LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
	MicrosoftIds        : salesadmin@contoso.com
	Used                : 2


#### Atualizando autorizações

O proprietário do circuito pode modificar autorizações usando o seguinte cmdlet:

	Set-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -AuthorizationId "&&&&&&&&&&&&&&&&&&&&&&&&&&&&"-Limit 5

	Description         : Dev-Test Links
	Limit               : 5
	LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
	MicrosoftIds        : devtest@contoso.com
	Used                : 0


#### Excluindo autorizações

O proprietário do circuito pode revogar/excluir autorizações usando o seguinte cmdlet:

	Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey "*****************************" -AuthorizationId "###############################"


### Operações do usuário do circuito

#### Examinando autorizações

O usuário do circuito pode examinar autorizações usando o seguinte cmdlet:

	Get-AzureAuthorizedDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : ContosoIT
	Location                         : Washington DC
	MaximumAllowedLinks              : 2
	ServiceKey                       : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Status                           : Enabled
	UsedLinks                        : 0

#### Resgatando autorizações de link

O usuário de circuito pode executar o seguinte cmdlet para resgatar uma autorização de vínculo:

	New-AzureDedicatedCircuitLink –servicekey "&&&&&&&&&&&&&&&&&&&&&&&&&&" –VnetName 'SalesVNET1'

	State VnetName
	----- --------
	Provisioned SalesVNET1

## Próximas etapas

Para obter mais informações sobre a Rota Expressa, consulte [Perguntas Frequentes sobre Rota Expressa](expressroute-faqs.md).

<!---HONumber=AcomDC_0518_2016-->