---
title: Proteger aplicativo Web do Windows para setores regulamentados
description: Cenário comprovado para criar um aplicativo Web seguro e de várias camadas com o Windows Server no Azure que usa conjuntos de dimensionamento, o Gateway de Aplicativo e balanceadores de carga.
author: iainfoulds
ms.date: 07/11/2018
ms.openlocfilehash: 780b82791510b6ca06ef918b66d2547794dfcf87
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428747"
---
# <a name="secure-windows-web-application-for-regulated-industries"></a>Proteger aplicativo Web do Windows para setores regulamentados

Este exemplo de cenário é aplicável para setores regulamentados que têm uma necessidade de proteger aplicativos de várias camadas. Nesse cenário, um aplicativo ASP.NET de front-end se conecta de forma segura a um cluster de Microsoft SQL Server de back-end protegido.

Os cenários de aplicativo de exemplo incluem a execução de aplicativos de sala de operações, horas marcadas de pacientes e manutenção de registros ou renovação e pedidos de receitas médicas. Tradicionalmente, as organizações tinham que manter os aplicativos locais herdados e os serviços para esses cenários. Com uma maneira segura de forma e escalonável para implantar esses aplicativos do Windows Server no Azure, as organizações podem modernizar sua implantações e reduzir seus custos operacionais locais e despesas gerais de manutenção.

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere este cenário para os casos de uso a seguir:

* Modernizar as implantações de aplicativo em um ambiente de nuvem seguro.
* Reduzir o gerenciamento de serviços e aplicativos locais herdados.
* Melhorar a saúde e experiência de pacientes com novas plataformas de aplicativo.

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes do Azure envolvidos no aplicativo do Windows Server de várias camadas para setores regulamentados][architecture]

Este cenário aborda um aplicativo de setores regulamentados de várias camadas que usa o ASP.NET e o Microsoft SQL Server. O fluxo de dados neste cenário ocorre da seguinte forma:

1. Os usuários acessam o aplicativo front-end de setores regulamentados do ASP.NET por meio de um Gateway de Aplicativo do Azure.
2. O Gateway de Aplicativo distribui o tráfego para instâncias de VM dentro de um conjunto de dimensionamento de máquina virtual do Azure.
3. O aplicativo ASP.NET conecta-se ao cluster do Microsoft SQL Server em uma camada de back-end por meio de um balanceador de carga do Azure. Essas instâncias do SQL Server de back-end estão em uma Azure rede virtual separada do Azure, protegida pelas regras do grupo de segurança de rede que limitam o fluxo de tráfego.
4. O balanceador de carga distribui o tráfego do SQL Server para instâncias de VM em outro conjunto de dimensionamento de máquina virtual.
5. O Armazenamento de Blobs do Azure atua como uma testemunha de nuvem para o cluster do SQL Server na camada de back-end.  A conexão de dentro da VNet é habilitado com um ponto de extremidade de serviço de rede virtual para o Armazenamento do Azure.

### <a name="components"></a>Componentes

