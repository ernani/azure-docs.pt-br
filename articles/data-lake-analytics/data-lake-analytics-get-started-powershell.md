---
title: Criar & consulta Azure Data Lake Analytics-PowerShell
description: Use o Azure PowerShell para criar uma conta do Azure Data Lake Analytics e envie um trabalho do U-SQL.
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 8a4e901e-9656-4a60-90d0-d78ff2f00656
ms.topic: conceptual
ms.date: 05/04/2017
ms.openlocfilehash: 8b176434ce450382cc6463e7d21d341a74581ed8
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "71316603"
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-powershell"></a>Introdução ao Azure Data Lake Analytics usando o Azure PowerShell

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Saiba como usar o Azure PowerShell para criar contas do Azure Data Lake Analytics e, em seguida, enviar e executar trabalhos do U-SQL. Para obter mais informações sobre Data Lake Analytics, consulte [Azure data Lake Analytics visão geral](data-lake-analytics-overview.md).

## <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Antes de começar este tutorial, você deve ter as seguintes informações:

* **Uma conta da Análise Azure Data Lake**. Veja [Introdução ao Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal).
* **Uma estação de trabalho com Azure PowerShell**. Consulte [Como instalar e configurar o PowerShell do Azure](/powershell/azure/overview).

## <a name="log-in-to-azure"></a>Fazer logon no Azure

Este tutorial pressupõe que você já esteja familiarizado com o uso do Azure PowerShell. Em particular, você precisa saber como fazer logon no Azure. Veja a [Introdução ao Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) se precisar de ajuda.

Para fazer logon com um nome de assinatura:

```powershell
Connect-AzAccount -SubscriptionName "ContosoSubscription"
```

Em vez do nome da assinatura, você também pode usar uma id de assinatura para fazer logon:

```powershell
Connect-AzAccount -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Se for bem-sucedido, a saída desse comando se parece com o seguinte texto:

```
Environment           : AzureCloud
Account               : joe@contoso.com
TenantId              : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionId        : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionName      : ContosoSubscription
CurrentStorageAccount :
```

## <a name="preparing-for-the-tutorial"></a>Preparando-se para o tutorial

Os snippets do PowerShell neste tutorial usam essas variáveis para armazenar estas informações:

```powershell
$rg = "<ResourceGroupName>"
$adls = "<DataLakeStoreAccountName>"
$adla = "<DataLakeAnalyticsAccountName>"
$location = "East US 2"
```

## <a name="get-information-about-a-data-lake-analytics-account"></a>Obter informações sobre uma conta do Data Lake Analytics

```powershell
Get-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla  
```

## <a name="submit-a-u-sql-job"></a>Enviar um trabalho do U-SQL

Crie uma variável do PowerShell para manter o script U-SQL.

```powershell
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();

"@
```

Envie o texto do script com o cmdlet `Submit-AdlJob` e o parâmetro `-Script`.

```powershell
$job = Submit-AdlJob -Account $adla -Name "My Job" -Script $script
```

Como alternativa, você pode enviar um arquivo de script usando o parâmetro `-ScriptPath`:

```powershell
$filename = "d:\test.usql"
$script | out-File $filename
$job = Submit-AdlJob -Account $adla -Name "My Job" -ScriptPath $filename
```

Obter o status de um trabalho com `Get-AdlJob`. 

```powershell
$job = Get-AdlJob -Account $adla -JobId $job.JobId
```

Em vez de chamar Get-AdlJob repetidamente até que um trabalho seja concluído, use o cmdlet `Wait-AdlJob`.

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```

Baixe o arquivo de saída usando `Export-AdlStoreItem`.

```powershell
Export-AdlStoreItem -Account $adls -Path "/data.csv" -Destination "C:\data.csv"
```

## <a name="see-also"></a>Confira também

* Para ver o mesmo tutorial usando outras ferramentas, clique nos seletores de guias na parte superior da página.
* Para aprender a usar o U-SQL, veja [Introdução à linguagem U-SQL da Análise do Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
* Para obter as tarefas de gerenciamento, confira [Gerenciar o Azure Data Lake Analytics usando o portal do Azure](data-lake-analytics-manage-use-portal.md).
