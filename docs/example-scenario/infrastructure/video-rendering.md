---
title: Renderização de vídeo em 3D no Azure
description: Executar cargas de trabalho de HPC nativas no Azure usando o serviço de Lote do Azure
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 67dc8496656a75eab8f5d0ce45d00f8b1f4ea03f
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428050"
---
# <a name="3d-video-rendering-on-azure"></a>Renderização de vídeo em 3D no Azure

A renderização 3D é um processo demorado que exige uma quantidade significativa de tempo de CPU para ser concluída.  Em um único computador, o processo de geração de um arquivo de vídeo de ativos estáticos pode levar horas ou mesmo dias dependendo do tamanho e da complexidade do vídeo criado.  Muitas empresas compram computadores topo de linha para executar essas tarefas ou investem em grandes farms de renderização para os quais elas podem enviar trabalhos.  No entanto, ao tirar proveito do Lote do Azure, esse poder está disponível para você quando necessário e deixa de estar disponível quando você não precisa dele, tudo isso sem nenhum investimento de capital.

O Lote oferece uma experiência consistente de gerenciamento e agendamento de trabalhos, quer você opte por nós de computação do Windows Server ou do Linux. Com o Lote, você pode usar os seus aplicativos Windows ou Linux existentes, incluindo AutoDesk Maya e Blender, para executar em trabalhos de renderização de grande escala no Azure.

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere este cenário para esses casos de uso semelhantes:

* Modelagem 3D
* Renderização visual FX (VFX)
* Transcodificação de vídeo
* Processamento de imagens, correção de cor e redimensionamento

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes envolvidos em uma solução de HPC nativa de nuvem usando o Lote do Azure][architecture]

Este exemplo de cenário aborda o fluxo de trabalho ao usar o Lote do Azure, o fluxo de dados ocorre da seguinte maneira:

1. Carregar os arquivos de entrada e o aplicativo para processar esses arquivos em sua conta do Armazenamento do Azure
2. Criar um pool de nós de computação do Lote na sua conta do Lote, um trabalho para executar a carga de trabalho no pool e tarefas no trabalho.
3. Baixe arquivos de entrada e aplicativos para o Lote
4. Monitorar a execução de tarefas
5. Carregar a saída da tarefa
6. Baixar os arquivos de saída

Para simplificar esse processo, você também pode usar os [plug-ins do Lote para Maya & 3ds Max][batch-plugins]

### <a name="components"></a>Componentes

O Lote do Azure tem como base as tecnologias do Azure a seguir:

* [Grupos de Recursos][resource-groups] são um contêiner lógico para recursos do Azure.
* [Redes Virtuais][vnet] são usadas para o nó do cabeçalho e os recursos de computação
* Contas de [Armazenamento][storage] são usadas para a retenção de dados e a sincronização
* [Conjuntos de Dimensionamento de Máquinas Virtuais][vmss] são usados pelo CycleCloud para recursos de computação

## <a name="considerations"></a>Considerações

### <a name="machine-sizes-available-for-azure-batch"></a>Tamanhos de máquina disponíveis para o Lote do Azure

Embora a maioria dos clientes de renderização escolha recursos com alta potência de CPU, outras cargas de trabalho que usam conjuntos de dimensionamento de máquinas virtuais podem escolher as VMs de forma diferente e dependerão de vários fatores:

* O aplicativo que está sendo executado tem um limite de memória?
* O aplicativo precisa usar GPUs? 
* Os tipos de trabalho são paralelos ou precisam de conectividade do Infiniband para trabalhos firmemente acoplados?
* Exige E/S rápida para o armazenamento em nós de computação

O Azure tem uma grande variedade de tamanhos de VM que podem atender todos os requisitos de aplicativo acima. Alguns são específicos para HPC, mas até mesmo os tamanhos menores podem ser utilizados para fornecer uma implementação de grade efetiva:

* [Tamanhos de VM de HPC][compute-hpc] Devido à limitação natural de renderização da CPU, a Microsoft normalmente sugere VMs da série H do Azure.  Esse tipo de VM é criado especificamente para necessidades de computação de alto nível, está disponível em tamanhos de 8 e 16 núcleos de vCPU, memória DDR4, armazenamento temporário de SSD e tecnologia Haswell E5 da Intel.
* [Tamanhos de VM de GPU][compute-gpu] Os tamanhos de VM otimizados para GPU são máquinas virtuais especializadas disponíveis com um ou vários GPUs NVIDIA. Esses tamanhos são projetados para cargas de trabalho de visualização e com muita computação e muitos gráficos.
* Os tamanhos NC, NCv2, NCv3 e ND são otimizados para aplicativos de rede e computação intensiva e algoritmos, incluindo aplicativos baseados em CUDA e OpenCL e simulações, AI e Aprendizagem Profunda. Os tamanhos NV são otimizados e projetados para cenários de visualização remota, streaming, jogos, codificação e VDI usando estruturas como OpenGL e DirectX.
* [Tamanhos de VM otimizados para memória][compute-memory] Quando for necessária mais memória, os tamanhos de VM otimizados para memória oferecem uma maior proporção de memória para CPU.
* [Tamanhos de VM para uso geral][compute-general] Tamanhos de VM para uso geral também estão disponíveis e oferecem uma proporção balanceada de CPU e memória.