* O [Gateway de Aplicativo do Azure][appgateway-docs] é um balanceador de carga de tráfego da Web do protocolo Layer 7 que reconhece aplicativos e pode distribuir o tráfego com base em regras de roteamento específicas. Gateway de aplicativo também pode lidar com o descarregamento de SSL para um desempenho de servidor Web aprimorado.
* A [Rede Virtual do Azure][vnet-docs] permite vários tipos de recursos, como VMs do Azure, a fim de se comunicar de forma segura com a Internet, com as redes locais e com outras VMs. Redes virtuais fornecem isolamento e segmentação, filtram e roteiam o tráfego e permitem a conexão entre locais. Duas redes virtuais combinadas com os NSGs apropriados são usadas neste cenário para fornecer uma [zona desmilitarizada][dmz] (DMZ) e o isolamento dos componentes do aplicativo. O emparelhamento de rede virtual conecta as duas redes.
* O [conjunto de dimensionamento de máquinas virtuais do Azure][scaleset-docs] permite criar e gerenciar um grupo de VMs idênticas e com balanceamento de carga. O número de instâncias de VM pode aumentar ou diminuir automaticamente em resposta à demanda ou a um agendamento definido. Dois conjuntos de dimensionamento de máquina virtual separados são usados nesse cenário: um para as instâncias de aplicativos ASP.NET de front-end e outro para as instâncias de VM do cluster de SQL Server de back-end. A configuração de estado desejado (DSC) do PowerShell ou a extensão de script personalizado do Azure pode ser usada para provisionar as instâncias de VM com o software e as definições de configuração necessárias.
* [Grupos de segurança de rede do Azure][nsg-docs] contêm uma lista de regras de segurança que permitem ou negam o tráfego de rede de entrada ou saída com base no endereço IP de origem ou destino, na porta e no protocolo. As redes virtuais neste cenário são protegidas com regras de grupo de segurança de rede que restringem o fluxo de tráfego entre os componentes do aplicativo.
* O [balanceador de carga do Azure][loadbalancer-docs] distribui o tráfego de entrada de acordo com regras e investigações de integridade. O balanceador de carga fornece baixa latência e alta taxa de transferência e pode ser escalado verticalmente em milhões de fluxos para aplicativos TCP e UDP. Um balanceador de carga interno é usado neste cenário para distribuir o tráfego da camada de aplicativo de front-end para o cluster do SQL Server de back-end.
* O [Armazenamento de Blobs][cloudwitness-docs] do Azure atua como uma testemunha de nuvem para o cluster do SQL Server. Este testemunha é usada para operações de cluster e as decisões que exigem um voto adicional para decidir o quorum. Usar testemunha de nuvem elimina a necessidade de uma VM adicional atuar como uma testemunha de compartilhamento de arquivos tradicional.

### <a name="alternatives"></a>Alternativas

* *O Nix, no Windows, pode ser facilmente substituído por vários outros sistemas operacionais como se nada na infraestrutura dependesse do sistema operacional.

* O [SQL Server para Linux][sql-linux] pode substituir o armazenamento de dados de back-end.

* O [Cosmos DB][cosmos] é outra alternativa para o armazenamento de dados.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

Nesse cenário, as instâncias de VM são implantadas em Zonas de Disponibilidade. Cada zona é composta por um ou mais datacenters equipados com energia, resfriamento e rede independentes. Um mínimo de três zonas estão disponíveis em todas as regiões habilitadas. Essa distribuição de instâncias de VM em zonas fornece alta disponibilidade para as camadas de aplicativo. Para obter mais informações, consulte [o que são Zonas de Disponibilidade no Azure?][azureaz-docs]

A camada de banco de dados pode ser configurada para usar grupos de disponibilidade Always On. Com essa configuração do SQL Server, um banco de dados primário em um cluster é configurado com até oito bancos de dados secundários. Se ocorrer um problema com o banco de dados primário, o cluster faz o failover para um banco de dados secundários, permitindo que o aplicativo continue disponível. Para saber mais informações, confira [Visão geral dos grupos de disponibilidade Always On para SQL Server][sqlalwayson-docs].

Para ver outros tópicos sobre disponibilidade, consulte a [lista de verificação de disponibilidade][availability] no Azure Architecture Center.

### <a name="scalability"></a>Escalabilidade

Esse cenário usa conjuntos de dimensionamento de máquina virtual para os componentes de front-end e back-end. Com conjuntos de dimensionamento, o número de instâncias VM que executam a camada de aplicativo de front-end pode dimensionar automaticamente em resposta à demanda do cliente ou com base em um agendamento definido. Para saber mais, veja [Visão geral sobre dimensionamento automático com conjuntos de dimensionamento de máquinas virtuais][vmssautoscale-docs].

Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Todo o tráfego de rede virtual para a camada de aplicativo de front-end e protegido por grupos de segurança de rede. As regras limitam o fluxo de tráfego para que somente as instâncias de VM da camada de aplicativo de front-end possam acessar a camada de banco de dados de back-end. Nenhum tráfego de Internet de saída é permitido da camada de banco de dados. Para reduzir a superfície de ataque, nenhuma porta de gerenciamento remoto direto fica aberta. Para saber mais, confira [Grupos de segurança de rede do Azure][nsg-docs].

