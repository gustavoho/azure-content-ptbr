<properties
	pageTitle="Preparando seu ambiente para fazer backup das máquinas virtuais implantadas com o Gerenciador de Recursos | Microsoft Azure"
	description="Verificar se o ambiente está preparado para fazer backup de máquinas virtuais no Azure"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="cfreeman"
	editor=""
	keywords="backups; fazendo backup;"/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/03/2016"
	ms.author="trinadhk; jimpark; markgal;"/>


# Preparar seu ambiente para fazer backup das máquinas virtuais implantadas com o Gerenciador de Recursos

> [AZURE.SELECTOR]
- [Modelo do gerenciador de recursos](backup-azure-arm-vms-prepare.md)
- [Modelo clássico](backup-azure-vms-prepare.md)

Este artigo fornece as etapas para preparar seu ambiente para fazer backup de uma VM (máquina virtual) implantada com o Gerenciador de Recursos. As etapas mostradas nos procedimentos usam o Portal do Azure.

O serviço de Backup do Azure tem dois tipos de cofres (cofres de backup e de serviços de recuperação) para proteger suas VMs. Um cofre de backup protege VMs implantadas com o modelo de implantação Clássico. Um cofre dos serviços de recuperação protege **tanto as VMs implantadas com o modelo de implantação Clássico quanto com o Gerenciador de Recursos**. Você deve usar um cofre dos Serviços de Recuperação para proteger uma VM implantada com o Gerenciador de Recursos.

>[AZURE.NOTE] O Azure tem dois modelos de implantação para criar e trabalhar com recursos: [Resource Manager e Clássico](../resource-manager-deployment-model.md). Consulte [Preparar seu ambiente para fazer backup de máquinas virtuais do Azure](backup-azure-vms-prepare.md) para obter detalhes sobre como trabalhar com VMs do modelo de implantação Clássico.

Antes de proteger ou fazer backup de uma VM (máquina virtual) implantada com o Gerenciador de Recursos, verifique se esses pré-requisitos foram cumpridos:

- Crie um cofre dos serviços de recuperação (ou identifique um cofre dos serviços de recuperação existente) *no mesmo local que sua VM*.
- Selecione um cenário, defina a política de backup e os itens a serem protegidos.
- Verifique a instalação do Agente de VM na máquina virtual.
- Verificar a conectividade de rede

Se você souber que essas condições já existem em seu ambiente, prossiga para o [artigo Fazer backup das suas VMs](backup-azure-vms.md). Se você precisa configurar ou verificar qualquer um desses pré-requisitos, este artigo orienta você sobre etapas para prepará-los.


## Limitações durante o backup e a restauração de uma VM

Antes de preparar seu ambiente, compreenda as limitações.

