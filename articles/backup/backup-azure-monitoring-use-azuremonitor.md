---
title: Monitorar o backup do Azure com o Azure Monitor
description: Monitore as cargas de trabalho de backup do Azure e crie alertas personalizados usando Azure Monitor.
ms.topic: conceptual
ms.date: 06/04/2019
ms.assetid: 01169af5-7eb0-4cb0-bbdb-c58ac71bf48b
ms.openlocfilehash: 54a98cebc2887f7508543a4dc752b2145c3bbda2
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "82183646"
---
# <a name="monitor-at-scale-by-using-azure-monitor"></a>Monitorar em escala usando Azure Monitor

O Backup do Azure oferece [funcionalidades internas de monitoramento e alertas](backup-azure-monitoring-built-in-monitor.md) em um cofre dos Serviços de Recuperação. Essas funcionalidades estão disponíveis sem nenhuma infraestrutura de gerenciamento adicional. Mas esse serviço interno é limitado nos seguintes cenários:

- Se você monitorar dados de vários cofres de serviços de recuperação entre assinaturas
- Se o canal de notificação preferencial *não* for email
- Se os usuários quiserem alertas para mais cenários
- Se você quiser exibir informações de um componente local, como o System Center Data Protection Manager no Azure, que o portal não mostra em trabalhos de [**backup**](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault) ou [**alertas de backup**](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault)

## <a name="using-log-analytics-workspace"></a>Usando Log Analytics espaço de trabalho

### <a name="create-alerts-by-using-log-analytics"></a>Criar alertas usando Log Analytics

No Azure Monitor, você pode criar seus próprios alertas em um espaço de trabalho do Log Analytics. No espaço de trabalho, você usa *grupos de ações do Azure* para selecionar o mecanismo de notificação preferencial.