### <a name="alternatives"></a>Alternativas

Se você precisar de mais controle sobre seu ambiente de renderização no Azure ou precisar de uma implementação híbrida, então a computação CycleCloud pode ajudar a coordenar uma grade de IaaS na nuvem. Usando as mesmas tecnologias subjacentes do Azure como Lote do Azure, ele torna a criação e manutenção de uma grade de IaaS um processo eficiente. Para obter mais informações e saber mais sobre os princípios de design, use o seguinte link:

Para obter uma visão geral completa de todas as soluções de HPC que estão disponíveis para você no Azure, consulte o artigo [Soluções de HPC, Lote e Big Compute usando VMs do Azure][hpc-alt-solutions]

### <a name="availability"></a>Disponibilidade

O monitoramento dos componentes do Lote do Azure está disponível por meio de vários serviços, ferramentas e APIs. Isso é discutido mais detalhadamente no artigo [Monitorar soluções do Lote][batch-monitor].

### <a name="scalability"></a>Escalabilidade

Pools dentro de uma conta do Lote do Azure podem ser dimensionados por meio de intervenção manual ou, usando uma fórmula com base em métricas do Lote do Azure, podem ser dimensionados automaticamente. Para obter mais informações sobre escalabilidade, consulte o artigo [Criar uma fórmula de dimensionamento automático para dimensionar nós em um pool do Lote][batch-scaling].

### <a name="security"></a>Segurança

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Embora não exista atualmente nenhuma funcionalidade de failover no Lote do Azure, é recomendável usar as seguintes etapas para garantir a disponibilidade em caso de interrupção não planejada:

* Criar uma conta do Lote do Azure em um local alternativo do Azure com uma conta de armazenamento alternativa
* Criar os mesmos pools de nós com o mesmo nome, com nenhum nó alocado
* Certificar-se de que os aplicativos são criados e atualizados para a conta de armazenamento alternativa
* Carregar arquivos de entrada e enviar trabalhos para a conta do Lote do Azure alternativa

## <a name="deploy-this-scenario"></a>Implantar este cenário

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>Criar uma conta do Lote do Azure e pools manualmente

Este cenário de exemplo ajudará a aprender como o Lote do Azure funciona durante a apresentação de Laboratórios do Lote do Azure como um exemplo de solução de SaaS que pode ser desenvolvida para seus próprios clientes:

[Masterclass do Lote do Azure][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a>Implantar o cenário de exemplo usando um modelo do Azure Resource Manager

O modelo será implantado:

* Uma nova conta do Lote do Azure
* Uma conta de armazenamento
* Um pool de nós associado à conta do lote
* O pool de nós será configurado para usar VMs de A2 v2 com imagens do Ubuntu da Canonical
* O pool de nós conterá zero VMs inicialmente e exigirá dimensionando manual para adicionar máquinas virtuais

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

[Saiba mais sobre modelos do Resource Manager][azure-arm-templates]

## <a name="pricing"></a>Preços

O custo de usar o Lote do Azure dependerá dos tamanhos de VM que são usados para os pools e quanto tempo essas VMs ficam alocadas e em execução, não há nenhum custo associado a uma criação de conta do Lote do Azure. A saída de dados e o armazenamento devem ser considerados já que implicarão em custos adicionais.

A seguir estão exemplos de custos que poderiam ser cobrados para um trabalho que termina em 8 horas usando um número diferente de servidores:

* 100 VMs de CPU de alto desempenho: [estimativa de custo][hpc-est-high]

  100 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB

* 50 VMs de CPU de alto desempenho: [estimativa de custo][hpc-est-med]

  50 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB

* 10 VMs de CPU de alto desempenho: [estimativa de custo][hpc-est-low]
  
  10 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB

### <a name="low-priority-vm-pricing"></a>Preços de VM de baixa prioridade

O Lote do Azure também dá suporte ao uso de VMs de baixa prioridade* nos pools de nó, que potencialmente podem fornecer uma economia significativa de custos. Para obter uma comparação de preços entre as VMs padrão e as VMs de baixa prioridade e para obter mais informações sobre as VMs de baixa prioridade, consulte [Preços do Lote][batch-pricing].

\* Observe que somente determinados aplicativos e cargas de trabalho serão adequados para execução em VMs de baixa prioridade.

## <a name="related-resources"></a>Recursos relacionados

[Visão Geral do Lote do Azure][batch-overview]

[Documentação do Lote do Azure][batch-doc]

[Usar contêineres no Lote do Azure][batch-containers]

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job