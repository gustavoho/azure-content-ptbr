<properties
   pageTitle="Segurança de rede e serviços em nuvem da Microsoft | Microsoft Azure"
   description="Conheça alguns dos principais recursos disponíveis no Azure para ajudar a criar ambientes de rede segura"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="03/14/2016"
   ms.author="jonor;sivae"/>

# Segurança de rede e serviços em nuvem da Microsoft

Os serviços em nuvem da Microsoft entregam serviços e estrutura em larga escala, recursos de nível empresarial e várias opções de conectividade híbrida. Os clientes podem optar por acessar esses serviços através da Internet ou com a Rota Expressa do Azure, que fornece conectividade de rede privada. A plataforma Microsoft Azure permite que os clientes estendam facilmente sua infraestrutura para a nuvem e criem arquiteturas com várias camadas. Além disso, terceiros podem habilitar recursos avançados oferecendo serviços de segurança e soluções de virtualização. Este white paper fornece uma visão geral sobre segurança e problemas de arquitetura que os clientes devem considerar ao usar os serviços em nuvem da Microsoft acessados através da Rota Expressa. Ele também aborda a criação de serviços mais seguros em redes virtuais do Azure.

## Início rápido
O gráfico lógico a seguir pode direcioná-lo a um exemplo específico das várias técnicas de segurança disponíveis na plataforma Azure. Para referência rápida, encontre o exemplo que melhor se ajusta a seu caso. Para obter explicações mais completas, continue lendo o artigo. ![Fluxograma de opções de segurança][0]

