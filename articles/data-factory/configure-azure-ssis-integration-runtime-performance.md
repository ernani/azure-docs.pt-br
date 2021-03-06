---
title: Configurar o desempenho para o Azure-SSIS Integration Runtime
description: Saiba como configurar as propriedades do Microsoft Integration Runtime do Azure-SSIS para alto desempenho
services: data-factory
ms.date: 01/10/2018
ms.topic: conceptual
ms.service: data-factory
ms.workload: data-services
author: swinarko
ms.author: sawinark
ms.reviewer: ''
manager: anandsub
ms.openlocfilehash: ca88e42438c7cb48b062aa67d82053afbb9244bf
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "81418279"
---
# <a name="configure-the-azure-ssis-integration-runtime-for-high-performance"></a>Configurar o Microsoft Integration Runtime do Azure-SSIS para alto desempenho

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Este artigo descreve como configurar um IR (Microsoft Integration Runtime) do Azure-SSIS para alto desempenho. O IR do Azure-SSIS permite que você implante e execute pacotes SSIS (SQL Server Integration Services) no Azure. Para obter mais informações sobre o IR do Azure-SSIS, consulte o artigo [Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime). Para obter informações sobre como implantar e executar pacotes SSIS no Azure, consulte [Método lift-and-shift de cargas de trabalho do SQL Server Integration Services para a nuvem](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview).

> [!IMPORTANT]
> Este artigo contém resultados de desempenho e observações de testes internos realizados por membros da equipe de desenvolvimento do SSIS. Os resultados podem variar. Faça seu próprio teste antes de finalizar suas configurações, o que afeta custo e desempenho.

## <a name="properties-to-configure"></a>Propriedades para configurar

A seguinte parte de um script de configuração mostra as propriedades que você pode configurar ao criar um Integration Runtime do Azure-SSIS. Para obter o script e a descrição completos do PowerShell, consulte [Implantar pacotes do SQL Server Integration Services no Azure](tutorial-deploy-ssis-packages-azure-powershell.md).

```powershell
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime information - This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[specify a name for your Azure-SSIS IR]"
$AzureSSISDescription = "[specify a description for your Azure-SSIS IR]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, while Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to max(2 x number of cores, 8) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL to provide SAS URI of blob container where your custom setup script and its associated files are stored
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use Azure SQL Database with virtual network service endpoints/Managed Instance/on-premises data, Azure Resource Manager virtual network is recommended, Classic virtual network will be deprecated soon
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Please use the same subnet as the one used with your Azure SQL Database with virtual network service endpoints or a different subnet than the one used for your Managed Instance

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name or Managed Instance name.DNS prefix].database.windows.net" # WARNING: Please ensure that there is no existing SSISDB, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure Active Directory (AAD)
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for AAD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for AAD authentication]"
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database or leave it empty for Managed Instance]"
```

## <a name="azuressislocation"></a>AzureSSISLocation
**AzureSSISLocation** é o local para o nó de trabalho do runtime de integração. O nó de trabalho mantém uma conexão constante ao SSISDB (banco de dados do Catálogo SSIS) em um Banco de dados SQL do Azure. Defina **AzureSSISLocation** para o mesmo local que o servidor do Banco de Dados SQL do Microsoft Azure que hospeda o SSISDB, permitindo que o runtime de integração funcione tão eficientemente quanto possível.

## <a name="azuressisnodesize"></a>AzureSSISNodeSize
Data Factory, incluindo o IR do Azure-SSIS, dá suporte às opções a seguir:
-   Standard\_A4\_v2
-   Standard\_A8\_v2
-   Standard\_D1\_v2
-   Standard\_D2\_v2
-   Standard\_D3\_v2
-   Standard\_D4\_v2
-   D2\_\_v3 Standard
-   D4\_\_v3 Standard
-   Standard\_D8\_v3
-   Standard\_D16\_v3
-   Standard\_D32\_v3
-   Standard\_D64\_v3
-   Standard\_E2\_v3
-   Standard\_E4\_v3
-   Standard\_E8\_v3
-   Standard\_E16\_v3
-   Standard\_E32\_v3
-   Standard\_E64\_v3

No teste interno não oficial da equipe de engenharia do SSIS, a série D mostra-se ser mais adequada para a execução do pacote SSIS do que a série A.