- Não há suporte para o backup de máquinas virtuais com mais de 16 discos de dados.
- Não há suporte para o backup de máquinas virtuais com um endereço IP reservado e nenhum ponto de extremidade definido.
- Não há suporte para a substituição de uma máquina virtual existente durante a restauração. Se você tentar restaurar a VM quando ela existir, a operação de restauração falhará.
- Não há suporte para backup e restauração entre regiões.
- Você pode fazer backup de máquinas virtuais em todas as regiões públicas do Azure (consulte o [lista de verificação](https://azure.microsoft.com/regions/#services) das regiões com suporte). Se a região que você procura ainda não tem suporte, ela não aparecerá na lista suspensa durante a criação de cofre.
- Você pode fazer backup de máquinas virtuais somente para determinadas versões de sistemas operacionais:
  - **Linux**: confira [a lista de distribuições endossadas pelo Azure](../virtual-machines/virtual-machines-linux-endorsed-distros.md). Outras distribuições personalizadas do Linux também devem funcionar, contanto que o agente de VM esteja disponível na máquina virtual.
  - **Windows Server**: não há suporte para versões anteriores ao Windows Server 2008 R2.
- A restauração de uma VM DC (controladora de domínio) que é parte de uma configuração multi-DC tem suporte somente usando o PowerShell. Leia mais sobre [como restaurar um controlador de domínio com vários DCs](backup-azure-restore-vms.md#restoring-domain-controller-vms)
- Apenas há suporte para a restauração de máquinas virtuais que têm as seguintes configurações de rede especial por meio do PowerShell. VMs criadas usando o fluxo de trabalho de restauração na interface do usuário não terão essas configurações de rede depois que a operação de restauração for concluída. Para saber mais, confira [Restaurando VMs com configurações de rede especiais](backup-azure-restore-vms.md#restoring-vms-with-special-netwrok-configurations).
  - Máquinas virtuais sob configuração do balanceador de carga (interno e externo)
  - Máquinas virtuais com vários endereços IP reservados
  - Máquinas virtuais com vários adaptadores de rede

## Criar um cofre de Serviços de Recuperação para uma VM

Um cofre dos serviços de recuperação é uma entidade que armazena os backups e os pontos de recuperação criados ao longo do tempo. O cofre dos serviços de recuperação também contêm as políticas de backup associadas às máquinas virtuais protegidas.

Para criar um cofre dos serviços de recuperação:

1. Entre no [Portal do Azure](https://portal.azure.com/).

2. No menu Hub, clique em **Procurar** e, na lista de recursos, digite **Serviços de Recuperação**. Quando você começar a digitar, a lista será filtrada com base em sua entrada. Clique em **Cofre de Serviços de Recuperação**.

    ![Criar Cofre de Serviços de Recuperação - etapa 1](./media/backup-azure-vms-first-look-arm/browse-to-rs-vaults.png) <br/>

    A lista de cofres dos Serviços de Recuperação é exibida.

3. No menu **Cofres de Serviços de Recuperação**, clique em **Adicionar**.

    ![Criar Cofre de Serviços de Recuperação - etapa 2](./media/backup-azure-vms-first-look-arm/rs-vault-menu.png)

    A folha do cofre de Serviços de Recuperação será aberta, solicitando que você forneça o **Nome**, a **Assinatura**, o **Grupo de recursos** e o **Local**.

    ![Criar Cofre de Serviços de Recuperação - etapa 5](./media/backup-azure-vms-first-look-arm/rs-vault-attributes.png)

4. Em **Nome**, insira um nome amigável para identificar o cofre. O nome precisa ser exclusivo para a assinatura do Azure. Digite um nome que contenha de 2 a 50 caracteres. Ele deve começar com uma letra e pode conter apenas letras, números e hifens.

5. Clique em **Assinatura** para ver a lista de assinaturas disponíveis. Se você não tiver certeza sobre qual assinatura usar, utilize a assinatura padrão (ou sugerida). Só haverá múltiplas opções se sua conta organizacional estiver associada a várias assinaturas do Azure.

6. Clique em **Grupo de recursos** para ver a lista dos Grupos de recursos disponíveis ou clique em **Novo** para criar um novo Grupo de recursos. Para saber mais sobre os Grupos de recursos, confira [Uso do Portal do Azure para implantar e gerenciar os recursos do Azure](../azure-portal/resource-group-portal.md)

7. Clique em **Local** para selecionar a região geográfica do cofre. O cofre **deve** estar na mesma região que as máquinas virtuais que você deseja proteger.

    >[AZURE.IMPORTANT] Se você não souber ao certo em qual local sua VM se encontra, feche a caixa de diálogo de criação do cofre e vá para a lista de Máquinas Virtuais no portal. Se você tiver máquinas virtuais em várias regiões, será necessário criar um cofre de Serviços de Recuperação em cada região. Crie o cofre no primeiro local antes de ir para o próximo local. Não é necessário especificar as contas de armazenamento para armazenar os dados de backup - o cofre de Serviços de Recuperação e o serviço de Backup do Azure cuidarão disso automaticamente.

8. Clique em **Criar**. Talvez demore um pouco para o cofre de Serviços de Recuperação ser criado. Monitore as notificações de status na área superior direita no portal. Depois que o cofre é criado, ele aparece na lista de cofres dos Serviços de Recuperação.

    ![Lista de cofres de backup](./media/backup-azure-vms-first-look-arm/rs-list-of-vaults.png)

    Agora que você criou o cofre, saiba como configurar a replicação de armazenamento.

## Definir replicação de armazenamento

A opção de replicação de armazenamento permite que você escolha entre o armazenamento com redundância geográfica e armazenamento com redundância local. Por padrão, seu cofre tem armazenamento com redundância geográfica. Deixe a opção definida como armazenamento com redundância geográfica se este for seu backup principal. Escolha o armazenamento com redundância local se quiser uma opção mais barata que não seja tão durável. Leia mais sobre as opções de armazenamento [com redundância geográfica](../storage/storage-redundancy.md#geo-redundant-storage) e [com redundância local](../storage/storage-redundancy.md#locally-redundant-storage) na [visão geral da replicação do Armazenamento do Azure](../storage/storage-redundancy.md).

Para editar a configuração de replicação de armazenamento:

1. Selecione seu cofre para abrir o painel do cofre e a folha Configurações. Se a folha **Configurações** não abrir, clique em **Todas as configurações** no painel do cofre.

2. Na folha **Configurações**, clique em **Infraestrutura de Backup** > **Configuração de Backup** para abrir a folha **Configuração de Backup**. Na folha **Configuração de Backup**, escolha a opção de replicação de armazenamento para seu cofre.

    ![Lista de cofres de backup](./media/backup-azure-vms-first-look-arm/choose-storage-configuration-rs-vault.png)

    Depois de escolher a opção de armazenamento para o cofre, você estará pronto para associar a VM ao cofre. Para iniciar a associação, você deverá descobrir e registrar as máquinas virtuais do Azure.


## Selecionar a meta de backup, definir a política e os itens a serem protegidos

Antes de registrar uma VM em um cofre, execute o processo de descoberta para garantir que todas as novas máquinas virtuais adicionadas à assinatura sejam identificadas. O processo consulta o Azure quanto à lista de máquinas virtuais na assinatura, juntamente com informações adicionais, como o nome do serviço de nuvem e a região. No portal do Azure, o cenário se refere ao que você vai colocar no cofre de serviços de recuperação. A Política é o agendamento para quando e com que frequência os pontos de recuperação serão feitos. A Política também inclui o período de retenção dos pontos de recuperação.

1. Se você já tiver um cofre de Serviços de Recuperação aberto, vá para a etapa 2. Se você não tiver um cofre dos Serviços de Recuperação aberto, mas estiver no portal do Azure, no menu Hub, clique em **Procurar**.

  - Na lista de recursos, digite **Serviços de Recuperação**.
  - Quando você começar a digitar, a lista será filtrada com base em sua entrada. Quando você vir a opção **Cofres de Serviços de Recuperação**, clique nela.

    ![Criar Cofre de Serviços de Recuperação - etapa 1](./media/backup-azure-vms-first-look-arm/browse-to-rs-vaults.png) <br/>

    A lista de cofres de Serviços de Recuperação aparecerá.
  - Na lista de cofres de Serviços de Recuperação, selecione um cofre.

    O painel de cofres selecionados será aberto.

    ![Abrir a folha do cofre](./media/backup-azure-vms-first-look-arm/vault-settings.png)

2. No menu do painel de cofres, clique em **Backup** para abrir a folha Backup.

    ![Abrir a folha Backup](./media/backup-azure-vms-first-look-arm/backup-button.png)

    Quando a folha for aberta, o serviço de Backup pesquisará novas VMs na assinatura.

    ![Descobrir VMs](./media/backup-azure-vms-first-look-arm/discovering-new-vms.png)

3. Na folha Backup, clique em **Meta de backup** para abrir a folha Meta de Backup.

    ![Abrir a folha Cenário](./media/backup-azure-vms-first-look-arm/select-backup-goal-one.png)

4. Na folha Meta de Backup, defina **Onde está a carga de trabalho em execução** no Azure, **Do que você deseja fazer backup** na Máquina Virtual e clique **OK**.

    A folha Meta de Backup fecha e a folha Política de Backup abre.

    ![Abrir a folha Cenário](./media/backup-azure-vms-first-look-arm/select-backup-goal-two.png)

5. Na folha Política de Backup, selecione a política de backup que você deseja aplicar no cofre e clique em **OK**.

    ![Selecionar a política de backup](./media/backup-azure-vms-first-look-arm/setting-rs-backup-policy-new.png)

    Os detalhes da política padrão estão listados nos detalhes. Se você quiser criar uma nova política, selecione **Criar Nova** no menu suspenso. O menu suspenso também fornece uma opção para alternar a hora quando o instantâneo é criado para 19h. Para obter instruções sobre como definir uma política de backup, confira [Definindo uma política de backup](backup-azure-vms-first-look-arm.md#defining-a-backup-policy). Quando você clicar em **OK**, a política de backup será associada ao cofre.

    Em seguida, escolha as VMs para associar ao cofre.

6. Escolha as máquinas virtuais para associar à política especificada e clique em **Selecionar**.

    ![Selecionar carga de trabalho](./media/backup-azure-vms-first-look-arm/select-vms-to-backup-new.png)

    Se você não vê a VM desejada, verifique se ela existe no mesmo local do Azure que o cofre dos Serviços de Recuperação.

7. Agora que você definiu todas as configurações para o cofre, na folha Backup, clique em **Habilitar Backup** na parte inferior da página. Isso implanta a política para o cofre e as VMs.

    ![Habilitar o backup](./media/backup-azure-vms-first-look-arm/enable-backup-settings-new.png)

A próxima fase de preparação será instalar o Agente de VM ou verificar se ele está instalado.


## Instalar o Agente de VM na máquina virtual

O Agente de VM do Azure deve ser instalado na máquina virtual do Azure para a extensão de Backup funcionar. Se sua VM tiver sido criada da galeria do Azure, o agente de VM já estará presente na máquina virtual. Essas informações são fornecidas para as situações em que você *não* está usando uma VM criada da Galeria do Azure, por exemplo, tendo migrado uma VM de um datacenter local. Nesse caso, o Agente de VM precisa ser instalado para proteger a máquina virtual.

Saiba mais sobre o [Agente de VM](https://go.microsoft.com/fwLink/?LinkID=390493&clcid=0x409) e [como instalá-lo](../virtual-machines/virtual-machines-windows-classic-manage-extensions.md).

Se você tiver problemas para fazer backup da VM do Azure, verifique se o Agente de VM do Azure está instalado corretamente na máquina virtual (veja a tabela abaixo). Se você tiver criado uma VM personalizada, [verifique se a caixa de seleção **Instalar o Agente de VM** está marcada](../virtual-machines/virtual-machines-windows-classic-agents-and-extensions.md) antes da máquina virtual ser provisionada.

A tabela a seguir oferece informações adicionais sobre o Agente de VM para VMs do Windows e do Linux.

| **Operação** | **Windows** | **Linux** |
| --- | --- | --- |
| Instalação do agente de VM | <li>Baixe e instale o [agente MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Você precisará de privilégios de Administrador para concluir a instalação. <li>[Atualize a propriedade de VM](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) para indicar que o agente está instalado. | <li> Instale o [agente Linux](https://github.com/Azure/WALinuxAgent) mais recente do GitHub. Você precisará de privilégios de Administrador para concluir a instalação. <li> [Atualize a propriedade de VM](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) para indicar que o agente está instalado. |
| Atualizar o Agente de VM | Atualizar o agente de VM é tão simples quanto reinstalar os [Binários do Agente de VM](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). <br>Verifique se nenhuma operação de backup está em execução durante a atualização do agente de VM. | Siga as instruções em [como atualizar o Agente de VM do Linux](../virtual-machines-linux-update-agent.md). <br>Verifique se nenhuma operação de backup está em execução durante a atualização do Agente de VM. |
| Validação da instalação do Agente de VM | <li>Navegue até a pasta *C:\\WindowsAzure\\Packages* na VM do Azure. <li>Você deve encontrar o arquivo WaAppAgent.exe presente.<li> Clique com o botão direito do mouse no arquivo, vá para **Propriedades** e selecione a guia **Detalhes**. O campo Versão do Produto deve ser 2.6.1198.718 ou mais recente. | N/D |


### Extensão de backup

Assim que o Agente de VM for instalado na máquina virtual, o serviço Backup do Azure instalará a extensão de backup no Agente de VM. O serviço do Backup do Azure atualiza e corrige continuamente a extensão de backup.

A extensão de backup será instalada pelo serviço Backup, esteja a VM em execução ou não. Uma VM em execução oferece uma maior chance de obter um ponto de recuperação consistente com o aplicativo. No entanto, o serviço do Backup do Azure continuará a realizar o backup da VM mesmo quando ela estiver desativada e a extensão não puder ser instalada. Isso é conhecido como VM Offline. Nesse caso, o ponto de recuperação será *consistente com a falha*.


## Conectividade de rede

Para gerenciar os instantâneos de VM, a extensão de backup precisa de conectividade com os endereços IP públicos do Azure. Sem a conexão correta com a Internet, as solicitações HTTP da máquina virtual atingirão o tempo limite e a operação de backup falhará. Se sua implantação possui restrições de acesso em vigor (por meio de um NSG, Grupo de Segurança de Rede, por exemplo), escolha uma destas opções para fornecer um caminho livre para o tráfego de backup:

- [Lista branca de intervalos de IP de datacenter do Azure](http://www.microsoft.com/pt-BR/download/details.aspx?id=41653): consulte o artigo para obter instruções sobre como colocar os endereços IP na lista branca.
- Implante um servidor de proxy HTTP para rotear o tráfego.

Ao decidir qual opção usar, desvantagens entre a capacidade de gerenciamento, controle granular e custo.

|Opção|Vantagens|Desvantagens|
|------|----------|-------------|
|Intervalos de IPs na lista branca| Sem custo adicional.<br><br>Para habilitar o acesso em NSG, use o cmdlet <i>Set-AzureNetworkSecurityRule</i> | É complexo de gerenciar, já que os intervalos de IP afetados mudam com o tempo.<br><br>Fornece acesso ao Azure por completo, não somente ao Armazenamento.|
|Proxy HTTP| É permitido o controle granular no proxy em relação às URLs de armazenamento, <br>Ponto único de acesso à Internet nas VMs, <br>Não está sujeito a alterações do endereço IP do Azure| Custos adicionais para a execução de uma VM com o software do proxy|

### Realizar a lista branca de intervalos de IP do datacenter do Azure

Para colocar os intervalos IP do datacenter do Azure na lista branca, consulte o [site do Azure](http://www.microsoft.com/pt-BR/download/details.aspx?id=41653) para obter detalhes sobre os intervalos de IP e as instruções.

### Usando um proxy HTTP para backups de uma VM
Ao fazer backup de uma VM, a extensão de backup na VM envia os comandos de gerenciamento de instantâneo para o Armazenamento do Azure usando a API de HTTPS. Roteie o tráfego da extensão de backup por meio do proxy HTTP, pois ele é o único componente configurado para dar acesso à Internet pública.

>[AZURE.NOTE] Não há recomendações do software de proxy a serem usadas. Escolha um proxy que seja compatível com as etapas de configuração abaixo.

A imagem de exemplo abaixo mostra as três etapas de configuração necessárias para usar um proxy HTTP:

- A VM de aplicativo roteia todo o tráfego HTTP voltado para a Internet pública por meio da VM do Proxy.
- A VM do Proxy permite a passagem do tráfego de entrada de VMs na rede virtual.
- O NSG (Grupo de Segurança de Rede) chamado bloqueio de NSF precisa de uma regra de segurança para permitir a passagem de tráfego de Internet da VM do Proxy.

![Diagrama de implantação de proxy HTTP com NSG](./media/backup-azure-vms-prepare/nsg-with-http-proxy.png)

Para usar um proxy HTTP para se comunicar com a Internet pública, siga estas etapas:

#### Etapa 1. Configurar conexões de rede de saída

###### Para computadores Windows
Isso definirá a configuração do servidor de proxy para a Conta do Sistema Local.

1. Baixe o [PsExec](https://technet.microsoft.com/sysinternals/bb897553)
2. Execute o seguinte comando no prompt com privilégios elevados,

     ```
     psexec -i -s "c:\Program Files\Internet Explorer\iexplore.exe"
     ```
    Isso abrirá a janela do Internet Explorer.
3. Acesse Ferramentas -> Opções da Internet -> Conexões -> Configurações de LAN.
4. Verifique as configurações de proxy para a conta do Sistema. Defina o IP e a porta do proxy.
5. Feche o Internet Explorer.

Isso configurará um proxy de todo o computador e será usado para qualquer tráfego de saída HTTP/HTTPS.

Se você configurou um servidor de proxy em uma conta de usuário atual (não uma Conta do Sistema Local), use o script a seguir para aplicá-la ao SYSTEMACCOUNT:

```
   $obj = Get-ItemProperty -Path Registry::”HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections"
   Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" -Name DefaultConnectionSettings -Value $obj.DefaultConnectionSettings
   Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" -Name SavedLegacySettings -Value $obj.SavedLegacySettings
   $obj = Get-ItemProperty -Path Registry::”HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
   Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name ProxyEnable -Value $obj.ProxyEnable
   Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name Proxyserver -Value $obj.Proxyserver
```

>[AZURE.NOTE] Se você receber "(407) Autenticação de Proxy Necessária" no log do servidor de proxy, verifique se a configuração da sua autenticação está correta.

######Para computadores Linux

Adicione a seguinte linha ao arquivo ```/etc/environment```:

```
http_proxy=http://<proxy IP>:<proxy port>
```

Adicione as linhas abaixo ao arquivo ```/etc/waagent.conf```:

```
HttpProxy.Host=<proxy IP>
HttpProxy.Port=<proxy port>
```

#### Etapa 2. Permitir conexões de entrada no servidor de proxy:

1. No servidor proxy, abra o Firewall do Windows. A maneira mais fácil de acessar o firewall é procurar o Firewall do Windows com Segurança Avançada.

    ![Abrir o Firewall](./media/backup-azure-vms-prepare/firewall-01.png)

2. No diálogo Firewall do Windows, clique com o botão direito do mouse em **Regras de Entrada** e clique em **Nova Regra...**

    ![Criar uma nova regra](./media/backup-azure-vms-prepare/firewall-02.png)

3. No **Assistente para Nova Regra de Entrada**, selecione a opção **Personalizar** para **Tipo de Regra** e clique em **Avançar**.
4. Na página para selecionar **Programa**, escolha **Todos os Programas** e clique em **Avançar**.

5. Na página **Protocolo e Portas**, insira as seguintes informações e clique em **Avançar**:

    ![Criar uma nova regra](./media/backup-azure-vms-prepare/firewall-03.png)

    - para *Tipo de protocolo*, escolha *TCP*
    - para *Porta local*, escolha *Portas Específicas*, no campo abaixo especifique o ```<Proxy Port>``` que foi configurado.
    - para *Porta remota*, escolha *Todas as Portas*

    Para o restante do assistente, clique até o fim e dê um nome a essa regra.

#### Etapa 3. Adicionar uma regra de exceção ao NSG:

Em um prompt de comando do Azure PowerShell, digite o seguinte comando:

O comando a seguir adiciona uma exceção ao NSG. Essa exceção permite a passagem de tráfego TCP de qualquer porta em 10.0.0.5 para qualquer endereço da Internet na porta 80 (HTTP) ou 443 (HTTPS). Se você exigir uma porta específica na Internet pública, adicione-a também ao ```-DestinationPortRange```.

```
Get-AzureNetworkSecurityGroup -Name "NSG-lockdown" |
Set-AzureNetworkSecurityRule -Name "allow-proxy " -Action Allow -Protocol TCP -Type Outbound -Priority 200 -SourceAddressPrefix "10.0.0.5/32" -SourcePortRange "*" -DestinationAddressPrefix Internet -DestinationPortRange "80-443"
```


*Essas etapas usam valores e nomes específicos para esse exemplo. Use os nomes e valores da sua implantação ao inserir, recortar e colar os detalhes no seu código.*


Agora que sabemos que há conectividade de rede, você está pronto para fazer backup da sua VM. Confira [Back up Resource Manager-deployed VMs](backup-azure-arm-vms.md) (Fazer backup das máquinas virtuais implantadas com o Gerenciador de Recursos).

## Perguntas?
Se você tiver dúvidas ou gostaria de ver algum recurso incluído, [envie-nos seus comentários](http://aka.ms/azurebackup_feedback).

## Próximas etapas
Agora que você já preparou seu ambiente para fazer backup de sua VM, a próxima etapa lógica será criar um backup. O artigo de planejamento oferece informações mais detalhadas sobre o backup de máquinas virtuais.

- [Backup de máquinas virtuais](backup-azure-vms.md)
- [Planeje sua infraestrutura de backup da VM](backup-azure-vms-introduction.md)
- [Gerenciar backups de máquinas virtuais](backup-azure-manage-vms.md)

<!---HONumber=AcomDC_0608_2016-->