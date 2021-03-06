<properties
	pageTitle="Diretrizes da infraestrutura de rede | Microsoft Azure"
	description="Saiba mais sobre as principais diretrizes de design e implementação para a implantação de rede virtual nos serviços de infraestrutura do Azure."
	documentationCenter=""
	services="virtual-machines-windows"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/30/2016"
	ms.author="iainfou"/>

# Diretrizes da infraestrutura de rede

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-intro](../../includes/virtual-machines-windows-infrastructure-guidelines-intro.md)]

Este artigo se concentra em compreender as etapas de planejamento necessárias para a rede virtual no Azure, e a conectividade entre ambientes locais existentes.


## Diretrizes de implementação de redes virtuais

Decisões:

- Que tipo de rede virtual é necessário para hospedar sua infraestrutura ou carga de trabalho de TI (somente em nuvem ou entre instalações)?
- Para redes virtuais entre instalações, quanto espaço de endereço você precisa para hospedar as sub-redes e VMs agora e para uma expansão razoável no futuro?
- Você vai criar redes virtuais centralizadas ou criar redes virtuais individuais para cada grupo de recursos?

Tarefas:

- Defina o espaço de endereço para a rede virtual a ser criada.
- Defina o conjunto de sub-redes e o espaço de endereço para cada uma.
- Para redes virtuais entre instalações, defina o conjunto de espaços de endereços de redes locais para as instalações que as VMs da rede virtual precisam acessar.
- Trabalhe com uma equipe local de rede para garantir a configuração do roteamento apropriado ao criar redes virtuais entre instalações.
- Crie a rede virtual usando a sua convenção de nomenclatura.


## Redes virtuais

Redes virtuais são necessárias para oferecer suporte a comunicações entre VMs (máquinas virtuais). Você pode definir sub-redes, endereços IP personalizados, configurações de DNS, filtragem de segurança e balanceamento de carga, assim como em redes físicas. Com o uso de uma [VPN Site a Site](../vpn-gateway/vpn-gateway-topology.md) ou [Circuito de Rota Expressa](../expressroute/expressroute-introduction.md), você pode conectar redes virtuais do Azure com suas redes locais. Leia mais sobre [redes virtuais e seus componentes](../virtual-network/virtual-networks-overview.md).

Com o uso de Grupos de Recursos, você tem flexibilidade para projetar os componentes da rede virtual. As VMs podem se conectar às redes virtuais de fora de seu próprio grupo de recursos. Uma abordagem de design comum seria criar grupos de recursos centralizados contendo sua infraestrutura de rede central, que possam ser gerenciados por uma equipe comum e, depois, implantar as VMs e seus aplicativos nos grupos de recursos separados. Isso permite o acesso dos proprietários do aplicativo ao grupo de recursos que contém suas VMs, sem abrir o acesso à configuração dos recursos rede virtual mais amplos.

## Conectividade de site

### Redes virtuais somente na nuvem
Se os computadores e usuários locais não exigirem conectividade contínua com as VMs em uma rede virtual do Azure, o design de sua rede virtual será bastante simples:

![Diagrama básico de rede virtual somente na nuvem](./media/virtual-machines-common-infrastructure-service-guidelines/vnet01.png)

Normalmente isso é usado para cargas de trabalho para a Internet, como um servidor Web baseado na Internet. Você pode gerenciar essas VMs usando conexões RDP ou VPN ponto a site.

Como elas não se conectam à sua rede local, as redes virtuais somente do Azure podem usar qualquer parte do espaço de endereço IP privado, mesmo se o espaço privado estiver sendo usado localmente.


### Redes virtuais entre instalações
Se os computadores e usuários locais exigirem conectividade contínua para VMs em uma rede virtual do Azure, crie uma rede virtual entre locais e conecte-se à sua rede local com uma Rota Expressa ou uma conexão VPN site a site.

![Diagrama de rede virtual entre instalações](./media/virtual-machines-common-infrastructure-service-guidelines/vnet02.png)

Nessa configuração, a rede virtual do Azure é essencialmente uma extensão baseada em nuvem da sua rede local.

Como eles se conectam à sua rede local, as redes virtuais entre instalações devem usar uma parte exclusiva do espaço de endereço usado por sua organização. Da mesma forma que diferentes locais da empresa receberão uma sub-rede IP específica, o Azure torna-se outro local à medida que você estende sua rede.