-   A taxa de desempenho/preço da série D é maior do que a série a e a taxa de desempenho/preço da série V3 é maior do que a série v2.
-   A taxa de transferência para a série D é maior do que a série a com o mesmo preço e a taxa de transferência da série V3 é maior do que a série v2 pelo mesmo preço.
-   Os nós da série v2 de Azure-SSIS IR não são adequados para a instalação personalizada, portanto, use os nós da série V3 em vez disso. Se você já usa os nós da série v2, altere para usar os nós da série V3 assim que possível.
-   A série E tem tamanhos de VM com otimização de memória que fornecem uma taxa maior de memória para CPU do que outras máquinas. Se o pacote exigir muita memória, você pode considerar escolher a VM da série E.

### <a name="configure-for-execution-speed"></a>Configurar para velocidade de execução
Caso não tenha muitos pacotes para executar e deseja que os pacotes sejam executados rapidamente, utilize as informações no gráfico a seguir para escolher um tipo de máquina virtual adequado ao seu cenário.

Esses dados representam uma única execução de pacote em um nó de trabalho único. O pacote carrega 3 milhões registros com as colunas First Name e Last Name do armazenamento de BLOBs do Azure, gera uma coluna Full Name e grava os registros que têm o nome completo com mais de 20 caracteres para o armazenamento de BLOBs do Azure.

![Velocidade de execução do pacote Integration Runtime do SSIS](media/configure-azure-ssis-integration-runtime-performance/ssisir-execution-speedV2.png)

### <a name="configure-for-overall-throughput"></a>Configurar a taxa de transferência geral

Caso você tenha muitos lotes de pacotes para executar, e se preocupa mais com a taxa de transferência geral, utilize as informações no gráfico a seguir para escolher um tipo de máquina virtual adequado ao seu cenário.

![Taxa de transferência geral máxima do Integration Runtime do SSIS](media/configure-azure-ssis-integration-runtime-performance/ssisir-overall-throughputV2.png)

## <a name="azuressisnodenumber"></a>AzureSSISNodeNumber

**AzureSSISNodeNumber** ajusta a escalabilidade do runtime de integração. A taxa de transferência do runtime de integração é proporcional ao **AzureSSISNodeNumber**. Primeiramente, defina **AzureSSISNodeNumber** para um valor pequeno, monitore a taxa de transferência do runtime de integração e, em seguida, ajuste o valor para o seu cenário. Para reconfigurar a contagem do nó de trabalho, consulte [Gerenciar um runtime de integração do Azure-SSIS](manage-azure-ssis-integration-runtime.md).

## <a name="azuressismaxparallelexecutionspernode"></a>AzureSSISMaxParallelExecutionsPerNode

Quando você já estiver utilizando um nó de trabalho avançado para executar pacotes, aumentar o **AzureSSISMaxParallelExecutionsPerNode** poderá aumentar a taxa de transferência geral do runtime de integração. Para nós Standard_D1_v2, há suporte para execuções paralelas 1-4 por nó. Para todos os outros tipos de nós, são suportados 1-Max (2 x número de núcleos, 8) execuções paralelas por nó. Se você quiser **AzureSSISMaxParallelExecutionsPerNode** além do valor máximo com suporte, poderá abrir um tíquete de suporte e poderemos aumentar o valor máximo para você e depois disso, você precisará usar o Azure PowerShell para atualizar o **AzureSSISMaxParallelExecutionsPerNode**.
É possível estimar o valor apropriado com base no custo do pacote e nas seguintes configurações para os nós de trabalho. Para obter mais informações, consulte [Tamanhos das máquinas virtuais para uso geral](../virtual-machines/windows/sizes-general.md).