Para exibir as diretrizes sobre como implantar a [infraestrutura compatível com][pci-dss] Payment Card Industry Data Security Standards (PCI DSS 3.2). Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Em combinação com o uso de Zonas de Disponibilidade e conjuntos de dimensionamento de máquina virtual, esse cenário usa o Gateway de Aplicativo do Azure e o balanceador de carga. Esses dois componentes de rede distribuem o tráfego para as instâncias de VM conectadas e incluem as investigações de integridade que garantem que o tráfego é distribuído apenas para VMs íntegras. Duas instâncias de Gateway de Aplicativo são configuradas em uma configuração ativo-passivo, e um balanceador de carga com redundância de zona é usado. Essa configuração torna os recursos de rede e aplicativo resilientes a problemas que, de outra forma, interromperiam o tráfego e impactariam o acesso do usuário final.

Para obter diretrizes gerais sobre como criar cenários resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implantar o cenário

**Pré-requisitos.**

* Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
* Para implantar um cluster do SQL Server no conjunto de dimensionamento de back-end, você precisaria de um domínio nos Azure Active Directory Domain Services.

Para implantar a infraestrutura básica para esse cenário com um modelo do Azure Resource Manager, execute as etapas a seguir.

1. Selecione o botão **Implantar no Azure**:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Aguarde até que a implantação de modelo seja aberta no portal do Azure e conclua as seguintes etapas:
   * Escolha **Criar novo** grupo de recursos e insira um nome como *myWindowsscenario* na caixa de texto.
   * Selecione uma região na caixa suspensa **Local**.
   * Forneça um nome de usuário e uma senha segura para as instâncias do conjunto de dimensionamento da máquina virtual.
   * Analise os termos e condições e marque a opção **Concordo com os termos e condições declarados acima**.
   * Selecione o botão **Comprar**.

Pode levar 15 a 20 minutos para a implantação ser concluída.

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos.  Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Nós fornecemos três perfis de custo de exemplo com base no número de instâncias de VM do conjunto de dimensionamento que executam seus aplicativos.

* [Pequeno][small-pricing]: esse exemplo de preço refere-se a duas instâncias de VM de front-end e duas de back-end.
* [Médio][medium-pricing]: esse exemplo de preço refere-se a 20 instâncias de VM de front-end e cinco de back-end.
* [Grande][large-pricing]: esse exemplo de preço refere-se a 100 instâncias de VM de front-end e 10 de back-end.

## <a name="related-resources"></a>Recursos relacionados

Esse cenário usou um conjunto de dimensionamento de máquina virtual de back-end que executa um cluster do Microsoft SQL Server. O Azure Cosmos DB também pode ser usado como uma camada de banco de dados escalonável e segura para os dados de aplicativo. Um [ponto de extremidade de serviço da rede virtual do Azure][vnetendpoint-docs] permite que você possa proteger os recursos essenciais do serviço do Azure somente para suas redes virtuais. Nesse cenário, os pontos de extremidade de rede virtual permitem proteger o tráfego entre a camada de aplicativo de front-end e o Cosmos DB. Para obter mais informações sobre o Cosmos DB, consulte [Visão geral do Azure Cosmos DB][azurecosmosdb-docs].

Você também ter uma visão detalhada de uma [arquitetura de referência para um aplicativo de N camadas genérico com o SQL Server][ntiersql-ra].

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/regulated-multitier-app/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[azureaz-docs]: /azure/availability-zones/az-overview
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/ 
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability 
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[cosmos]: /azure/cosmos-db/
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017

[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93