Para permitir que os pacotes viajem da sua rede virtual entre instalações para a sua rede local, você deve configurar o conjunto de prefixos de endereços locais relevantes como parte da definição da Rede Local para a rede virtual. Dependendo do espaço de endereço da rede virtual e do conjunto de locais relevantes na instalação, pode haver vários prefixos de endereço na Rede Local.

Você pode converter uma rede virtual somente em nuvem em uma rede virtual entre instalações, mas provavelmente será necessário renumerar o IP de seu espaço de endereço de rede virtual, suas sub-redes e as VMs que usam endereços IP estáticos do Azure. Portanto, considere cuidadosamente se uma rede virtual precisará estar conectada à sua rede local quando você atribuir uma sub-rede IP.

## Sub-redes
As sub-redes permitem organizar os recursos que estão relacionados, seja logicamente (por exemplo, uma sub-rede para VMs associadas ao mesmo aplicativo) ou fisicamente (por exemplo, uma sub-rede por grupo de recursos), ou empregar técnicas de isolamento de sub-redes para maior segurança.

Para redes virtuais entre instalações, você deve criar sub-redes com as mesmas convenções que você usa para recursos locais, tendo em mente que **o Azure sempre usa os três primeiros endereços IP do espaço de endereços de cada sub-rede**. Para determinar o número de endereços necessários para a sub-rede, conte o número de VMs de que você precisa agora, faça uma estimativa do crescimento futuro e, em seguida, use a tabela a seguir para determinar o tamanho da sub-rede.

Número de VMs necessárias | Número de bits de host necessários | Tamanho da sub-rede
--- | --- | ---
1 – 3 | 3 | / 29
4 – 11 | 4 | / 28
12 – 27 | 5 | / 27
28-59 | 6 | / 26
60 – 123 | 7 | / 25

> [AZURE.NOTE] Para sub-redes locais normais, o número máximo de endereços de host para uma sub-rede com n bits de host é 2<sup>n</sup> – 2. Para uma sub-rede do Azure, o número máximo de endereços de host para uma sub-rede com n bits de host é 2<sup>n</sup> – 5 (2 mais 3 para os endereços que o Azure usa em cada sub-rede).

Se você escolher um tamanho de sub-rede que seja muito pequeno, precisará renumerar o IP e reimplantar as VMs na sub-rede.


## Grupos de segurança de rede
Você pode aplicar regras de filtragem no tráfego que flui através de suas redes virtuais usando Grupos de Segurança de Rede. Você pode criar regras de filtragem muito granulares para proteger seu ambiente de rede virtual, controlar o tráfego de entrada e saída, intervalos de IP de origem e destino, portas permitidas etc. Grupos de Segurança de Rede podem ser aplicados a sub-redes de uma rede virtual ou diretamente em uma determinada interface de rede virtual. Recomendamos algum nível de filtragem de tráfego no Grupo de Segurança de Rede em suas redes virtuais. Você pode ler mais sobre[Grupos de Segurança de Rede](../virtual-network/virtual-networks-nsg.md).


## Componentes de rede adicionais
Assim como acontece com uma infraestrutura de rede física local, a rede virtual do Azure pode conter mais do que apenas sub-redes e endereçamento IP. Ao projetar a infraestrutura de seu aplicativo, convém incorporar alguns desses componentes adicionais:

- [Gateways de VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) - conecte redes virtuais do Azure a outras redes virtuais do Azure, redes locais através de uma conexão VPN Site a Site, forneça aos usuários acesso direto com conexões VPN Site a ponto ou implemente conexões de Rota Expressa para conexões dedicadas e seguras.
- [Balanceador de carga](../load-balancer/load-balancer-overview.md) - fornece balanceamento de carga de tráfego para o tráfego interno e externo, conforme o desejado.
- [Application Gateway](../application-gateway/application-gateway-introduction.md) - balanceamento de carga HTTP na camada do aplicativo, fornecendo alguns benefícios adicionais para aplicativos Web do que simplesmente implantar o balanceador de carga do Azure.
- [Gerenciador de Tráfego](../traffic-manager/traffic-manager-overview.md) - distribuição de tráfego baseado em DNS apara s usuários finais diretos para o ponto de extremidade do aplicativo disponível mais próximo, permitindo que você hospede seu aplicativo fora dos datacenters do Azure em regiões diferentes de tráfego.


## Próximas etapas

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-next-steps](../../includes/virtual-machines-windows-infrastructure-guidelines-next-steps.md)]

<!---HONumber=AcomDC_0706_2016-->