| Tamanho             | vCPU | Memória: GiB | Armazenamento temporário (SSD) GiB | Taxa de transferência máxima de armazenamento temporário: IOPS / MBps de leitura / MBps de gravação | Discos de dados máximos / taxa de transferência: IOPS | Máximo de NICs/Desempenho de rede esperado (Mbps) |
|------------------|------|-------------|------------------------|------------------------------------------------------------|-----------------------------------|------------------------------------------------|
| Standard\_D1\_v2 | 1    | 3,5         | 50                     | 3000 / 46 / 23                                             | 2 / 2 x 500                         | 2 / 750                                        |
| Standard\_D2\_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 4 / 4 x 500                         | 2 / 1500                                       |
| Standard\_D3\_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 8 / 8 x 500                         | 4 / 3000                                       |
| Standard\_D4\_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 16 / 16 x 500                       | 8 / 6000                                       |
| Standard\_A4\_v2 | 4    | 8           | 40                     | 4000 / 80 / 40                                             | 8 / 8 x 500                         | 4 / 1000                                       |
| Standard\_A8\_v2 | 8    | 16          | 80                     | 8000 / 160 / 80                                            | 16 / 16 x 500                       | 8 / 2000                                       |
| D2\_\_v3 Standard | 2    | 8           | 50                     | 3000 / 46 / 23                                             | 4/6x500                         | 2 / 1000                                       |
| D4\_\_v3 Standard | 4    | 16          | 100                    | 6000 / 93 / 46                                             | 8/12x500                        | 2 / 2000                                       |
| Standard\_D8\_v3 | 8    | 32          | 200                    | 12000 / 187 / 93                                           | 16/24x500                       | 4 / 4000                                       |
| Standard\_D16\_v3| 16   | 64          | 400                    | 24000 / 375 / 187                                          | 32/48x500                        | 8 / 8000                                       |
| Standard\_D32\_v3| 32   | 128         | 800                    | 48000 / 750 / 375                                          | 32/96x500                       | 8 / 16000                                      |
| Standard\_D64\_v3| 64   | 256         | 1600                   | 96000/1000/500                                         | 32/192x500                      | 8 / 30000                                      |
| Standard\_E2\_v3 | 2    | 16          | 50                     | 3000 / 46 / 23                                             | 4/6x500                         | 2 / 1000                                       |
| Standard\_E4\_v3 | 4    | 32          | 100                    | 6000 / 93 / 46                                             | 8/12x500                        | 2 / 2000                                       |
| Standard\_E8\_v3 | 8    | 64          | 200                    | 12000 / 187 / 93                                           | 16/24x500                       | 4 / 4000                                       |
| Standard\_E16\_v3| 16   | 128         | 400                    | 24000 / 375 / 187                                          | 32/48x500                       | 8 / 8000                                       |
| Standard\_E32\_v3| 32   | 256         | 800                    | 48000 / 750 / 375                                          | 32/96x500                       | 8 / 16000                                      |
| Standard\_E64\_v3| 64   | 432         | 1600                   | 96000/1000/500                                         | 32/192x500                      | 8 / 30000                                      |

Aqui estão as diretrizes para definir o valor direito para a propriedade **AzureSSISMaxParallelExecutionsPerNode**: 

1. Primeiramente, defina-o para um valor pequeno.
2. Aumente-o por em pequena quantidade para verificar se a taxa de transferência geral é aperfeiçoada.
3. Pare de aumentar o valor quando a taxa de transferência geral atingir o valor máximo.

## <a name="ssisdbpricingtier"></a>SSISDBPricingTier

**SSISDBPricingTier** é o tipo de preço para o SSISDB (Banco de dados do catálogo do SSIS) em um banco de dados SQL do Azure. Essa configuração afeta o número máximo de funções de trabalho na instância do IR, a velocidade para colocar na fila uma execução de pacote e a velocidade para carregar o log de execução.

-   Se você não se preocupar com a velocidade para colocar na fila a execução do pacote e para carregar o log de execução, você poderá escolher o menor tipo de preço do banco de dados. O banco de dados SQL do Azure com preço Básico fornece suporte a 8 funções de trabalho em uma instância de runtime de integração.

-   Escolha um banco de dados mais avançado do que Básico, se o número de funções de trabalho for superior a 8 ou a contagem do núcleo for superior a 50. Caso contrário, o banco de dados se torna o gargalo da instância de runtime de integração e o desempenho geral é impactado negativamente.

-   Escolha um banco de dados mais potente, como S3, se o nível de log estiver definido como detalhado. De acordo com nosso teste interno não oficial, o tipo de preço S3 pode dar suporte à execução do pacote SSIS com dois nós, 128 contagens paralelas e nível de log detalhado.

Também é possível ajustar o tipo de preço do banco de dados com base nas informações de uso de DTU [(unidade de transações do banco de dados)](../sql-database/sql-database-what-is-a-dtu.md) disponível no Portal do Azure.

## <a name="design-for-high-performance"></a>Design para alto desempenho
Projetar um pacote SSIS para executar no Azure é diferente de projetar um pacote para execução local. Em vez de combinar várias tarefas independentes no mesmo pacote, separe-as em vários pacotes para uma execução mais eficiente no IR do Azure-SSIS. Crie uma execução de pacote para cada pacote, de modo que não precisem aguardar uns aos outros para concluir. Essa abordagem beneficia da escalabilidade do runtime de integração do Azure-SSIS e aprimora a taxa de transferência geral.

## <a name="next-steps"></a>Próximas etapas
Saiba mais sobre o Microsoft Integration Runtime do Azure-SSIS. Consulte [Microsoft Integration Runtime do Azure-SSIS](concepts-integration-runtime.md#azure-ssis-integration-runtime).