[Exemplo 1: crie uma rede de perímetro (também conhecida como DMZ, zona desmilitarizada, e sub-rede filtrada) para ajudar a proteger aplicativos com NSGs (grupos de segurança de rede).](#example-1-build-a-simple-dmz-with-nsgs)</br> [Exemplo 2: crie uma rede de perímetro para proteger aplicativos com um firewall e NSGs.](#example-2-build-a-dmz-to-protect-applications-with-a-firewall-and-nsgs)</br> [Exemplo 3: crie uma rede de perímetro para ajudar a proteger as redes com um firewall, UDR (rota definida pelo usuário) e NSG.](#example-3-build-a-dmz-to-protect-networks-with-a-firewall-udr-and-nsg)</br> [Exemplo 4: adicione uma conexão híbrida com uma VPN (rede virtual privada) site a site de solução de virtualização.](#example-4-adding-a-hybrid-connection-with-a-site-to-site-virtual-appliance-vpn)</br> [Exemplo 5: adicione uma conexão híbrida com uma VPN site a site de gateway do Azure.](#example-5-adding-a-hybrid-connection-with-a-site-to-site-azure-gateway-vpn)</br> [Exemplo 6: adicione uma conexão híbrida com a Rota Expressa.](#example-6-adding-a-hybrid-connection-with-expressroute)</br> Exemplos para adicionar conexões entre redes virtuais, alta disponibilidade e encadeamento de serviços serão adicionados a esse documento durante os próximos meses.

## Proteção de infraestrutura e conformidade da Microsoft
A Microsoft tomou uma posição de liderança dando suporte às iniciativas de conformidade exigidas pelos clientes corporativos. Seguem algumas das certificações de conformidade do Azure: ![Selos de conformidade do Azure][1]

Para obter mais detalhes, consulte as informações de conformidade no [Microsoft Trust Center](https://azure.microsoft.com/support/trust-center/compliance/).

A Microsoft tem uma abordagem abrangente para proteger a infraestrutura de nuvem necessária para executar serviços globais em larga escala. Além dos datacenters físicos, a infraestrutura de nuvem da Microsoft também inclui hardware, software, redes, administrativas e equipe de operações.

![Recursos de segurança do Azure][2]

A abordagem acima fornece uma base segura para os clientes implantarem seus serviços na Microsoft Cloud. A próxima etapa é o design e a criação de uma arquitetura de segurança pelos clientes para proteger esses serviços.

## Arquiteturas de segurança tradicionais e redes de perímetro
Embora a Microsoft faça investimentos consideráveis para proteger a infraestrutura de nuvem, os clientes também devem proteger os serviços de nuvem e os grupos de recursos. Uma abordagem multicamada para a segurança oferece a melhor defesa. Uma zona de segurança de rede de perímetro protege recursos da rede interna de uma rede não confiável. Uma rede de perímetro refere-se às bordas ou partes da rede que ficam entre a Internet e a infraestrutura de TI da empresa protegida.

Em redes empresariais típicas, a infraestrutura básica é reforçada intensamente no perímetro, com várias camadas de dispositivos de segurança. O limite de cada camada consiste em dispositivos e pontos de imposição de políticas. Os dispositivos podem incluir o seguinte: firewalls, prevenção de DDoS (ataque de negação de serviço distribuído), IDS/IPS (sistemas de detecção de intrusão ou sistemas de proteção) e dispositivos VPN. A imposição de políticas pode assumir a forma de políticas de firewall, ACLs (listas de controle de acesso) ou roteamento específico. A primeira linha de defesa da rede, que aceita diretamente o tráfego de entrada da Internet, é uma combinação desses mecanismos para bloquear ataques e tráfego perigoso, permitindo que solicitações legítimas adentrem a rede. Esse tráfego é roteado diretamente aos recursos na rede de perímetro. Esse recurso pode "falar" com recursos mais dentro da rede, passando primeiro pelo próximo limite de validação. A camada mais externa é chamada de a rede de perímetro, porque essa parte da rede está exposta à Internet, geralmente com alguma forma de proteção em ambos os lados. A figura a seguir mostra um exemplo de uma rede de perímetro de sub-rede individual em uma rede corporativa, com dois limites de segurança.

![Uma rede de perímetro em uma rede corporativa][3]

Há muitas arquiteturas usadas para implementar uma rede de perímetro, de um simples do balanceador de carga na frente de um Web farm até uma rede de perímetro com várias sub-redes e diferentes mecanismos em cada limite para bloquear o tráfego e proteger as camadas mais profundas da rede corporativa. Como a rede de perímetro é criada depende das necessidades específicas da organização e da tolerância a riscos geral.

Conforme os clientes vão movendo suas cargas de trabalho para nuvens públicas, é essencial dar suporte a recursos semelhantes de arquitetura de rede de perímetro no Azure para atender aos requisitos de conformidade e de segurança. Este documento fornece diretrizes sobre como os clientes podem criar um ambiente de rede segura no Azure. Ele se concentra na rede de perímetro, mas também inclui uma discussão abrangente sobre muitos aspectos de segurança de rede. As perguntas a seguir informam essa discussão:

- Como é possível criar uma rede de perímetro no Azure?
- Quais são alguns dos recursos do Azure disponíveis para criar a rede de perímetro?
- Como cargas de trabalho de back-end podem ser protegidas?
- Como a comunicação com a Internet é controlada para as cargas de trabalho no Azure?
- Como as redes locais podem ser protegidas contra implantações no Azure?
- Quando recursos de segurança nativos do Azure devem ser usados em vez de dispositivos ou serviços de terceiros?

O diagrama a seguir mostra várias camadas de segurança que Azure fornece aos clientes. Essas camadas são tanto nativas na plataforma Azure em si quanto recursos definidos pelo cliente:

![Arquitetura de segurança do Azure][4]

Vinda da Internet a proteção DDoS do Azure ajuda a proteger contra ataques em grande escala contra o Azure. A próxima camada é de pontos de extremidade públicos definidos pelo cliente, que são usados para determinar qual tráfego pode passar pelo serviço de nuvem para a rede virtual. O isolamento de rede virtual Nativa do Azure garante o isolamento completo de todas as outras redes e garante que o tráfego flua somente através de métodos e caminhos configurados pelo usuário. Esses caminhos e métodos são a próxima camada em que NSGs, UDR e soluções de virtualização de rede podem ser usados para criar limites de segurança para proteger as implantações de aplicativo na rede protegida.

A próxima seção fornece uma visão geral das redes virtuais do Azure. As redes virtuais do Azure são criadas por clientes e tratam-se daquilo a que as cargas de trabalho implantadas por esses clientes estão conectadas. Redes virtuais são a base de todos os recursos de segurança de rede necessários para estabelecer uma rede de perímetro e proteger as implantações dos clientes no Azure.

## Visão geral das redes virtuais do Azure
Antes que o tráfego da Internet possa acessar as redes virtuais do Azure, há duas camadas de segurança inerentes à plataforma Azure:

1.	**Proteção DDoS**: a proteção DDoS é uma camada de rede física do Azure que protege a plataforma do Azure em si contra ataques em larga escala baseados na Internet. Esses ataques usam vários nós de "bot" em uma tentativa de sobrecarregar um serviço de Internet. O Azure tem uma malha de proteção sólida DDoS em todas as conexões de Internet de entrada. Essa camada de proteção DDoS não tem atributos configuráveis pelo usuário e não é acessível ao cliente. Isso protege o Azure, como plataforma, de ataques em grande escala, mas não protege diretamente o aplicativo cliente individual. Camadas adicionais de resiliência podem ser configuradas pelo cliente contra um ataque localizado. Por exemplo, se um cliente “A” foi atacado com DDoS em grande escala em um ponto de extremidade público, o Azure bloqueia as conexões a esse serviço. Um cliente pode realizar o failover a outra rede virtual ou ponto de extremidade de serviço não envolvidos com o ataque para restaurar o serviço. Observe que embora um cliente A possa ser afetado naquele ponto de extremidade, nenhum outro serviço fora desse ponto de extremidade seria afetado. Além disso, outros clientes e serviços não veem nenhum impacto vindo desse ataque.
2.	**Pontos de extremidade de serviço**: pontos de extremidade permitem que serviços de nuvem ou grupos de recursos tenham portas e endereços IP de Internet públicos e expostos. O ponto de extremidade usará NAT (Conversão de Endereços de Rede) para rotear o tráfego para a porta e endereço internos na rede virtual do Azure. Esse é o caminho principal por onde o tráfego externo passa para a rede virtual. Os pontos de extremidade de serviço são configuráveis pelo usuário para determinar tanto qual tráfego pode passar quanto como e para onde esse tráfego deve ser convertido na rede virtual.

Depois que o tráfego alcança a rede virtual, há muitos recursos que entram em cena. Redes virtuais do Azure são a base para que clientes possam anexar suas cargas de trabalho e o local em que a segurança no nível da rede básica se aplica. É uma rede privada (uma sobreposição de rede virtual) no Azure para clientes com os seguintes recursos e características:
 
- **Isolamento de tráfego**: uma rede virtual é o limite de isolamento de tráfego na plataforma Azure. As VMs (máquinas virtuais) em uma rede virtual não podem comunicar-se diretamente com VMs em uma rede virtual diferente, mesmo que ambas as redes virtuais sejam criadas pelo mesmo cliente. Essa é uma propriedade vital que garante que as VMs e as comunicações do cliente permaneçam privadas em uma rede virtual.
- **Topologia multicamadas**: as redes virtuais permitem que os clientes definam a topologia multicamadas alocando sub-redes e designando espaços de endereço separados para elementos ou "camadas" diferentes de suas cargas de trabalho. Esses agrupamentos e topologias lógicos permitem que os clientes definam políticas de acesso diferentes baseadas nos tipos de carga de trabalho e também controlam fluxos de tráfego entre as camadas.
- **Conectividade entre locais**: os clientes podem estabelecer a conectividade entre locais entre uma rede virtual e vários sites locais ou outras redes virtuais no Azure. Para fazer isso, os clientes podem usar Gateways de VPN do Azure ou soluções de virtualização de rede de terceiros. O Azure dá suporte a VPNs site a site (S2S) usando protocolos padrão de IPsec/IKE e conectividade privada de Rota Expressa.
- O **NSG** permite que os clientes criem regras (ACLs) no nível desejado de granularidade: sub-redes virtuais, VMs individuais ou interfaces de rede. Os clientes podem controlar o acesso permitindo ou negando a comunicação entre as cargas de trabalho em uma rede virtual, de sistemas nas redes do cliente por meio de conectividade entre locais ou comunicação direta via Internet.
- **UDR** e **Encaminhamento IP** permitem que os clientes definam os caminhos de comunicação entre camadas diferentes em uma rede virtual. Os clientes podem implantar firewall, IDS/IPS e outras soluções de virtualização, além de rotear o tráfego de rede por meio dessas soluções de segurança para a aplicação, auditoria e inspeção das políticas de limites de segurança.
- **Soluções de virtualização de rede** no Azure Marketplace: dispositivos de segurança, como firewalls, balanceadores de carga e IDS/IPS estão disponíveis no Azure Marketplace e na Galeria de Imagens de VM. Os clientes podem implantar essas soluções em suas redes virtuais e, especificamente, em seus limites de segurança (incluindo as sub-redes da rede de perímetro) para concluir um ambiente de rede segura com várias camadas.

Com esses recursos e funções, um exemplo de como uma arquitetura de rede de perímetro poderia ser criada no Azure é o seguinte:

![Uma rede de perímetro em uma rede virtual do Azure][5]

## Requisitos e características da rede de perímetro
A rede de perímetro é projetada para ser o front-end da rede, fazendo a interface de comunicação diretamente da Internet. Os pacotes de entrada devem fluir por dispositivos de segurança, como firewall, IDS, IPS e afins, antes de alcançar os servidores back-end. Pacotes das cargas de trabalho associados à Internet também podem fluir pelos dispositivos de segurança na rede de perímetro para fins de auditoria, inspeção e imposição da política antes de sair da rede. Além disso, a rede de perímetro pode hospedar os gateways de VPN entre locais entre redes virtuais de cliente e redes locais.

### Características da rede de perímetro
Algumas das características de uma boa rede de perímetro, referenciando a figura anterior, são as seguintes:

- Voltada para a Internet:
    - A sub-rede da rede de perímetro em si é voltada para a Internet, comunicando-se diretamente com ela.
    - IPs públicas, VIPs e/ou pontos de extremidade de serviço passam tráfego da Internet para a rede e dispositivos de front-end.
    - O tráfego de entrada da Internet passa por dispositivos de segurança antes de outros recursos na rede de front-end.
    - Se segurança de saída estiver habilitada, o tráfego passará por dispositivos de segurança, como a etapa final, antes de passar para a Internet.
- Rede protegida:
    - Não há caminho direto da Internet para a infraestrutura básica.
    - Os canais para a infraestrutura básica devem percorrer dispositivos de segurança, como NSGs, firewalls ou dispositivos VPN.
    - Outros dispositivos não devem ligar a Internet à infraestrutura básica.
    - Dispositivos de segurança tanto no limite para a Internet quanto nos limites voltados à rede protegida da rede de perímetro (por exemplo, os dois ícones de firewall mostrados na figura anterior) podem, na verdade, ser uma única solução de virtualização com regras ou interfaces diferenciadas para cada limite. (Ou seja, um dispositivo, logicamente separado, lidando com a carga para ambos os limites da rede de perímetro.)
- Outras práticas e restrições comuns:
    - As cargas de trabalho não devem armazenar informações essenciais aos negócios.
    - O acesso e as atualizações para configurações e implantações da rede de perímetro são limitados somente aos administradores autorizados.

### Requisitos de rede de perímetro
Para habilitar essas características, siga essas diretrizes sobre requisitos de rede virtual para implementar uma rede de perímetro com êxito:

- **Arquitetura de sub-rede:** especifique a rede virtual de modo que uma sub-rede inteira seja dedicada como a rede de perímetro, separada de outras sub-redes na mesma rede virtual. Isso garante o tráfego entre a rede de perímetro e outras camadas de sub-rede internas ou privadas flua por meio de um firewall ou solução de virtualização IDS/IPS nos limites da sub-rede com rotas definidas pelo usuário.
- **NSG:** a sub-rede da rede de perímetro em si deve estar aberta para permitir a comunicação com a Internet, mas isso não significa que os clientes devem ignorar os NSGs. Siga as práticas de segurança comuns para minimizar as superfícies de rede expostas à Internet. Bloqueie os intervalos de endereços remotos com permissão para acessar as implantações ou os protocolos de aplicativo específicos e as portas que estão abertas. Pode haver circunstâncias, porém, em que isso nem sempre é possível. Por exemplo, se os clientes têm um site externo no Azure, a rede de perímetro deve permitir solicitações de entrada da Web de endereços IP públicos, mas só deve abrir as portas do aplicativo Web: TCP:80 e TCP:443.
- **Tabela de roteamento:** a sub-rede da rede de perímetro em si deve ser capaz de se comunicar diretamente com a Internet, mas não deve permitir a comunicação direta de e para o back-end ou em redes locais sem passar por um dispositivo de firewall ou de segurança.
- **Configuração de dispositivo de segurança:** para rotear e inspecionar os pacotes entre a rede de perímetro e o restante das redes protegidas, os dispositivos de segurança, como dispositivos IPS, IDS e firewall podem ter vários locais de origem. Eles podem ter NICs separadas para a rede de perímetro e as sub-redes de back-end. As NICs na rede de perímetro se comunicarão diretamente de e para a Internet, com os NSGs correspondentes e a tabela de roteamento da rede de perímetro. As NICs conectadas às sub-redes de back-end têm NSGs e tabelas de roteamento mais restritos das sub-redes correspondentes do back-end.
- **Funcionalidade de dispositivo de segurança:** os dispositivos de segurança implantados na rede de perímetro normalmente executam a seguinte funcionalidade:
    - Firewall: imposição de regras de firewall ou políticas de controle de acesso para solicitações de entrada.
    - Detecção e prevenção de ameaças: detecta ameaças e atenua ataques da Internet.
    - Auditoria e registro em log: mantém logs detalhados de auditoria e análise.
    - Proxy reverso: redireciona as solicitações de entrada para os servidores back-end correspondentes. Isso envolve o mapeamento e conversão de endereços de destino nos dispositivos de front-end, normalmente firewalls, para os endereços de servidores back-end.
    - Proxy de encaminhamento: fornece NAT e também realiza auditoria para a comunicação iniciada de dentro da rede virtual para a Internet.
    - Roteador: encaminha tráfego de entrada de sub-rede e entre sub-redes dentro da rede virtual.
    - Dispositivo VPN: atua como os gateways de VPN entre instalações para conectividade VPN entre instalações entre redes locais do cliente e redes virtuais do Azure.
    - Servidor VPN: aceita clientes VPN que se conectam a redes virtuais do Azure.

>[AZURE.TIP] Mantenha os dois grupos a seguir separados: as pessoas autorizadas a acessar o mecanismo de segurança da rede de perímetro e as pessoas autorizadas como administradores de operações, implantação ou desenvolvimento de aplicativos. Manter esses grupos separados permite uma diferenciação de deveres e impede que uma única pessoa ignore controles de segurança de aplicativos e de rede.

### Perguntas a serem feitas durante a criação de limites de rede
Nesta seção, a menos que especificamente mencionadas, o termo "redes" refere-se a redes virtuais privadas do Azure criadas por um administrador de assinatura. O termo não se refere às redes físicas subjacentes no Azure.

Além disso, as redes virtuais do Azure geralmente são usadas para estender redes locais tradicionais. É possível incorporar soluções de rede híbridas site a site ou de Rota Expressa com arquiteturas de rede de perímetro. Essa é uma consideração importante na criação de limites de segurança de rede.

É essencial responder as três perguntas a seguir quando você está criando uma rede com uma rede de perímetro e vários limites de segurança.

#### 1) Quantos limites são necessários?
O primeiro ponto de decisão é decidir quantos limites de segurança são necessários em um determinado cenário:

- Um único limite: um na rede de perímetro de front-end, entre a rede virtual e a Internet.
- Dois limites: um do lado da Internet da rede de perímetro, outro entre a sub-rede da rede de perímetro e as sub-redes de back-end nas redes virtuais do Azure.
- Três limites: o primeiro do lado da Internet da rede de perímetro, o segundo entre a sub-rede da rede de perímetro e as sub-redes de back-end e o terceiro entre as sub-redes de back-end e a rede local.
- Limites de N: um número variável. Dependendo dos requisitos de segurança, pode-se aplicar qualquer número de limites de segurança a uma determinada rede.

O número e tipo dos limites necessários irão variar dependendo da tolerância a riscos da empresa e o cenário específico que está sendo implementado. Geralmente, essa é uma decisão conjunta feita por vários grupos dentro de uma organização, geralmente incluindo uma equipe de risco e conformidade, uma equipe de rede e plataforma e uma equipe de desenvolvimento de aplicativos. As pessoas com conhecimento de segurança, os dados envolvidos e as tecnologias usadas devem pesar na decisão para garantir a postura de segurança apropriada em cada implementação.

>[AZURE.TIP] Use a menor quantidade de limites que satisfaçam os requisitos de segurança em uma determinada situação. Com mais limites, mais difíceis ficam as operações e a solução de problemas, bem como a sobrecarga do gerenciamento envolvido com a gestão de várias políticas de limites ao longo do tempo. No entanto, os limites insuficientes aumentam o risco. Encontrar o equilíbrio é fundamental.

![Rede híbrida com três limites de segurança][6]

A figura anterior mostra uma visão geral de uma rede de limite com três limites de segurança. Os limites estão entre a rede de perímetro e a Internet, entre as sub-redes privadas de front-end e back-end do Azure e entre a sub-rede de back-end do Azure e rede corporativa local.

#### 2) Onde estão localizados os limites?
Depois que o número de limites é decidido, onde implementá-los é o próximo ponto a se decidir. Geralmente, há três opções:
- Usar um serviço intermediário baseado na Internet (por exemplo, um firewall do aplicativo Web baseado em nuvem, que não é discutido neste documento)
- Usar recursos nativos e/ou soluções de virtualização de rede no Azure
- Usar dispositivos físicos na rede local

Em redes puramente do Azure, as opções são recursos nativos do Azure (por exemplo, Balanceadores de Carga do Azure) ou soluções de virtualização de rede do ecossistema variado de parceiros do Azure (por exemplo, firewalls de Ponto de Verificação).

Se for necessário um limite entre o Azure e uma rede local, os dispositivos de segurança poderão residir em qualquer um dos lados da conexão (ou em ambos). Assim, uma decisão deve ser tomada sobre o local para colocar o mecanismo de segurança.

Na figura anterior, os limites Internet para rede de perímetro e front-end para back-end estão totalmente contidos no Azure e devem ser recursos nativos do Azure ou soluções de virtualização de rede. Os dispositivos de segurança no limite entre o Azure (sub-rede de back-end) e a rede corporativa podem estar no lado do Azure ou no lado local, ou até como uma combinação de dispositivos em ambos os lados. Pode haver vantagens e desvantagens relevantes em ambas as opções; portanto, elas devem ser levadas em consideração.

Por exemplo, usar o mecanismo de segurança físico existente no lado da rede local tem a vantagem de não haver necessidade de nenhuma nova engrenagem. Ele precisa apenas de reconfiguração. A desvantagem, no entanto, é que todo o tráfego deve voltar do Azure para a rede local para ser visto pelo mecanismo de segurança. Assim, o tráfego do Azure para o Azure poderia incorrer em latência significativa e afetar a experiência do usuário e o desempenho do aplicativo se ele fosse forçado novamente à rede local para a imposição de política de segurança.

#### 3) Como os limites são implementados?
Cada limite de segurança provavelmente terá requisitos de capacidade diferentes (por exemplo, IDS e regras de firewall no lado da Internet da rede de perímetro, mas somente ACLs entre a rede de perímetro e a sub-rede de back-end). A decisão de quais dispositivos usar depende dos requisitos de cenário e de segurança. Na seção a seguir, os exemplos 1, 2 e 3 discutem algumas opções que podem ser usadas. A análise dos recursos de rede nativos do Azure e dos dispositivos disponíveis no Azure do ecossistema de parceiros mostra as inúmeras opções disponíveis para resolver praticamente qualquer cenário.

Outro ponto-chave de decisão de implementação é como se conectar à rede local com o Azure. Você deveria usar o gateway virtual do Azure ou uma solução de virtualização de rede? Essas opções são discutidas mais detalhadamente na seção a seguir (exemplos 4, 5 e 6).

Além disso, o tráfego entre redes virtuais no Azure pode ser necessário. Esses cenários serão adicionados posteriormente.

Depois de saber as respostas para as perguntas acima, a seção [Início Rápido](#fast-start) pode ajudá-lo a identificar quais exemplos são mais apropriados para um determinado cenário.

## Exemplos: criando limites de segurança com redes virtuais do Azure
### Exemplo 1: Criar uma rede de perímetro para proteger aplicativos com NSGs
[Voltar ao Início rápido](#fast-start) | [Instruções detalhadas de build para este exemplo][Example1]

![Rede de perímetro de entrada com NSG][7]

#### Descrição do ambiente
Neste exemplo, há uma assinatura que contém o seguinte:

- Dois serviços de nuvem: "FrontEnd001" e "BackEnd001"
- Uma rede virtual, “CorpNetwork”, com duas sub-redes: “FrontEnd” e “BackEnd”
- Um grupo de segurança de rede que é aplicado a ambas as sub-redes
- Um servidor Windows que representa um servidor Web de aplicativos ("IIS01")
- Dois servidores Windows que representam servidores de back-end de aplicativos ("AppVM01", "AppVM02")
- Um servidor Windows que representa um servidor DNS ("DNS01")

Para scripts e um modelo do Azure Resource Manager, consulte as [instruções detalhadas de build][Example1].

#### Descrição de NSG
Neste exemplo, um grupo NSG é criado e então carregado com seis regras.

>[AZURE.TIP] Em geral, você deve criar suas regras específicas "Permitir" primeiro, seguidas pelas regras “Negar” de caráter mais genérico. A prioridade dada determina quais regras são avaliadas primeiro. Quando o tráfego se aplicar a uma regra específica, nenhuma regra adicional será avaliada. As regras NSG podem se aplicar na direção de entrada ou de saída (na perspectiva da sub-rede).

Declarativamente, as regras a seguir estão sendo criadas para tráfego de entrada:

1.	O tráfego interno de DNS (porta 53) é permitido.
2.	O tráfego de RDP (porta 3389) da Internet para qualquer Máquina Virtual é permitido.
3.	O tráfego HTTP (porta 80) da Internet para o servidor Web (IIS01) é permitido.
4.	Todo o tráfego (todas as portas) de IIS01 para AppVM1 é permitido.
5.	Todo o tráfego (todas as portas) da Internet para a rede virtual inteira (ambas as sub-redes) é negado.
6.	Todo o tráfego (todas as portas) da sub-rede de front-end para a sub-rede de back-end é negado.

Com essas regras associado a cada sub-rede, se uma solicitação HTTP entrou pela Internet no servidor Web, as duas regras 3 (permitir) e 5 (negar) seriam aplicáveis. Mas como a regra 3 tem uma prioridade mais alta, apenas ela seria aplicável e a regra 5 não seria entram em cena. Portanto, a solicitação HTTP seria permitida para o servidor Web. Se esse mesmo tráfego estivesse tentando acessar o servidor DNS01, a regra 5 (negar) seria a primeira a ser aplicada e o tráfego não teria permissão para passar para o servidor. A regra 6 (negar) bloqueia a comunicação entre a sub-rede de front-end e a sub-rede de back-end (exceto pelo tráfego permitido nas regras 1 e 4). Isso protege a rede de back-end no caso de um invasor comprometer o aplicativo Web no front-end. O invasor teria acesso limitado à rede back-end "protegida" (apenas para recursos expostos no servidor AppVM01).

Há uma regra de saída padrão que permite o tráfego de saída para a Internet. Neste exemplo, permitiremos o tráfego de saída e não modificaremos quaisquer regras de saída. Para bloquear o tráfego em ambos os trajetos, é necessário o roteamento definido pelo usuário (veja o exemplo 3).

#### Conclusão
Essa é uma maneira relativamente simples e direta de isolar a sub-rede de back-end do tráfego de entrada. Para obter mais informações, consulte as [instruções detalhadas de build][Example1]. Essas instruções incluem:

- Como criar essa rede de perímetro com scripts do PowerShell.
- Como criar essa rede de perímetro com um modelo do Azure Resource Manager.
- Descrições detalhadas de cada comando NSG.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego é permitido ou negado em cada camada.


 ### Exemplo 2: Criar uma rede de perímetro para ajudar a proteger aplicativos com um firewall e NSGs [Voltar ao Início rápido](#fast-start) | [Instruções detalhadas de build para este exemplo][Example2]

![Rede de perímetro de entrada com NVA e NSG][8]

#### Descrição do ambiente
Neste exemplo, há uma assinatura que contém o seguinte:

- Dois serviços de nuvem: "FrontEnd001" e "BackEnd001"
- Uma rede virtual, “CorpNetwork”, com duas sub-redes: “FrontEnd” e “BackEnd”
- Um grupo de segurança de rede que é aplicado a ambas as sub-redes
- Uma solução de virtualização de rede, neste caso um firewall, conectada à sub-rede de front-end
- Um servidor Windows que representa um servidor Web de aplicativos ("IIS01")
- Dois servidores Windows que representam servidores de back-end de aplicativos ("AppVM01", "AppVM02")
- Um servidor Windows que representa um servidor DNS ("DNS01")

Para scripts e um modelo do Azure Resource Manager, consulte as [instruções detalhadas de build][Example2].

#### Descrição de NSG
Neste exemplo, um grupo NSG é criado e então carregado com seis regras.

>[AZURE.TIP] Em geral, você deve criar suas regras específicas "Permitir" primeiro, seguidas pelas regras “Negar” de caráter mais genérico. A prioridade dada determina quais regras são avaliadas primeiro. Quando o tráfego se aplicar a uma regra específica, nenhuma regra adicional será avaliada. As regras NSG podem se aplicar na direção de entrada ou de saída (na perspectiva da sub-rede).

Declarativamente, as regras a seguir estão sendo criadas para tráfego de entrada:

1.	O tráfego interno de DNS (porta 53) é permitido.
2.	O tráfego de RDP (porta 3389) da Internet para qualquer Máquina Virtual é permitido.
3.	Todo tráfego da Internet (todas as portas) para a solução de virtualização de rede (firewall) é permitido.
4.	Todo o tráfego (todas as portas) de IIS01 para AppVM1 é permitido.
5.	Todo o tráfego (todas as portas) da Internet para a rede virtual inteira (ambas as sub-redes) é negado.
6.	Todo o tráfego (todas as portas) da sub-rede de front-end para a sub-rede de back-end é negado.

Com essas regras associadas a cada sub-rede, se uma solicitação HTTP entrasse da Internet para firewall, as duas regras 3 (permitir) e 5 (negar) seriam aplicáveis. Mas como a regra 3 tem uma prioridade mais alta, apenas ela seria aplicável e a regra 5 não seria entram em cena. Assim, a solicitação HTTP seria permitida ao firewall. Se esse mesmo tráfego estava tentando alcançar o servidor IIS01, mesmo que ele estivesse na sub-rede de front-end, a regra 5 (negar) seria aplicável e o tráfego não teria permissão para passar para o servidor. A regra 6 (negar) bloqueia a comunicação entre a sub-rede de front-end e a sub-rede de back-end (exceto pelo tráfego permitido nas regras 1 e 4). Isso protege a rede de back-end no caso de um invasor comprometer o aplicativo Web no front-end. O invasor teria acesso limitado à rede back-end "protegida" (apenas para recursos expostos no servidor AppVM01).

Há uma regra de saída padrão que permite o tráfego de saída para a Internet. Neste exemplo, permitiremos o tráfego de saída e não modificaremos quaisquer regras de saída. Para bloquear o tráfego em ambos os trajetos, é necessário o roteamento definido pelo usuário (veja o exemplo 3).

#### Descrição da regra de firewall
No firewall, será necessário criar regras de encaminhamento. Como este exemplo só roteia o tráfego de Internet de entrada para o firewall e, em seguida, para o servidor Web, somente uma regra de NAT (conversão de endereços de rede) de encaminhamento é necessária.

A regra de encaminhamento aceita qualquer endereço de origem de entrada que acesse o firewall tentando alcançar HTTP (porta 80 ou 443 para HTTPS). Ela sai da interface local do firewall e é redirecionada para o servidor Web com o Endereço IP 10.0.1.5.

#### Conclusão
Essa é uma maneira relativamente simples de proteger seu aplicativo com um firewall e isolar a sub-rede de back-end do tráfego de entrada. Para obter mais informações, consulte as [instruções detalhadas de build][Example2]. Essas instruções incluem:

- Como criar essa rede de perímetro com scripts do PowerShell.
- Como criar esse exemplo com um modelo do Azure Resource Manager.
- Descrições detalhadas de cada comando NSG e regra de firewall.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego é permitido ou negado em cada camada.

### Exemplo 3: Criar uma rede de perímetro para ajudar a proteger as redes com um firewall, UDR e NSG
[Voltar ao Início rápido](#fast-start) | [Instruções detalhadas de build para este exemplo][Example3]

![Rede de perímetro bidirecional com NVA, NSG e UDR][9]

#### Descrição do ambiente
Neste exemplo, há uma assinatura que contém o seguinte:

- Três serviços de nuvem: “SecSvc001”, “FrontEnd001” e “BackEnd001”
- Uma rede virtual, “CorpNetwork”, com três sub-redes; “SecNet”, “FrontEnd” e “BackEnd”
- Uma solução de virtualização de rede, neste caso um firewall, conectada à sub-rede SecNet
- Um servidor Windows que representa um servidor Web de aplicativos ("IIS01")
- Dois servidores Windows que representam servidores de back-end de aplicativos ("AppVM01", "AppVM02")
- Um servidor Windows que representa um servidor DNS ("DNS01")

Para scripts e um modelo do Azure Resource Manager, consulte as [instruções detalhadas de build][Example3].

#### Descrição de UDR
Por padrão, as rotas do sistema a seguir são definidas como:

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.0.0/16}     VNETLocal                            Active   Default    
         {0.0.0.0/0}       Internet                             Active   Default    
         {10.0.0.0/8}      Null                                 Active   Default    
         {100.64.0.0/10}   Null                                 Active   Default    
         {172.16.0.0/12}   Null                                 Active   Default    
         {192.168.0.0/16}  Null                                 Active   Default

O VNETLocal é sempre o(s) prefixo(s) de endereço definido(s) da rede virtual para essa rede específica (ou seja, ele muda de rede virtual para rede virtual, dependendo de como cada rede virtual específica é definida). As rotas do sistema restantes são estáticas e padrão conforme indicado na tabela.

Neste exemplo, duas tabelas de roteamento são criadas, uma para a sub-rede de front-end e a outra para a sub-rede de back-end. Cada tabela é carregada com as rotas estáticas apropriadas para determinada sub-rede. Nesse exemplo, cada tabela tem três rotas que direcionam todo o tráfego (0.0.0.0/0) através do firewall (Próximo salto = endereço IP da Solução de Virtualização):

1. O tráfego de sub-rede local sem Próximo Salto definido para permitir que o tráfego de sub-rede local ignore o firewall.
2. Tráfego de rede virtual com um Próximo Salto definido como firewall, isso substitui a regra padrão que permite que o tráfego de rede virtual local seja roteado diretamente.
3. Todo o tráfego restante (0/0) com um Próximo Salto definido como o firewall.

Depois que as tabelas de roteamento são criadas, elas são associadas às respectivas sub-redes. A tabela de roteamento da sub-rede de front-end, uma vez criada e associada à sub-rede, deverá ter essa aparência:

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.1.0/24}     VNETLocal                            Active 
		 {10.0.0.0/16}     VirtualAppliance 10.0.0.4            Active    
         {0.0.0.0/0}       VirtualAppliance 10.0.0.4            Active

>[AZURE.NOTE] Há algumas restrições ao usar o UDR com a Rota Expressa devido à complexidade do roteamento dinâmico usado no gateway virtual do Azure:
>
>- O UDR não deve ser aplicado à sub-rede de gateway à qual o gateway virtual do Azure vinculado à Rota Expressa está conectado.
> - O gateway virtual do Azure vinculado à Rota Expressa não pode ser o dispositivo NextHop para outras sub-redes associadas ao UDR.
>
>Exemplos de como habilitar a rede de perímetro com a rede site a site ou de Rota Expressa são mostrados nos exemplos 3 e 4.


#### Descrição de Encaminhamento IP
O Encaminhamento IP é um recurso complementar para o UDR. Essa é uma configuração em uma solução de virtualização que permite receber o tráfego endereçado não especificamente para a solução e, em seguida, encaminhar esse tráfego para seu destino final.

Por exemplo, se o tráfego de AppVM01 fizesse uma solicitação para o servidor DNS01, o UDR rotearia isso para o firewall. Com o Encaminhamento IP habilitado, o tráfego para o destino DNS01 (10.0.2.4) será aceito pelo dispositivo (10.0.0.4) e então encaminhado para seu destino final (10.0.2.4). Sem o Encaminhamento IP habilitado no firewall, o tráfego não seria aceito pela solução, embora a tabela de rota tenha o firewall como o próximo salto. Para usar uma solução de virtualização, é essencial lembrar-se de habilitar o Encaminhamento IP em conjunto com o UDR.

#### Descrição de NSG
Neste exemplo, um grupo NSG é criado e então carregado com uma única regra. Esse grupo é então associado somente às sub-redes de front-end e back-end (e não a SecNet). Declarativamente, a seguinte regra está sendo criada:

- Todo o tráfego (todas as portas) da Internet para a rede virtual inteira (todas as sub-redes) é negado.

Embora NSGs sejam usados neste exemplo, seu principal objetivo será ser como uma segunda camada de defesa contra erros de configuração manual. A meta é bloquear todo o tráfego de entrada da Internet para as sub-redes de front-end ou back-end. O tráfego deve fluir somente pela sub-rede SecNet para o firewall (e, se apropriado, continuar para as sub-redes de front-end ou back-end). Além disso, com as regras UDR em vigor, qualquer tráfego que tenha conseguido passar para as sub-redes de front-end ou de back-end seria direcionado para o firewall (graças ao UDR). O firewall veria isso como um fluxo assimétrico e descartaria o tráfego de saída. Assim, há três camadas de segurança protegendo as sub-redes:
- Nenhum ponto de extremidade aberto nos serviços de nuvem FrontEnd001 e BackEnd001.
- NSGs negando o tráfego da Internet.
- O firewall soltando tráfego assimétrico.

Um ponto interessante sobre o NSG neste exemplo é que ele contém apenas uma regra, que é negar o tráfego da Internet para toda a rede virtual, incluindo a sub-rede de Segurança. No entanto, como o NSG está associado apenas às sub-redes de front-end e de back-end, a regra não é processada em tráfego de entrada para a sub-rede de Segurança. Como resultado, o tráfego fluirá para a sub-rede de Segurança.

#### Regras de firewall
No firewall, será necessário criar regras de encaminhamento. Uma vez que o firewall está bloqueando ou encaminhando todos os tráfegos de entrada, de saída e entre redes virtuais, serão necessárias muitas regras de firewall. Além disso, todo o tráfego de entrada atingirá o endereço IP público do Serviço de Segurança (em portas diferentes), para ser processado pelo firewall. Uma prática recomendada é criar um diagrama dos fluxos lógicos antes de configurar as sub-redes e as regras de firewall, para evitar uma reformulação posterior. A figura a seguir é uma exibição lógica das regras de firewall para este exemplo:
 
![Exibição lógica das regras de firewall][10]

>[AZURE.NOTE] Com base no Dispositivo Virtual de Rede usado, as portas de gerenciamento variam. Neste exemplo, um Firewall NextGen Barracuda é mencionado e usa as portas 22, 801 e 807. Consulte a documentação do fornecedor do dispositivo para localizar as portas exatas usadas para o gerenciamento do dispositivo usado.

#### Descrição das regras de firewall
No diagrama lógico anterior, a sub-rede de segurança não é mostrada. Isso ocorre porque o firewall é o único recurso nessa sub-rede, e esse diagrama mostra as regras de firewall e como elas permitem ou negam logicamente fluxos de tráfego e não o caminho roteado real. Além disso, as portas externas selecionadas para o tráfego RDP são portas com um intervalo maior (8014 – 8026) e foram selecionadas para alinhar um pouco com os dois últimos octetos do endereço IP local para facilitar a leitura (por exemplo, o endereço do servidor local 10.0.1.4 está associado à porta externa 8014). No entanto, todas as portas não conflitantes acima disso poderiam ser usadas.

Para este exemplo, precisamos de sete tipos de regras:

- Regras externas (para o tráfego de entrada):
  1.	Regra de gerenciamento de firewall: essa regra de Redirecionamento de Aplicativo permite que o tráfego passe para as portas de gerenciamento da solução de virtualização de rede.
  2.	Regras de RDP (para cada servidor Windows): essas quatro regras (uma para cada servidor) permitem o gerenciamento dos servidores individuais via RDP. Isso também poderia ser empacotado em uma regra, dependendo dos recursos da solução de virtualização de rede usada.
  3.	Regras de tráfego do aplicativo: há duas regras desse tipo, a primeira para o tráfego da Web de front-end e a segunda para o tráfego de back-end (por exemplo, o servidor Web para a camada de dados). A configuração dessas regras depende da arquitetura de rede (na qual os servidores são colocados) e dos fluxos de tráfego (a direção na qual o tráfego flui e quais portas são usadas).
      - A primeira regra permite que o tráfego de aplicativo real alcance o servidor de aplicativos. Embora as outras regras permitam segurança e gerenciamento, as regras de aplicativo são as que permitem que usuários ou serviços externos acessem o(s) aplicativo(s). Neste exemplo, há um único servidor Web na porta 80. Assim, uma única regra de aplicativo de firewall redireciona o tráfego de entrada para o IP externo, para o endereço IP interno dos servidores Web. A sessão de tráfego redirecionado seria movida por meio de NAT para o servidor interno.
      - A segunda regra de tráfego é a regra de back-end que permite que o servidor Web converse com o servidor AppVM01 (mas não com AppVM02) por meio de qualquer porta.
- Regras internas (para tráfego entre redes virtuais)
  4.	Regra de saída para Internet: essa regra permite que o tráfego de qualquer rede passe para as redes selecionadas. Normalmente, essa regra é uma regra padrão que já existe no firewall, mas em um estado desabilitado. Essa regra deve ser habilitada para esse exemplo.
  5.	Regra DNS: essa regra permite que somente o tráfego DNS (porta 53) passe para o servidor DNS. Para esse ambiente, a maior parte do tráfego do front-end para o back-end está bloqueado. Essa regra permite especificamente DNS de qualquer sub-rede local.
  6.	Regra sub-rede para sub-rede: essa regra permite que qualquer servidor na rede de back-end se conecte a qualquer servidor na rede de front-end (mas não o contrário).
- Regra à prova de falhas (para tráfego que não atenda a nenhum dos itens acima):
  7.	Regra negar todo o tráfego: essa sempre deverá ser a última regra (em termos de prioridade) e, como tal, se um fluxo de tráfego falhar na correspondência a todas as regras anteriores, ele será removido por essa regra. Essa é uma regra padrão e geralmente ativada. Geralmente, nenhuma modificação é necessária.

>[AZURE.TIP] Na segunda regra de tráfego de aplicativo, para simplificar este exemplo, qualquer porta é permitida. Em um cenário real, a porta mais específica e os intervalos de endereços devem ser usados para reduzir a superfície de ataque desta regra.

Depois que todas as regras anteriores forem criadas, será importante examinar a prioridade de cada regra para garantir que o tráfego seja permitido ou negado como desejado. Neste exemplo, as regras estão em ordem de prioridade.

#### Conclusão
Essa é uma maneira mais complexa, porém mais completa de proteger e isolar a rede do que nos exemplos anteriores. (o Exemplo 2 protege apenas o aplicativo e o Exemplo 1 isola apenas sub-redes). Esse design permite monitorar o tráfego em ambos os trajetos e protege não apenas o servidor de aplicativos de entrada, mas impõe políticas de segurança de rede para todos os servidores nessa rede. Além disso, dependendo do dispositivo usado, pode-se conseguir auditoria e reconhecimento total de tráfego. Para obter mais informações, consulte as [instruções detalhadas de build][Example3]. Essas instruções incluem:

- Como criar essa rede de perímetro de exemplo com scripts do PowerShell.
- Como criar esse exemplo com um modelo do Azure Resource Manager.
- As descrições detalhadas de cada comando NSG, UDR e regra de firewall.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego é permitido ou negado em cada camada.

### Exemplo 4: adicionar uma conexão híbrida com uma VPN (rede virtual privada) site a site de solução de virtualização
[Voltar ao Início rápido](#fast-start) | Instruções detalhadas de build estarão disponíveis em breve

![Rede de perímetro com rede híbrida conectada com NVA][11]

#### Descrição do ambiente
Uma rede híbrida usando um NVA (solução de virtualização de rede) pode ser adicionada a qualquer um dos tipos de rede de perímetro descritas no exemplo 1, 2 ou 3.

Conforme mostrado na figura anterior, uma conexão VPN pela Internet (site a site) é usada para conectar uma rede local a uma rede virtual do Azure através de um NVA.

>[AZURE.NOTE] Se você usar a Rota Expressa com a opção de Emparelhamento Público do Azure habilitada, uma rota estática deverá ser criada. Isso deve rotear para o endereço IP de VPN de NVA fora de sua Internet corporativa, e não através de WAN de Rota Expressa. A NAT necessária na opção de Emparelhamento Público de Rota Expressa do Azure pode interromper a sessão de VPN.

Depois que a VPN estiver no local, o NVA torna-se o hub central para todas as redes e sub-redes. As regras de encaminhamento de firewall determinam quais fluxos de tráfego são permitidos, convertidos via NAT, redirecionados ou removidos (mesmo para fluxos de tráfego entre a rede local e o Azure).

Os fluxos de tráfego devem ser considerados com cuidado, já que podem ser otimizados ou degradados por esse padrão de design dependendo do caso de uso específico.

Usar o ambiente criado no exemplo 3 e então adicionar uma conexão de rede híbrida VPN site a site gera o design a seguir:

![Rede de perímetro com NVA conectada usando uma VPN site a site][12]

O roteador local, ou outro dispositivo de rede que seja compatível com seu NVA para VPN, seria o cliente VPN. Este dispositivo físico seria responsável por iniciar e manter a conexão VPN com seu NVA.

Logicamente para o NVA, a rede é igual a quatro "zonas de segurança" separadas com as regras sobre o NVA sendo as principais orientadoras do tráfego entre essas zonas:

![Rede lógica da perspectiva de NVA][13]

#### Conclusão
A adição de uma conexão de rede híbrida VPN site a site para uma rede virtual do Azure pode estender a rede local no Azure de maneira segura. Ao usar uma conexão VPN, o tráfego é criptografado e roteado pela Internet. O NVA neste exemplo fornece um local central para aplicar e gerenciar a política de segurança. Para obter mais informações, consulte as instruções detalhadas de build (próximo). Essas instruções incluem:

- Como criar essa rede de perímetro de exemplo com scripts do PowerShell.
- Como criar esse exemplo com um modelo do Azure Resource Manager.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego flui através desse design.

### Exemplo 5: adicionar uma conexão híbrida com uma VPN site a site de gateway do Azure
[Voltar ao Início rápido](#fast-start) | Instruções detalhadas de build estarão disponíveis em breve

![Rede de perímetro com rede híbrida conectada com gateway][14]

#### Descrição do ambiente
Uma rede híbrida usando um gateway de VPN do Azure pode ser adicionada a qualquer tipo de rede de perímetro descrito nos exemplos 1 ou 2.

Conforme mostrado na figura anterior, uma conexão VPN pela Internet (site a site) é usada para conectar uma rede local a uma rede virtual do Azure através de um gateway de VPN do Azure.

>[AZURE.NOTE] Se você usar a Rota Expressa com a opção de Emparelhamento Público do Azure habilitada, uma rota estática deverá ser criada. Isso deve rotear para o endereço IP de VPN de NVA fora de sua Internet corporativa, e não através de WAN de Rota Expressa. A NAT necessária na opção de Emparelhamento Público de Rota Expressa do Azure pode interromper a sessão de VPN.

A figura a seguir mostra as duas bordas de rede nessa opção. Na primeira borda, o NVA e os NSGs controlam os fluxos de tráfego entre redes do Azure e entre o Azure e a Internet. A segunda borda é o gateway de VPN do Azure, que é uma borda de rede completamente separada e isolada entre local e o Azure.

Os fluxos de tráfego devem ser considerados com cuidado, já que podem ser otimizados ou degradados por esse padrão de design dependendo do caso de uso específico.

Usar o ambiente criado no exemplo 1 e então adicionar uma conexão de rede híbrida VPN site a site gera o design a seguir:

![Rede de perímetro com gateway conectada usando uma conexão de Rota Expressa][15]

#### Conclusão
A adição de uma conexão de rede híbrida VPN site a site para uma rede virtual do Azure pode estender a rede local no Azure de maneira segura. Usando o gateway de VPN do Azure nativo, o tráfego é criptografado com IPsec e roteado via Internet. Além disso, o uso de gateway de VPN do Azure pode oferecer uma opção de menor custo (sem custos com licença adicional ou NVAs de terceiros). Isso é mais econômico no exemplo 1, em que nenhum NVA é usado. Para obter mais informações, consulte as instruções detalhadas de build (próximo). Essas instruções incluem:

- Como criar essa rede de perímetro de exemplo com scripts do PowerShell.
- Como criar esse exemplo com um modelo do Azure Resource Manager.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego flui através desse design.

### Exemplo 6: adicione uma conexão híbrida com a Rota Expressa
[Voltar ao Início rápido](#fast-start) | Instruções detalhadas de build estarão disponíveis em breve

![Rede de perímetro com rede híbrida conectada com gateway][16]

#### Descrição do ambiente
Uma rede híbrida usando uma conexão de emparelhamento privado de Rota Expressa pode ser adicionada a qualquer tipo de rede de perímetro descrita nos exemplos 1 ou 2.

Conforme mostrado na figura anterior, o emparelhamento privado de Rota Expressa fornece uma conexão direta entre sua rede local e de rede virtual do Azure. O tráfego transmite apenas a rede do provedor de serviços e a rede do Microsoft Azure, nunca em contato com a Internet.

>[AZURE.NOTE] Há algumas restrições ao usar o UDR com a Rota Expressa devido à complexidade do roteamento dinâmico usado no gateway virtual do Azure. Elas são as seguintes:
>
>- O UDR não deve ser aplicado à sub-rede de gateway à qual o gateway virtual do Azure vinculado à Rota Expressa está conectado.
>- O gateway virtual do Azure vinculado à Rota Expressa não pode ser o dispositivo NextHop para outras sub-redes associadas ao UDR.
>
>
<br />

>[AZURE.TIP] O uso da Rota Expressa mantém o tráfego de rede corporativa fora da Internet para melhorar a segurança e aumentar significativamente o desempenho. Ele também permite SLAs do seu provedor da Rota Expressa. O Gateway do Azure pode passar até 2 Gb/s com a Rota Expressa, enquanto com VPNs site a site, a taxa de transferência máxima do Gateway do Azure é de 200 Mbps.

Conforme mostrado no diagrama abaixo, com essa opção, o ambiente agora tem duas bordas de rede. O NVA e NSG controlam fluxos de tráfego para redes internas do Azure e entre o Azure e a Internet, enquanto o gateway é uma borda de rede completamente separada e isolada entre locais e o Azure.

Os fluxos de tráfego devem ser considerados com cuidado, já que podem ser otimizados ou degradados por esse padrão de design dependendo do caso de uso específico.

Usar o ambiente criado no exemplo 1 e então adicionar uma conexão de rede híbrida de Rota Expressa gera o design a seguir:

![Rede de perímetro com gateway conectada usando uma conexão de Rota Expressa][17]

#### Conclusão
A adição de uma conexão de rede de emparelhamento privado de Rota Expressa pode estender a rede local para o Azure de forma segura, com baixa latência e alto desempenho. Além disso, o uso de Gateway do Azure nativo, como visto neste exemplo, fornece uma opção de menor custo (sem licença adicional ou NVAs de terceiros). Para obter mais informações, consulte as instruções detalhadas de build (próximo). Essas instruções incluem:

- Como criar essa rede de perímetro de exemplo com scripts do PowerShell.
- Como criar esse exemplo com um modelo do Azure Resource Manager.
- Cenários de fluxo de tráfego detalhados mostrando como o tráfego flui através desse design.

## Referências
### Sites úteis e documentação
- Acesse o Azure com o Azure Resource Manager:
- Acessando o Azure com o PowerShell: [https://azure.microsoft.com/documentation/articles/powershell-install-configure/](./powershell-install-configure.md)
- Documentação de rede virtual: [https://azure.microsoft.com/documentation/services/virtual-network/](https://azure.microsoft.com/documentation/services/virtual-network/)
- Documentação do grupo de segurança de rede: [https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/](./virtual-network/virtual-networks-nsg.md)
- Documentação do roteamento definido pelo usuário: [https://azure.microsoft.com/documentation/articles/virtual-networks-udr-overview/](./virtual-network/virtual-networks-udr-overview.md)
- Gateways virtuais do Azure: [https://azure.microsoft.com/documentation/services/vpn-gateway/](https://azure.microsoft.com/documentation/services/vpn-gateway/)
- VPNs Site a Site: [https://azure.microsoft.com/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell](./vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
- Documentação da Rota Expressa (confira as seções "Introdução" e "Tutoriais"): [https://azure.microsoft.com/documentation/services/expressroute/](https://azure.microsoft.com/documentation/services/expressroute/)

<!--Image References-->
[0]: ./media/best-practices-network-security/flowchart.png "Fluxograma de opções de segurança"
[1]: ./media/best-practices-network-security/compliancebadges.png "Selos de conformidade do Azure"
[2]: ./media/best-practices-network-security/azuresecurityfeatures.png "Recursos de segurança do Azure"
[3]: ./media/best-practices-network-security/dmzcorporate.png "Uma DMZ em uma rede corporativa"
[4]: ./media/best-practices-network-security/azuresecurityarchitecture.png "Arquitetura de segurança do Azure"
[5]: ./media/best-practices-network-security/dmzazure.png "Uma DMZ em uma Rede Virtual do Azure"
[6]: ./media/best-practices-network-security/dmzhybrid.png "Rede híbrida com três limites de segurança"
[7]: ./media/best-practices-network-security/example1design.png "DMZ de entrada com NSG"
[8]: ./media/best-practices-network-security/example2design.png "DMZ de entrada com NVA e NSG"
[9]: ./media/best-practices-network-security/example3design.png "DMZ bidirecional com NVA, NSG e UDR"
[10]: ./media/best-practices-network-security/example3firewalllogical.png "Exibição lógica das regras de firewall"
[11]: ./media/best-practices-network-security/example4designoptions.png "DMZ com rede híbrida conectada com NVA"
[12]: ./media/best-practices-network-security/example4designs2s.png "DMZ com NVA conectado usando uma VPN site a site"
[13]: ./media/best-practices-network-security/example4networklogical.png "Rede lógica da perspectiva de NVA"
[14]: ./media/best-practices-network-security/example5designoptions.png "DMZ com rede híbrida site a site conectada ao Gateway do Azure"
[15]: ./media/best-practices-network-security/example5designs2s.png "DMZ com Gateway do Azure usando VPN site a site"
[16]: ./media/best-practices-network-security/example6designoptions.png "DMZ com rede híbrida da Rota Expressa conectada ao Gateway do Azure"
[17]: ./media/best-practices-network-security/example6designexpressroute.png "DMZ com Gateway do Azure usando uma conexão de Rota Expressa"

<!--Link References-->
[Example1]: ./virtual-network/virtual-networks-dmz-nsg-asm.md
[Example2]: ./virtual-network/virtual-networks-dmz-nsg-fw-asm.md
[Example3]: ./virtual-network/virtual-networks-dmz-nsg-fw-udr-asm.md
[Example4]: ./virtual-network/virtual-networks-hybrid-s2s-nva-asm.md
[Example5]: ./virtual-network/virtual-networks-hybrid-s2s-agw-asm.md
[Example6]: ./virtual-network/virtual-networks-hybrid-expressroute-asm.md
[Example7]: ./virtual-network/virtual-networks-vnet2vnet-direct-asm.md
[Example8]: ./virtual-network/virtual-networks-vnet2vnet-transit-asm.md

<!-----------HONumber=AcomDC_0330_2016-->