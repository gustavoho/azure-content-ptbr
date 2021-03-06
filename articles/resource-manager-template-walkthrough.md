<properties
   pageTitle="Passo a Passo do Modelo do Resource Manager | Microsoft Azure"
   description="Uma explicação passo a passo de um modelo do gerenciador de recursos fornecendo uma arquitetura básica do Azure IaaS."
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/04/2016"
   ms.author="navale;tomfitz"/>
   
# Passo a passo do modelo do Gerenciador de Recursos

Uma das primeiras perguntas ao criar um modelo é: "como começar?" É possível iniciar a partir de um modelo em branco seguindo a estrutura básica descrita no [Artigo Modelo de Criação](resource-group-authoring-templates.md#template-format) e adicionar os recursos, parâmetros e variáveis apropriados. Seria uma boa alternativa iniciar por meio da [galeria de início rápido](https://github.com/Azure/azure-quickstart-templates) e procurar cenários parecidos com aquele que você está tentando criar. Você pode mesclar vários modelos ou editar um já existente de acordo com seu cenário específico.

Vejamos uma infraestrutura comum:

* Duas máquinas virtuais que usam a mesma conta de armazenamento estão no mesmo conjunto de disponibilidade e na mesma sub-rede de uma rede virtual.
* Um único endereço IP da VM e NIC para cada máquina virtual.
* Um balanceador de carga com uma regra de balanceamento de carga na porta 80

![Arquitetura](./media/resource-group-overview/arm_arch.png)

Este tópico explica as etapas de criação de um modelo do Gerenciador de Recursos para essa infraestrutura. O modelo final que você cria é baseado em um modelo de Início Rápido denominado [2 VMs em um Balanceador de Carga e regras de balanceamento de carga](https://azure.microsoft.com/documentation/templates/201-2-vms-loadbalancer-lbrules/).

Mas, é muita coisa para compilar de uma só vez. Portanto, primeiro criaremos uma conta de armazenamento e iremos implantá-la. Depois de ter dominado a criação da conta de armazenamento, você adicionará outros recursos e reimplantará o modelo para concluir a infraestrutura.

>[AZURE.NOTE] Ao criar o modelo, você pode usar qualquer tipo de editor. O Visual Studio fornece ferramentas que simplificam o desenvolvimento do modelo, mas não é necessário o Visual Studio para concluir este tutorial. Para obter um tutorial sobre como usar o Visual Studio para criar uma implantação do Aplicativo Web e do Banco de Dados SQL, consulte [Criação e implantação dos grupos de recursos do Azure com o Visual Studio](vs-azure-tools-resource-groups-deployment-projects-create-deploy.md).

## Criar o modelo do Gerenciador de Recursos

O modelo é um arquivo JSON que define todos os recursos que você irá implantar. Ele também permite definir os parâmetros que são especificados durante a implantação, as variáveis construídas a partir de outros valores e expressões, e as saídas da implantação.

Comecemos com o modelo mais simples:

```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {  },
      "variables": {  },
      "resources": [  ],
      "outputs": {  }
    }
 ```

Salve esse arquivo como **azuredeploy.json** (observe que o modelo pode ter qualquer nome desejado; ele só deve ser um arquivo json).

## Criar uma conta de armazenamento
Na seção **recursos**, adicione um objeto que define a conta de armazenamento, conforme mostrado abaixo.

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[parameters('storageAccountName')]",
    "apiVersion": "2015-06-15",
    "location": "[resourceGroup().location]",
    "properties": {
      "accountType": "Standard_LRS"
    }
  }
]
```

Você pode estar imaginando de onde vêm essas propriedades e valores. As propriedades **type**, **name**, **apiVersion** e **location** são elementos padrão disponíveis para todos os tipos de recursos. Você pode aprender sobre os elementos comuns em [Recursos](resource-group-authoring-templates.md#resources). **name** é definido para um valor do parâmetro que você passa durante a implantação e **location**, como o local usado pelo grupo de recursos. Veremos como determinar **type** e **apiVersion** nas seções a seguir.

A seção **propriedades** contém todas as propriedades exclusivas para um tipo de recurso específico. Os valores especificados nesta seção correspondem exatamente à operação PUT na API REST para criar esse tipo de recurso. Ao criar uma conta de armazenamento, você deve fornecer um **accountType**. Observe na [API REST para criar uma Conta de armazenamento](https://msdn.microsoft.com/library/azure/mt163564.aspx) que a seção de propriedades da operação REST também contém uma propriedade **accountType** e os valores permitidos são documentados. Neste exemplo, o tipo de conta é definido para **Standard\_LRS**, mas você pode especificar outro valor ou permitir que os usuários passem o tipo de conta como um parâmetro.

Agora, vamos pular para a seção **parâmetros** e ver como você define o nome da conta de armazenamento. Você pode saber mais sobre o uso dos parâmetros em [Parâmetros](resource-group-authoring-templates.md#parameters).

```json
"parameters" : {
	"storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Storage Account Name"
      }
    }
}
```
Aqui, você definiu um parâmetro do tipo de cadeia de caracteres que conterá o nome da conta de armazenamento. O valor desse parâmetro será fornecido durante a implantação do modelo.

## Implantação do modelo
Temos um modelo completo para criar uma nova conta de armazenamento. Como você se lembra, o modelo foi salvo no arquivo **azuredeploy.json**:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters" : {
	"storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Storage Account Name"
      }
    }
  },  
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[parameters('storageAccountName')]",
      "apiVersion": "2015-06-15",
      "location": "[resourceGroup().location]",
      "properties": {
        "accountType": "Standard_LRS"
      }
    }
  ]
}
```