> [!IMPORTANT]
> Para obter informações sobre o custo de criação dessa consulta, consulte [preços de Azure monitor](https://azure.microsoft.com/pricing/details/monitor/).

Abra a seção **logs** do espaço de trabalho log Analytics e crie uma consulta para seus próprios logs. Quando você seleciona **nova regra de alerta**, a página Azure monitor criação de alertas é aberta, conforme mostrado na imagem a seguir.

![Criar um alerta em um espaço de trabalho Log Analytics](media/backup-azure-monitoring-laworkspace/custom-alert.png)

Aqui, o recurso já está marcado como o espaço de trabalho Log Analytics e a integração do grupo de ações é fornecida.

![A página de criação de alertas de Log Analytics](media/backup-azure-monitoring-laworkspace/inkedla-azurebackup-createalert.jpg)

#### <a name="alert-condition"></a>Condição de alerta

A característica de definição de um alerta é sua condição de disparo. Selecione a **condição** para carregar automaticamente a consulta Kusto na página **logs** , conforme mostrado na imagem a seguir. Aqui você pode editar a condição para atender às suas necessidades. Para obter mais informações, consulte [exemplos de consultas do Kusto](#sample-kusto-queries).

![Configurando uma condição de alerta](media/backup-azure-monitoring-laworkspace/la-azurebackup-alertlogic.png)

Se necessário, você pode editar a consulta Kusto. Escolha um limite, período e frequência. O limite determina quando o alerta será gerado. O período é a janela de tempo em que a consulta é executada. Por exemplo, se o limite for maior que 0, o período será de 5 minutos e a frequência será de 5 minutos, a regra executará a consulta a cada 5 minutos, examinando os 5 minutos anteriores. Se o número de resultados for maior que 0, você será notificado pelo grupo de ações selecionado.

#### <a name="alert-action-groups"></a>Grupos de ação de alerta

Use um grupo de ações para especificar um canal de notificação. Para ver os mecanismos de notificação disponíveis, em **grupos de ação**, selecione **criar novo**.

![Mecanismos de notificação disponíveis na janela "Adicionar grupo de ações"](media/backup-azure-monitoring-laworkspace/LA-AzureBackup-ActionGroup.png)

Você pode atender a todos os requisitos de alerta e monitoramento de Log Analytics sozinho ou pode usar Log Analytics para complementar as notificações internas.

Para obter mais informações, consulte [criar, exibir e gerenciar alertas de log usando Azure monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log) e [criar e gerenciar grupos de ações no portal do Azure](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups).

### <a name="sample-kusto-queries"></a>Consultas Kusto de exemplo

Os grafos padrão fornecem a você Kusto consultas para cenários básicos nos quais você pode criar alertas. Você também pode modificar as consultas para obter os dados que você deseja que sejam alertados. Cole as seguintes consultas Kusto de exemplo na página **logs** e, em seguida, crie alertas nas consultas:

- Todos os trabalhos de backup bem-sucedidos

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | where JobStatus=="Completed"
    ````

- Todos os trabalhos de backup com falha

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | where JobStatus=="Failed"
    ````

- Todos os trabalhos de backup de VM do Azure bem-sucedidos

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="VM" and BackupManagementType=="IaaSVM"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Todos os trabalhos de backup de log SQL bem-sucedidos

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup" and JobOperationSubType=="Log"
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="SQLDataBase" and BackupManagementType=="AzureWorkload"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Todos os trabalhos do agente de backup do Azure bem-sucedidos

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="FileFolder" and BackupManagementType=="MAB"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Armazenamento de backup consumido por item de backup

    ````Kusto
    CoreAzureBackup
    //Get all Backup Items
    | where OperationName == "BackupItem"
    //Get distinct Backup Items
    | distinct BackupItemUniqueId, BackupItemFriendlyName
    | join kind=leftouter
    (AddonAzureBackupStorage
    | where OperationName == "StorageAssociation"
    //Get latest record for each Backup Item
    | summarize arg_max(TimeGenerated, *) by BackupItemUniqueId
    | project BackupItemUniqueId , StorageConsumedInMBs)
    on BackupItemUniqueId
    | project BackupItemUniqueId , BackupItemFriendlyName , StorageConsumedInMBs
    | sort by StorageConsumedInMBs desc
    ````

### <a name="diagnostic-data-update-frequency"></a>Frequência de atualização de dados de diagnóstico

Os dados de diagnóstico do cofre são bombeados para o espaço de trabalho Log Analytics com algum retardo. Cada evento chega no espaço de trabalho de Log Analytics de *20 a 30 minutos* após ser enviado por push do cofre dos serviços de recuperação. Aqui estão mais detalhes sobre o retardo:

- Em todas as soluções, os alertas internos do serviço de backup são enviados por push assim que são criados. Eles geralmente aparecem no espaço de trabalho Log Analytics após 20 a 30 minutos.
- Em todas as soluções, trabalhos de backup sob demanda e trabalhos de restauração são enviados por push assim que forem *concluídos*.
- Para todas as soluções, exceto backup do SQL, os trabalhos de backup agendados são enviados assim que forem *concluídos*.
- Para o backup do SQL, como os backups de log podem ocorrer a cada 15 minutos, as informações para todos os trabalhos de backup agendados concluídos, incluindo logs, são colocadas em lote e enviadas a cada 6 horas.
- Em todas as soluções, outras informações, como o item de backup, a política, os pontos de recuperação, o armazenamento e assim por diante, são enviadas pelo menos *uma vez por dia.*
- Uma alteração na configuração de backup (como alteração de política ou política de edição) dispara um envio por push de todas as informações de backup relacionadas.

## <a name="using-the-recovery-services-vaults-activity-logs"></a>Usando os logs de atividade do cofre dos serviços de recuperação

> [!CAUTION]
> As etapas a seguir se aplicam somente a *backups de VM do Azure.* Você não pode usar estas etapas para soluções como o agente de backup do Azure, backups do SQL no Azure ou arquivos do Azure.

Você também pode usar logs de atividade para obter notificações para eventos como o êxito do backup. Para começar, siga estas etapas:

1. Entre no Portal do Azure.
1. Abra o cofre dos serviços de recuperação relevante.
1. Nas propriedades do cofre, abra a seção **log de atividades** .

Para identificar o log apropriado e criar um alerta:

1. Verifique se você está recebendo logs de atividade para backups bem-sucedidos aplicando os filtros mostrados na imagem a seguir. Altere o valor de **TimeSpan** conforme necessário para exibir os registros.

   ![Filtragem para localizar logs de atividade para backups de VM do Azure](media/backup-azure-monitoring-laworkspace/activitylogs-azurebackup-vmbackups.png)

1. Selecione o nome da operação para ver os detalhes relevantes.
1. Selecione **nova regra de alerta** para abrir a página **criar regra** .
1. Crie um alerta seguindo as etapas em [criar, exibir e gerenciar alertas do log de atividades usando Azure monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log).

   ![Nova regra de alerta](media/backup-azure-monitoring-laworkspace/new-alert-rule.png)

Aqui, o recurso é o cofre dos serviços de recuperação em si. Repita as mesmas etapas para todos os cofres nos quais você deseja ser notificado por meio dos logs de atividade. A condição não terá um limite, período ou frequência, pois esse alerta se baseia em eventos. Assim que o log de atividades relevante é gerado, o alerta é gerado.

## <a name="using-log-analytics-to-monitor-at-scale"></a>Usando Log Analytics para monitorar em escala

Você pode exibir todos os alertas criados de logs de atividade e Log Analytics espaços de trabalho no Azure Monitor. Basta abrir o painel **alertas** à esquerda.

Embora você possa obter notificações por meio de logs de atividade, é altamente recomendável usar Log Analytics em vez de logs de atividade para monitoramento em escala. Eis o porquê:

- **Cenários limitados**: as notificações por meio de logs de atividade se aplicam somente aos backups de VM do Azure. As notificações devem ser configuradas para cada cofre de serviços de recuperação.
- **Ajuste de definição**: a atividade de backup agendada não se ajusta à definição mais recente dos logs de atividade. Em vez disso, ele se alinha com [os logs de recursos](https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-workspace#what-you-can-do-with-platform-logs-in-a-workspace). Esse alinhamento causa efeitos inesperados quando os dados que fluem pelo canal do log de atividades são alterados.
- **Problemas com o canal do log de atividades**: em cofres dos serviços de recuperação, os logs de atividade que são bombeados do backup do Azure seguem um novo modelo. Infelizmente, essa alteração afeta a geração de logs de atividades no Azure governamental, no Azure Alemanha e no Azure China 21Vianet. Se os usuários desses serviços de nuvem criarem ou configurarem quaisquer alertas dos logs de atividade no Azure Monitor, os alertas não serão disparados. Além disso, em todas as regiões públicas do Azure, se um usuário [coletar logs de atividade dos serviços de recuperação em um espaço de trabalho log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/collect-activity-logs), esses logs não aparecerão.

Use um espaço de trabalho Log Analytics para monitoramento e alertas em escala para todas as suas cargas que são protegidas pelo backup do Azure.

## <a name="next-steps"></a>Próximas etapas

Para criar consultas personalizadas, consulte [modelo de dados do log Analytics](backup-azure-reports-data-model.md).
