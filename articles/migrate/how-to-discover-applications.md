---
title: Descobrir aplicativos, funções e recursos em servidores locais com as migrações para Azure
description: Saiba como descobrir aplicativos, funções e recursos em servidores locais com a avaliação de servidor de migrações para Azure.
ms.topic: article
ms.date: 03/12/2020
ms.openlocfilehash: ff9f5489b513cd1405e6b093d7537e4cbcead041
ms.sourcegitcommit: 3beb067d5dc3d8895971b1bc18304e004b8a19b3
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82744627"
---
# <a name="discover-machine-apps-roles-and-features"></a>Descobrir aplicativos, funções e recursos de computador

Este artigo descreve como descobrir aplicativos, funções e recursos em servidores locais, usando migrações para Azure: avaliação de servidor.

Descobrir o inventário de aplicativos e funções/recursos em execução em suas máquinas locais ajuda você a identificar e planejar um caminho de migração para o Azure adaptado para suas cargas de trabalho.

> [!NOTE]
> A descoberta de aplicativos está atualmente em visualização apenas para VMs VMware e é limitada apenas à descoberta. Ainda não oferecemos a avaliação baseada em aplicativo. Avaliação baseada em máquina para VMs VMware locais, VMs Hyper-V e servidores físicos.

Descoberta de aplicativos usando migrações para Azure: a avaliação do servidor é sem agente. Nada é instalado em máquinas e VMs. A avaliação do servidor usa o dispositivo de migrações para Azure para executar a descoberta junto com as credenciais de convidado do computador. O dispositivo acessa remotamente as máquinas VMware usando APIs do VMware.


## <a name="before-you-start"></a>Antes de começar

1. Certifique-se de ter [criado](how-to-add-tool-first-time.md) um projeto de migrações para Azure.
2. Verifique se você [adicionou](how-to-assess.md) a ferramenta migrações do Azure: Server Assessment a um projeto.
4. Verifique os [requisitos de VMware](migrate-support-matrix-vmware.md#vmware-requirements) para descobrir e avaliar VMs VMware com o dispositivo de migrações para Azure.
5. Verifique os [requisitos](migrate-appliance.md) para implantar o dispositivo de migrações para Azure.
6. [Verifique o suporte e os requisitos](migrate-support-matrix-vmware.md#application-discovery) para a descoberta de aplicativos.



## <a name="deploy-the-azure-migrate-appliance"></a>Implantar o dispositivo de migrações para Azure

1. [Examine](migrate-appliance.md#appliance---vmware) os requisitos para implantar o dispositivo de migrações para Azure.
2. Examine as URLs do Azure que o dispositivo precisará acessar nas nuvens [públicas](migrate-appliance.md#public-cloud-urls) e [governamentais](migrate-appliance.md#government-cloud-urls).
3. [Examine os dados](migrate-appliance.md#collected-data---vmware) que o dispositivo coleta durante a descoberta e a avaliação.
4. [Observe](migrate-support-matrix-vmware.md#port-access) os requisitos de acesso da porta para o dispositivo.
5. [Implante o dispositivo de migrações para Azure](how-to-set-up-appliance-vmware.md) para iniciar a descoberta. Para implantar o dispositivo, você baixa e importa um modelo OVA no VMware para criar o dispositivo como uma VM VMware. Você configura o dispositivo e o registra com as migrações para Azure.
6. Ao implantar o dispositivo, para iniciar a descoberta contínua, especifique o seguinte:
    - O nome do vCenter Server ao qual você deseja se conectar.
    - Credenciais que você criou para o dispositivo se conectar ao vCenter Server.
    - As credenciais de conta que você criou para o dispositivo se conectar a VMs do Windows/Linux.

Depois que o dispositivo é implantado e você forneceu credenciais, o dispositivo inicia a descoberta contínua de metadados de VM e dados de desempenho, juntamente com a descoberta de aplicativos, recursos e funções.  A duração da descoberta de aplicativos depende de quantas VMs você tem. Normalmente leva uma hora para a descoberta de aplicativos de 500 VMs.

## <a name="prepare-a-user-account"></a>Preparar uma conta de usuário

Crie uma conta a ser usada para descoberta e adicione-a ao dispositivo.

### <a name="create-a-user-account-for-discovery"></a>Criar uma conta de usuário para descoberta

Configure uma conta de usuário para que a avaliação do servidor possa acessar a VM para descoberta. [Saiba mais](migrate-support-matrix-vmware.md#application-discovery) sobre os requisitos da conta.


### <a name="add-the-user-account-to-the-appliance"></a>Adicionar a conta de usuário ao dispositivo

Adicione a conta de usuário ao dispositivo.

1. Abra o aplicativo de gerenciamento de dispositivo. 
2. Navegue até o painel **fornecer detalhes do vCenter** .
3. Em **descobrir aplicativo e dependências em VMs**, clique em **Adicionar credenciais**
3. Escolha o **sistema operacional**, forneça um nome amigável para a conta e a**senha** do **nome**/de usuário
6. Clique em **Save** (Salvar).
7. Clique em **salvar e iniciar descoberta**.

    ![Adicionar conta de usuário da VM](./media/how-to-create-group-machine-dependencies-agentless/add-vm-credential.png)





## <a name="review-and-export-the-inventory"></a>Revisar e exportar o inventário

Após o término da descoberta, se você forneceu credenciais para a descoberta de aplicativos, poderá revisar e exportar o inventário de aplicativos no portal do Azure.

1. Em **migrações para Azure-servidores** > **migrações para Azure: avaliação do servidor**, clique na contagem exibida para abrir a página **servidores descobertos** .

    > [!NOTE]
    > Nesse estágio, opcionalmente, você também pode configurar o mapeamento de dependência para computadores descobertos, para que você possa Visualizar dependências entre computadores que deseja avaliar. [Saiba mais](how-to-create-group-machine-dependencies.md).

2. Em **aplicativos descobertos**, clique na contagem exibida.
3. No **inventário de aplicativos**, você pode examinar os aplicativos, funções e recursos descobertos.
4. Para exportar o inventário, em **servidores descobertos**, clique em **Exportar inventário de aplicativos**.

O inventário de aplicativos é exportado e baixado no formato do Excel. A folha de **inventário de aplicativos** exibe todos os aplicativos descobertos em todos os computadores.

## <a name="next-steps"></a>Próximas etapas

- [Crie uma avaliação](how-to-create-assessment.md) para a migração de comparação e de deslocamento dos servidores descobertos.
- Avaliar um SQL Server bancos de dados usando [migrações para Azure: avaliação de banco de dados](https://docs.microsoft.com/sql/dma/dma-assess-sql-data-estate-to-sqldb?view=sql-server-2017).