Há algumas maneiras de implantar um modelo, como você pode ver no [Artigo de Implantação de Recursos](resource-group-template-deploy.md). Para implantar o modelo usando o Azure PowerShell, use:

```powershell
# create a new resource group
New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "West Europe"

# deploy the template to the resource group
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile azuredeploy.json
```

Ou para implantar o modelo usando a CLI do Azure, use:

```
azure group create -n ExampleResourceGroup -l "West Europe"

azure group deployment create -f azuredeploy.json -g ExampleResourceGroup -n ExampleDeployment
```

Agora, você é o orgulhoso proprietário de uma conta de armazenamento!

As próximas etapas serão para adicionar todos os recursos necessários para implantar a arquitetura descrita no início deste tutorial. Você adicionará esses recursos no mesmo modelo no qual estava trabalhando.

## Conjunto de disponibilidade
Após a definição da conta de armazenamento, adicione um conjunto disponível para as máquinas virtuais. Nesse caso, não há nenhuma propriedade adicional necessária, portanto, sua definição é bastante simples. Consulte a [API REST para criar um Conjunto de Disponibilidade](https://msdn.microsoft.com/library/azure/mt163607.aspx) para ver a seção de propriedades completa, caso queira definir os valores de contagem do domínio de atualização e do domínio de falha.

```json
{
   "type": "Microsoft.Compute/availabilitySets",
   "name": "[variables('availabilitySetName')]",
   "apiVersion": "2015-06-15",
   "location": "[resourceGroup().location]",
   "properties": {}
}
```

Observe que **name** é definido para o valor de uma variável. Para esse modelo, o nome do conjunto de disponibilidade é necessário em alguns lugares diferentes. Você pode manter mais facilmente seu modelo definindo esse valor uma vez e usando-o em vários lugares.

O valor especificado para **type** contém o provedor de recursos e o tipo de recurso. Para os conjuntos de disponibilidade, o provedor de recursos é **Microsoft.Compute** e o tipo de recurso é **availabilitySets**. Você pode obter a lista de provedores de recursos disponíveis executando o seguinte comando do PowerShell:

```powershell
    Get-AzureRmResourceProvider -ListAvailable
```

Ou se estiver usando a CLI do Azure, você poderá executar o comando a seguir:
```
    azure provider list
```
Considerando que, neste tópico, você está criando com contas de armazenamento, máquinas virtuais e redes virtuais, você trabalhará com o:

- Microsoft.Storage
- Microsoft.Compute
- Microsoft.Network

Para ver os tipos de recursos para um provedor específico, execute o seguinte comando do PowerShell:

```powershell
    (Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute).ResourceTypes
```

Ou, para a CLI do Azure, o comando a seguir retornará os tipos disponíveis no formato JSON; salve-o em um arquivo.

```
    azure provider show Microsoft.Compute --json > c:\temp.json
```

Você deverá ver **availabilitySets** como um dos tipos em **Microsoft.Compute**. O nome completo do tipo é **Microsoft.Compute/availabilitySets**. Você pode determinar o nome do tipo de recurso para qualquer um dos recursos no modelo.

## IP público
Defina um endereço IP público. Novamente, veja a [API REST para obter os endereços IP públicos](https://msdn.microsoft.com/library/azure/mt163590.aspx) para definir as propriedades.

```json
{
  "apiVersion": "2015-06-15",
  "type": "Microsoft.Network/publicIPAddresses",
  "name": "[parameters('publicIPAddressName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "publicIPAllocationMethod": "Dynamic",
    "dnsSettings": {
      "domainNameLabel": "[parameters('dnsNameforLBIP')]"
    }
  }
}
```

O método de alocação é definido para **Dinâmico**, mas você pode defini-lo para o valor necessário ou defini-lo para aceitar um valor de parâmetro. Você habilitou os usuários do seu modelo a passarem um valor para o rótulo de nome do domínio.

Agora, vejamos como determinar a **apiVersion**. O valor especificado simplesmente corresponde à versão da API REST que você deseja usar ao criar o recurso. Portanto, você pode examinar a documentação da API REST para esse tipo de recurso. Ou pode executar o seguinte comando do PowerShell para um tipo específico.

```powershell
    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Network).ResourceTypes | Where-Object ResourceTypeName -eq publicIPAddresses).ApiVersions
```
Isso retorna os seguintes valores:

    2015-06-15
    2015-05-01-preview
    2014-12-01-preview

Para ver as versões da API com a CLI do Azure, execute o mesmo comando **azure provider show** mostrado anteriormente.

Ao criar um novo modelo, selecione a versão mais recente de API.

## Rede virtual e sub-rede
Crie uma rede virtual com uma sub-rede. Veja a [API REST para as redes virtuais](https://msdn.microsoft.com/library/azure/mt163661.aspx) para obter todas as propriedades a definir.

```json
{
   "apiVersion": "2015-06-15",
   "type": "Microsoft.Network/virtualNetworks",
   "name": "[parameters('vnetName')]",
   "location": "[resourceGroup().location]",
   "properties": {
     "addressSpace": {
       "addressPrefixes": [
         "10.0.0.0/16"
       ]
     },
     "subnets": [
       {
         "name": "[variables('subnetName')]",
         "properties": {
           "addressPrefix": "10.0.0.0/24"
         }
       }
     ]
   }
}
```

## Balanceador de carga
Agora, você criará um balanceador de carga externo. Como esse balanceador de carga usa o endereço IP público, você deve declarar uma dependência no endereço IP público na seção **dependsOn**. Isso significa que o balanceador de carga não será implantado até que o endereço IP público termine de implantar. Sem definir essa dependência, você receberá um erro porque o Gerenciador de Recursos tentará implantar os recursos paralelamente e tentará definir o balanceador de carga para um endereço IP público que ainda não existe.

Você também criará um pool de endereços de back-end, algumas regras NAT de entrada para o RDP nas VMs e uma regra de balanceamento de carga com uma investigação tcp na porta 80 nessa definição de recurso. Verifique a [API REST do balanceador de carga](https://msdn.microsoft.com/library/azure/mt163574.aspx) para ver todas as propriedades.

```json
{
   "apiVersion": "2015-06-15",
   "name": "[parameters('lbName')]",
   "type": "Microsoft.Network/loadBalancers",
   "location": "[resourceGroup().location]",
   "dependsOn": [
     "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]"
   ],
   "properties": {
     "frontendIPConfigurations": [
       {
         "name": "LoadBalancerFrontEnd",
         "properties": {
           "publicIPAddress": {
             "id": "[variables('publicIPAddressID')]"
           }
         }
       }
     ],
     "backendAddressPools": [
       {
         "name": "BackendPool1"
       }
     ],
     "inboundNatRules": [
       {
         "name": "RDP-VM0",
         "properties": {
           "frontendIPConfiguration": {
             "id": "[variables('frontEndIPConfigID')]"
           },
           "protocol": "tcp",
           "frontendPort": 50001,
           "backendPort": 3389,
           "enableFloatingIP": false
         }
       },
       {
         "name": "RDP-VM1",
         "properties": {
           "frontendIPConfiguration": {
             "id": "[variables('frontEndIPConfigID')]"
           },
           "protocol": "tcp",
           "frontendPort": 50002,
           "backendPort": 3389,
           "enableFloatingIP": false
         }
       }
     ],
     "loadBalancingRules": [
       {
         "name": "LBRule",
         "properties": {
           "frontendIPConfiguration": {
             "id": "[variables('frontEndIPConfigID')]"
           },
           "backendAddressPool": {
             "id": "[variables('lbPoolID')]"
           },
           "protocol": "tcp",
           "frontendPort": 80,
           "backendPort": 80,
           "enableFloatingIP": false,
           "idleTimeoutInMinutes": 5,
           "probe": {
             "id": "[variables('lbProbeID')]"
           }
         }
       }
     ],
     "probes": [
       {
         "name": "tcpProbe",
         "properties": {
           "protocol": "tcp",
           "port": 80,
           "intervalInSeconds": 5,
           "numberOfProbes": 2
         }
       }
     ]
   }
}
```

## Interface de rede
Você criará duas interfaces de rede, uma para cada VM. Em vez de precisar incluir entradas duplicadas para as interfaces de rede, você poderá usar a [função copyindex()](resource-group-create-multiple.md) para iterar o loop de cópia (referido como nicLoop) e criar várias interfaces de rede, conforme definido nas variáveis `numberOfInstances`. A interface de rede depende da criação da rede virtual e do balanceador de carga. Ela usa a sub-rede definida na criação da rede virtual e a ID do balanceador de carga para configurar o pool de endereços do balanceador de carga e as regras NAT de entrada. Veja a [API REST para as interfaces de rede](https://msdn.microsoft.com/library/azure/mt163668.aspx) para obter todas as propriedades.

```json
{
   "apiVersion": "2015-06-15",
   "type": "Microsoft.Network/networkInterfaces",
   "name": "[concat(parameters('nicNamePrefix'), copyindex())]",
   "location": "[resourceGroup().location]",
   "copy": {
     "name": "nicLoop",
     "count": "[variables('numberOfInstances')]"
   },
   "dependsOn": [
     "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]",
     "[concat('Microsoft.Network/loadBalancers/', parameters('lbName'))]"
   ],
   "properties": {
     "ipConfigurations": [
       {
         "name": "ipconfig1",
         "properties": {
           "privateIPAllocationMethod": "Dynamic",
           "subnet": {
             "id": "[variables('subnetRef')]"
           },
           "loadBalancerBackendAddressPools": [
             {
               "id": "[concat(variables('lbID'), '/backendAddressPools/BackendPool1')]"
             }
           ],
           "loadBalancerInboundNatRules": [
             {
               "id": "[concat(variables('lbID'),'/inboundNatRules/RDP-VM', copyindex())]"
             }
           ]
         }
       }
     ]
   }
}
```

## Máquina virtual
Você criará duas máquinas virtuais usando a função copyIndex(), como fez na criação das [interfaces de rede](#network-interface). A criação da VM depende da conta de armazenamento, da interface de rede e do conjunto de disponibilidade. Essa VM será criada a partir de uma imagem do marketplace, conforme definido na propriedade `storageProfile` - `imageReference` é usada para definir o editor da imagem, oferta, sku e versão. Por fim, um perfil de diagnóstico é configurado para habilitar o diagnóstico para a VM.

Para localizar as propriedades relevantes para uma imagem do marketplace, siga os artigos [Selecionar imagens da máquina virtual do Linux](./virtual-machines/virtual-machines-linux-cli-ps-findimage.md) ou [Selecionar imagens da máquina virtual do Windows](./virtual-machines/virtual-machines-windows-cli-ps-findimage.md).

```json
{
   "apiVersion": "2015-06-15",
   "type": "Microsoft.Compute/virtualMachines",
   "name": "[concat(parameters('vmNamePrefix'), copyindex())]",
   "copy": {
     "name": "virtualMachineLoop",
     "count": "[variables('numberOfInstances')]"
   },
   "location": "[resourceGroup().location]",
   "dependsOn": [
     "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]",
     "[concat('Microsoft.Network/networkInterfaces/', parameters('nicNamePrefix'), copyindex())]",
     "[concat('Microsoft.Compute/availabilitySets/', variables('availabilitySetName'))]"
   ],
   "properties": {
     "availabilitySet": {
       "id": "[resourceId('Microsoft.Compute/availabilitySets',variables('availabilitySetName'))]"
     },
     "hardwareProfile": {
       "vmSize": "[parameters('vmSize')]"
     },
     "osProfile": {
       "computerName": "[concat(parameters('vmNamePrefix'), copyIndex())]",
       "adminUsername": "[parameters('adminUsername')]",
       "adminPassword": "[parameters('adminPassword')]"
     },
     "storageProfile": {
       "imageReference": {
         "publisher": "[parameters('imagePublisher')]",
         "offer": "[parameters('imageOffer')]",
         "sku": "[parameters('imageSKU')]",
         "version": "latest"
       },
       "osDisk": {
         "name": "osdisk",
         "vhd": {
           "uri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/vhds/','osdisk', copyindex(), '.vhd')]"
         },
         "caching": "ReadWrite",
         "createOption": "FromImage"
       }
     },
     "networkProfile": {
       "networkInterfaces": [
         {
           "id": "[resourceId('Microsoft.Network/networkInterfaces',concat(parameters('nicNamePrefix'),copyindex()))]"
         }
       ]
     },
     "diagnosticsProfile": {
       "bootDiagnostics": {
          "enabled": "true",
          "storageUri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net')]"
       }
     }
}
```

>[AZURE.NOTE] Para as imagens publicadas por **terceiros**, você precisará especificar outra propriedade denominada `plan`. Um exemplo pode ser encontrado [neste modelo](https://github.com/Azure/azure-quickstart-templates/tree/master/checkpoint-single-nic) da galeria de início rápido.

Você terminou de definir os recursos para o modelo.

## Parâmetros

Na seção de parâmetros, defina os valores que podem ser especificados ao implantar o modelo. Apenas defina os parâmetros para os valores que você acha que devem ser variados durante a implantação. Você pode fornecer um valor padrão para um parâmetro que será usado se um não for fornecido durante a implantação. Você também pode definir os valores permitidos conforme mostrado para o parâmetro **imageSKU**.

```json
"parameters": {
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of storage account"
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password"
      }
    },
    "dnsNameforLBIP": {
      "type": "string",
      "metadata": {
        "description": "DNS for Load Balancer IP"
      }
    },
    "vmNamePrefix": {
      "type": "string",
      "defaultValue": "myVM",
      "metadata": {
        "description": "Prefix to use for VM names"
      }
    },
    "imagePublisher": {
      "type": "string",
      "defaultValue": "MicrosoftWindowsServer",
      "metadata": {
        "description": "Image Publisher"
      }
    },
    "imageOffer": {
      "type": "string",
      "defaultValue": "WindowsServer",
      "metadata": {
        "description": "Image Offer"
      }
    },
    "imageSKU": {
      "type": "string",
      "defaultValue": "2012-R2-Datacenter",
      "allowedValues": [
        "2008-R2-SP1",
        "2012-Datacenter",
        "2012-R2-Datacenter"
      ],
      "metadata": {
        "description": "Image SKU"
      }
    },
    "lbName": {
      "type": "string",
      "defaultValue": "myLB",
      "metadata": {
        "description": "Load Balancer name"
      }
    },
    "nicNamePrefix": {
      "type": "string",
      "defaultValue": "nic",
      "metadata": {
        "description": "Network Interface name prefix"
      }
    },
    "publicIPAddressName": {
      "type": "string",
      "defaultValue": "myPublicIP",
      "metadata": {
        "description": "Public IP Name"
      }
    },
    "vnetName": {
      "type": "string",
      "defaultValue": "myVNET",
      "metadata": {
        "description": "VNET name"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D1",
      "metadata": {
        "description": "Size of the VM"
      }
    }
  }
```

## Variáveis

Na seção de variáveis, você pode definir os valores que são usados em mais de um lugar em seu modelo ou os valores que são construídos a partir de outras expressões ou variáveis. As variáveis são usadas com frequência para simplificar a sintaxe do modelo.

```json
"variables": {
    "availabilitySetName": "myAvSet",
    "subnetName": "Subnet-1",
    "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('vnetName'))]",
    "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables ('subnetName'))]",
    "publicIPAddressID": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]",
    "numberOfInstances": 2,
    "lbID": "[resourceId('Microsoft.Network/loadBalancers',parameters('lbName'))]",
    "frontEndIPConfigID": "[concat(variables('lbID'),'/frontendIPConfigurations/LoadBalancerFrontEnd')]",
    "lbPoolID": "[concat(variables('lbID'),'/backendAddressPools/BackendPool1')]",
    "lbProbeID": "[concat(variables('lbID'),'/probes/tcpProbe')]"
  }
```

Você concluiu o modelo! Você pode comparar o modelo em relação ao modelo completo na [galeria de início rápido](https://github.com/Azure/azure-quickstart-templates) no [modelo de 2 VMs com um balanceador de carga e regras de balanceamento de carga](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-lbrules). O modelo pode ser ligeiramente diferente com base no uso de números de versão diferentes.

Você pode reimplantar o modelo usando os mesmos comandos que usou ao implantar a conta de armazenamento. Você não precisa excluir a conta de armazenamento antes de reimplantar porque o Gerenciador de Recursos irá ignorar a recriação dos recursos que já existem e não foram alterados.

## Próximas etapas

- O [Visualizador do Modelo de Azure Resource Manager (ARMViz)](http://armviz.io/#/) é uma excelente ferramenta para visualizar os modelos do ARM, pois eles podem ficar muito grandes para compreender apenas lendo o arquivo json.
- Para saber mais sobre a estrutura de um modelo, consulte [Criando modelos do Azure Resource Manager](resource-group-authoring-templates.md).
- Para saber mais sobre como implantar um modelo, consulte [Implantar um Grupo de Recursos com o modelo do Azure Resource Manager](resource-group-template-deploy.md).

<!---HONumber=AcomDC_0511_2016-->